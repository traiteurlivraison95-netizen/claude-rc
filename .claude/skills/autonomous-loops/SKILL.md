---
name: autonomous-loops
description: "Mechanism layer for autonomous agent loops — how to wire one in this harness and keep it recoverable. Covers the runtime primitives (ScheduleWakeup, /loop, cron, subagents, background tasks), the four loop patterns /loop-start dispatches (sequential, continuous-pr, rfc-dag, infinite), the iteration contract, on-disk checkpoint and runbook layout, stall detection and recovery via ecc loop-status, budget bounds, and escalation. Use when building, wiring, resuming, or debugging a long-running loop; when choosing between a wakeup, a cron trigger and a subagent fan-out; or when a loop has stalled, is re-doing work, or lost its state. Does NOT decide whether the loop's goal is right — that is loop-design-check. 中文触发：怎么接 loop、loop 跑不动了、loop 卡住、断点续跑、loop 状态存哪、定时还是唤醒、loop 模式选哪个。English triggers: wire a loop, loop mechanism, loop patterns, loop stalled, resume a loop, loop checkpoint, loop state, schedule vs wakeup."
license: MIT
metadata:
  origin: ECC
---

# Autonomous Loops (mechanism layer)

> **Scope boundary.** This skill answers *how do I wire this loop so it runs, survives a restart, and can be stopped* — patterns, primitives, state, recovery. It does **not** answer *is this goal right, will it cheat, should it exist* — that is [`loop-design-check`](../loop-design-check/SKILL.md), and you run that **first**. A loop wired perfectly around an undecidable goal is a perfectly reliable way to burn tokens.

**Order of operations:** `loop-design-check` (should this exist? is the goal decidable?) → **this skill** (wire it) → `/loop-start` (launch it) → `/loop-status` (watch it).

---

## 1 · Pick the primitive

The pattern is the shape of the work; the primitive is what actually re-invokes the agent. Match them or the loop will not survive its first restart.

| You need | Primitive | Bounds | Notes |
|---|---|---|---|
| Self-paced iterations, agent decides the cadence | `ScheduleWakeup` (`/loop` dynamic mode) | delay clamped to **60–3600s** | Pass the same prompt back each turn or the loop ends. `stop: true` ends it. Mark `noop: true` on a quiet tick. |
| Fixed interval, same prompt | `/loop <interval> <prompt>` | interval set at launch | The thermostat/regulator case. No design needed for a plain poll. |
| Must happen at a wall-clock time, or must survive this session ending | `CronCreate` / `create_trigger` | normally hourly minimum | Use fresh-session-per-fire when each run should start from a clean slate; bind to a session when the run needs this conversation's context. |
| One deferred resumption of *this* conversation | `send_later` | one-shot | Cheapest way to re-enter with context intact. |
| Parallel independent units in one iteration | subagents (`Agent`, fan-out) | context per child | The parent **must** collect results before ending its turn — a spawned task is not a completed task. |
| Waiting on external state you cannot be notified about | `Monitor` with an until-loop, or a long fallback wakeup (1200s+) | — | Never `sleep` in the foreground to wait. |
| Waiting on PR events | `subscribe_pr_activity` | until merged/closed | Events wake the session; do not poll. Pair with an hourly `send_later` fallback, because CI-success and merge-conflict events can be dropped. |

**Anti-pattern:** a short-interval wakeup polling for work the harness already notifies you about. That is a pure waste — schedule a long fallback (1200s+) instead so the loop survives a hang, and let the notification drive the fast path.

---

## 2 · Pattern catalog

These are the four patterns `/loop-start <pattern>` dispatches, and what `ecc-recipes` means when it names one.

### `sequential` — one unit at a time, in order

Shape: `unit[i] → verify → checkpoint → unit[i+1]`. A work queue drained front to back.

- **State:** an ordered unit list plus a cursor. The cursor advances **only after** the unit's verification passed and the checkpoint was written.
- **Stops when:** the queue is empty, or a unit fails its retry cap.
- **Primitive:** `ScheduleWakeup` between units; cron if it must span sessions.
- **Fails by:** a half-applied unit — the agent edits three files, dies on the fourth, and the next iteration cannot tell how far it got. Fix with the iteration contract (§3): one unit, one commit, cursor last.
- **Use for:** migrations, file-by-file refactors, a task list with real ordering constraints.

### `continuous-pr` — drive a change to mergeable, then the next

Shape: `open/adopt PR → react to CI + review events → push fix → repeat until green and mergeable`.

- **State:** the PR number is the state. Its CI status, review threads, and mergeability are the queue — do not maintain a private copy that can drift.
- **Stops when:** merged or closed. Not when green — green plus waiting reviewers is still open.
- **Primitive:** `subscribe_pr_activity` for the fast path, plus an hourly `send_later` fallback because event delivery is best-effort.
- **Fails by:** treating a red check as "waiting on review", and by re-running a job and calling a real failure a flake. One re-run, at most, and only to confirm a failure that is not this change's.
- **Use for:** getting a change landed; babysitting CI.

### `rfc-dag` — a dependency graph of units, fanned out

Shape: units with declared dependencies; every unit whose deps are all satisfied runs in parallel; the frontier advances as units complete.

- **State:** a dependency graph plus a per-unit status (`pending` / `running` / `awaiting-verification` / `done` / `blocked`). Status advances **one way and never rolls back**.
- **Stops when:** every unit is `done`, or the frontier is empty while units remain (a real deadlock — escalate; do not "unblock" by dropping an edge).
- **Primitive:** subagents for the frontier, `ScheduleWakeup` between waves. The parent collects every child before ending the turn.
- **Fails by:** two parallel units editing the same file. Partition by file or module before the wave launches, not after the conflict.
- **Use for:** a decomposed spec or epic where units are genuinely independent.

### `infinite` — no endpoint, maintain a state

Shape: a regulator. Sample, compare to the desired state, act **only on change**.

- **State:** the last observed value, so an unchanged world produces no action.
- **Stops when:** it does not. That is the point — so the budget bound (§6) and the human's kill switch are the only things between it and an unbounded spend.
- **Primitive:** `/loop <interval>` or cron. Never a tight wakeup.
- **Fails by:** no dead-band — it acts on noise, thrashing the same state back and forth. Require a change threshold before acting, and make actions idempotent.
- **Use for:** health checks, drift detection, inbox/alert triage.

> **Mode.** `--mode safe` (the `/loop-start` default) runs full verification and writes a checkpoint every iteration. `--mode fast` may batch verification across units — legitimate only when each unit is independently revertible. `fast` never removes the retry cap, the budget bound, or the human's sign-off.

---

## 3 · The iteration contract

Everything about recoverability reduces to this. **Every iteration must be atomic and idempotent.**

1. **Read state from disk, not from memory.** The next iteration may be a fresh session in a fresh container. If a fact only exists in this conversation, it is already lost.
2. **One iteration = one unit of work = one commit.** Not three units, not half a unit.
3. **Verify before you record.** Run the deterministic check, then write the checkpoint. Never the reverse.
4. **Advance the cursor last.** Order: do work → verify → commit → *then* advance state. A crash at any point leaves the unit merely un-started, which is safe. A cursor advanced first turns a crash into silently skipped work.
5. **Re-running an iteration must be harmless.** Append-only writes, `mkdir -p`, upserts. Assume every iteration runs at least once and possibly twice.
6. **State advances one way.** `pending → running → awaiting-verification → done`. Never roll a status backward to retry; add a new attempt record instead, so the retry count stays visible.
7. **The exit code is final.** If the verification script says non-zero, the script wins — the agent's opinion that the change "looks right" does not overturn it.
8. **`done` is flipped by a human.** The loop may advance a unit to `awaiting-verification`. Acceptance is not the loop's to grant (see `loop-design-check` red lines).

---

## 4 · On-disk layout

A loop that keeps its state in the conversation dies with the conversation. Two files, both committed or both in a known path:

```
.claude/plans/<loop-name>.md      # the runbook: goal, boundaries, units, stop condition,
                                  # retry cap, budget, escalation contacts, how to kill it
.claude/plans/<loop-name>.state.json
```

`/loop-start` writes the runbook under `.claude/plans/` (creating it — the directory does not exist until a loop needs it). The runbook is the human interface: a person reads one file and knows what the loop is doing and how to stop it.

Minimum state shape:

```json
{
  "loop": "nightly-green-keeper",
  "pattern": "sequential",
  "mode": "safe",
  "cursor": 7,
  "units": [{ "id": "u7", "status": "awaiting-verification", "attempts": 2 }],
  "iterations": 41,
  "started_at": "2026-09-27T02:00:00Z",
  "last_checkpoint_at": "2026-09-27T05:12:00Z",
  "budget": { "max_iterations": 200, "retry_cap": 3 },
  "last_failure": "pytest tests/test_sync.py::test_totals — assertion on upstream total"
}
```

Two disciplines from the maintenance skeleton apply to whatever holds this state: **the problem column is human-write-only, the result column is loop-write-only**, and nothing the loop writes may silently edit the goal or the acceptance conditions.

`ecc loop-status --write-dir ~/.claude/loops` maintains `index.json` plus per-session snapshots, so a sibling terminal or watchdog can read loop state without waiting for the Claude session to dequeue `/loop-status`.

---

## 5 · Stall detection and recovery

A stalled loop is worse than a stopped one: it looks alive and bills like it.

**Detect.** `/loop-status`, or from another terminal:

```bash
npx --package ecc-universal ecc loop-status --json
ecc loop-status --exit-code --watch --watch-count 3   # 2 = stale signals found
```

It scans local transcripts under `~/.claude/projects/**` for stale `ScheduleWakeup` calls and `Bash` calls with no matching `tool_result` — the two mechanical signatures of a wedged loop. Adjust the stale-Bash threshold with `--bash-timeout-seconds`.

**Escalate** when any of these holds (mirrors the `loop-operator` agent's triggers):

- no progress across two consecutive checkpoints — the cursor has not moved
- repeated failures with an identical stack trace — retrying will not help
- cost drift outside the budget window (§6)
- merge conflicts blocking queue advancement
- the frontier is empty but units remain `pending` — a dependency deadlock

**Recover, in this order:**

1. **Stop the loop first.** `ScheduleWakeup stop: true`, disable the trigger, or `TaskStop`. Do not diagnose a loop that is still spending.
2. **Read the state file, not the transcript.** The state file is the truth about how far it got.
3. **Reconcile disk against state.** Any unit marked `running` is suspect: check whether its work half-landed. This is the moment §3's "advance the cursor last" pays for itself.
4. **Reduce scope, then resume.** One unit, one iteration, watched by a human. A loop that failed at width 8 should not resume at width 8.
5. **Resume only after verification passes** on the reconciled state.
6. **Record the failure in the runbook.** A loop whose failures are not written down re-earns them.

**Never recover by:** loosening the verification, deleting or skipping the failing check, raising the retry cap to get past a real failure, or rolling a `done` status backward. Those convert a visible stall into an invisible wrong answer.

---

## 6 · Budget bounds

An autonomous loop spends without asking. Every loop declares, in the runbook, **before** the first iteration:

- **Retry cap per unit** (N, then escalate to a human — not N, then try forever)
- **Max iterations** or a wall-clock deadline — a hard stop independent of the goal
- **Max concurrent subagents** — fan-out multiplies spend per wave, not per loop
- **What "over budget" does** — pause and escalate, never silently continue

`infinite` loops need this most, because they have no natural terminus: the budget bound *is* their stop condition.

---

## 7 · Pre-flight checklist

Do not launch until every line is true:

- [ ] `loop-design-check` run: the goal is machine-decidable, boundaries are stated alongside it, clarifications are front-loaded
- [ ] Verification is a **deterministic command with an exit code**, and the judge is **not** the agent doing the work
- [ ] Baseline green: the verification passes *now*, before iteration 1 (otherwise iteration 1 cannot tell your bug from its own)
- [ ] State file and runbook paths exist and are written by iteration 1
- [ ] Iteration is atomic and idempotent (§3), re-runnable without damage
- [ ] Retry cap, max iterations, and budget are written in the runbook
- [ ] Branch or worktree isolation configured — the loop does not commit to the default branch
- [ ] A rollback path exists and has been tried once by hand
- [ ] Kill switch documented in the runbook: how a human stops this loop in one command
- [ ] Ran once by hand end to end — staging is `hand → skill/subagents → cron`, never straight to cron
- [ ] Sign-off stays with a human: the loop opens the PR, a person merges it

---

## 8 · One-line close

> Mechanism buys you exactly one thing: the loop can be stopped, inspected, and resumed without losing work. Everything else — whether it should run at all — is judgment, and that stays with the human.

---

> Judgment layer (goal decidability, the five failure modes, red lines): [`loop-design-check`](../loop-design-check/SKILL.md).
> Verification depth for a single iteration: [`verification-loop`](../verification-loop/SKILL.md); adversarial double-review before shipping: [`santa-method`](../santa-method/SKILL.md).
> Operator surface: `/loop-start`, `/loop-status`, and the `loop-operator` agent.
