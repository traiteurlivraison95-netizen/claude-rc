# Système de hooks

## Types de hooks

- **PreToolUse** : avant l'exécution d'un outil (validation, modification de paramètres)
- **PostToolUse** : après l'exécution d'un outil (formatage automatique, vérifications)
- **Stop** : à la fin de la session (vérification finale)

## Permissions en auto-acceptation

À utiliser avec prudence :
- Activer pour des plans fiables et bien définis
- Désactiver pour le travail exploratoire
- Ne jamais utiliser le flag dangerously-skip-permissions
- Configurer `allowedTools` dans `~/.claude.json` à la place

## Bonnes pratiques TodoWrite

Utiliser l'outil TodoWrite pour :
- Suivre la progression sur des tâches à plusieurs étapes
- Vérifier la compréhension des instructions
- Permettre un pilotage en temps réel
- Montrer les étapes d'implémentation en détail

La liste de todos révèle :
- Des étapes dans le désordre
- Des éléments manquants
- Des éléments superflus inutiles
- Une granularité incorrecte
- Des exigences mal interprétées
