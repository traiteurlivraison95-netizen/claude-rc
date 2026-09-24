# Workflow Git

## Format des messages de commit
```
<type>: <description>

<corps optionnel>
```

Types : feat, fix, refactor, docs, test, chore, perf, ci

Note : les installations gérées par ECC définissent `"includeCoAuthoredBy": false` dans `~/.claude/settings.json`, donc les commits ne portent pas de trailer `Co-Authored-By` par défaut. Pour conserver l'attribution Claude, définissez `"includeCoAuthoredBy": true` ou configurez `attribution` ; ECC n'écrase jamais un choix explicite.

## Workflow de Pull Request

Lors de la création de PR :
1. Analyser l'historique complet des commits (pas seulement le dernier)
2. Utiliser `git diff [base-branch]...HEAD` pour voir tous les changements
3. Rédiger un résumé de PR complet
4. Inclure un plan de test avec des TODO
5. Pousser avec le flag `-u` si nouvelle branche

> Pour le processus de développement complet (planification, TDD, revue de code) avant les opérations git,
> voir [development-workflow.md](./development-workflow.md).
