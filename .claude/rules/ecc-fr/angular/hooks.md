---
paths:
  - "**/*.component.ts"
  - "**/*.component.html"
  - "**/*.service.ts"
  - "**/*.directive.ts"
  - "**/*.pipe.ts"
  - "**/*.spec.ts"
---
# Hooks Angular

> Ce fichier étend [common/hooks.md](../common/hooks.md) avec du contenu spécifique à Angular.

## Hooks PostToolUse

À configurer dans `~/.claude/settings.json` :

- **Prettier** : formate automatiquement les fichiers `.ts` et `.html` après édition
- **ESLint / ng lint** : exécute `ng lint` après édition des fichiers source Angular pour détecter les mauvais usages de décorateurs, les erreurs de template et les violations de style
- **Vérification TypeScript** : exécute `tsc --noEmit` après édition des fichiers `.ts`
- **Vérification de build** : exécute `ng build` après génération ou modification significative de code Angular pour détecter tôt les erreurs de template et de typage

## Hooks Stop

- **Audit de lint** : exécute `ng lint` sur les fichiers modifiés avant la fin de session pour détecter toute violation restante
