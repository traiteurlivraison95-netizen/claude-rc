# Orchestration d'agents

## Agents disponibles

Les agents ECC sont livrés avec le plugin `ecc@ecc`, pas dans `~/.claude/agents/`.
Ils sont invoqués via l'outil Agent avec un `subagent_type` propre au plugin :

```text
Agent(subagent_type: "ecc:planner", prompt: "...")
```

| Agent | Objectif | Quand l'utiliser |
|-------|---------|-------------|
| ecc:planner | Planification d'implémentation | Fonctionnalités complexes, refactoring |
| ecc:architect | Conception système | Décisions architecturales |
| ecc:tdd-guide | Développement piloté par les tests | Nouvelles fonctionnalités, corrections de bugs |
| ecc:code-reviewer | Revue de code | Après avoir écrit du code |
| ecc:security-reviewer | Analyse de sécurité | Avant les commits |
| ecc:build-error-resolver | Correction des erreurs de build | En cas d'échec de build |
| ecc:e2e-runner | Tests E2E | Parcours utilisateur critiques |
| ecc:refactor-cleaner | Nettoyage de code mort | Maintenance du code |
| ecc:doc-updater | Documentation | Mise à jour de la documentation |
| ecc:rust-reviewer | Revue de code Rust | Projets Rust |
| ecc:harmonyos-app-resolver | Développement d'applications HarmonyOS | Projets HarmonyOS/ArkTS |

Pour la liste complète des 68 agents, voir `/ecc:ecc-guide`.

## Utilisation immédiate des agents

Sans besoin de prompt utilisateur :
1. Demande de fonctionnalité complexe - utiliser l'agent **ecc:planner**
2. Code venant d'être écrit/modifié - utiliser l'agent **ecc:code-reviewer**
3. Correction de bug ou nouvelle fonctionnalité - utiliser l'agent **ecc:tdd-guide**
4. Décision architecturale - utiliser l'agent **ecc:architect**

## Exécution parallèle des tâches

TOUJOURS utiliser l'exécution parallèle des tâches pour les opérations indépendantes :

```markdown
# BON : Exécution parallèle
Lancer 3 agents en parallèle :
1. Agent 1 : analyse de sécurité du module d'auth
2. Agent 2 : revue de performance du système de cache
3. Agent 3 : vérification de types des utilitaires

# MAUVAIS : séquentiel alors que ce n'est pas nécessaire
D'abord l'agent 1, puis l'agent 2, puis l'agent 3
```

## Contrat d'achèvement de délégation

S'applique à chaque agent, à chaque profondeur (parent, enfant, petit-enfant) :

1. **Votre message final EST le livrable.** Ne jamais terminer votre tour avec « en attente des agents en arrière-plan » — une tâche lancée n'est pas une tâche terminée. Terminer votre tour pendant que des enfants sont en cours d'exécution rend leurs résultats orphelins (un enfant terminé ne peut pas notifier un parent dont le tour est terminé).
2. **Si vous déléguez, vous êtes responsable de la collecte.** Attendre les résultats, les intégrer, puis répondre. La délégation « fire-and-forget » est interdite.
3. **Ne décomposer que lorsque le travail ne tient pas dans un seul contexte.** Ne pas re-déléguer une tâche déjà dimensionnée pour un seul agent — la profondeur est un résultat, pas un plan.

> Justification : mode d'échec observé — des agents de recherche ont suivi la règle « Exécution parallèle des tâches » ci-dessus, ont lancé des enfants, et ont renvoyé « en attente » comme réponse finale. Tous les enfants se sont terminés avec succès mais leurs résultats sont restés orphelins. La règle de parallélisation sans contrat d'achèvement produit des tâches zombies.

## Analyse multi-perspective

Pour les problèmes complexes, utiliser des sous-agents à rôles répartis :
- Réviseur factuel
- Ingénieur senior
- Expert en sécurité
- Réviseur de cohérence
- Vérificateur de redondance
