---
description: Start a managed autonomous loop pattern with safety defaults and explicit stop conditions.
---

# Loop Start Command

Start a managed autonomous loop pattern with safety defaults.

## Usage

`/loop-start [pattern] [--mode safe|fast]`

- `pattern`: `sequential`, `continuous-pr`, `rfc-dag`, `infinite`
  (each pattern's shape, state, stop condition and failure mode is defined in the
  `autonomous-loops` skill, §2 — read it before selecting one)
- `--mode`:
  - `safe` (default): strict quality gates and checkpoints
  - `fast`: reduced gates for speed

## Flow

1. Confirm repository state and branch strategy.
2. Select loop pattern and model tier strategy.
3. Enable required hooks/profile for the chosen mode.
4. Create loop plan and write runbook under `.claude/plans/`.
5. Print commands to start and monitor the loop.

## Required Safety Checks

- Verify tests pass before first loop iteration.
- Ensure `ECC_HOOK_PROFILE` is not disabled globally.
- Ensure loop has explicit stop condition.

Run the full pre-flight checklist in the `autonomous-loops` skill (§7) before the
first iteration, and run `loop-design-check` before that to confirm the goal is
machine-decidable. The runbook and state file this command writes under
`.claude/plans/` follow the layout in `autonomous-loops` §4.

## Arguments

$ARGUMENTS:
- `<pattern>` optional (`sequential|continuous-pr|rfc-dag|infinite`)
- `--mode safe|fast` optional
