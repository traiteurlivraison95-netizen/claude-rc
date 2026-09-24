---
paths:
  - "**/*.go"
  - "**/go.mod"
  - "**/go.sum"
---
# Hooks Go

> Ce fichier étend [common/hooks.md](../common/hooks.md) avec du contenu spécifique à Go.

## Hooks PostToolUse

À configurer dans `~/.claude/settings.json` :

- **gofmt/goimports** : formate automatiquement les fichiers `.go` après édition
- **go vet** : exécute une analyse statique après édition des fichiers `.go`
- **staticcheck** : exécute des vérifications statiques étendues sur les paquets modifiés
