---
paths:
  - "**/*.ts"
  - "**/*.tsx"
  - "**/*.js"
  - "**/*.jsx"
---
# Hooks TypeScript/JavaScript

> Ce fichier étend [common/hooks.md](../common/hooks.md) avec du contenu spécifique à TypeScript/JavaScript.

## Hooks PostToolUse

Configurer dans `~/.claude/settings.json` :

- **Prettier** : formater automatiquement les fichiers JS/TS après modification
- **Vérification TypeScript** : exécuter `tsc` après modification des fichiers `.ts`/`.tsx`
- **Avertissement console.log** : avertir en cas de `console.log` dans les fichiers modifiés

## Hooks Stop

- **Audit console.log** : vérifier tous les fichiers modifiés pour `console.log` avant la fin de session
