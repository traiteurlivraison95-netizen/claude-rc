---
paths:
  - "**/*.fs"
  - "**/*.fsx"
  - "**/*.fsproj"
  - "**/*.sln"
  - "**/*.slnx"
  - "**/Directory.Build.props"
  - "**/Directory.Build.targets"
---
# Hooks F#

> Ce fichier étend [common/hooks.md](../common/hooks.md) avec du contenu spécifique à F#.

## Hooks PostToolUse

À configurer dans `~/.claude/settings.json` :

- **fantomas** : formate automatiquement les fichiers F# modifiés
- **dotnet build** : vérifie que la solution ou le projet compile toujours après les modifications
- **dotnet test --no-build** : relance le projet de test le plus proche après un changement de comportement

## Hooks Stop

- Exécuter un `dotnet build` final avant de terminer une session avec des changements F# étendus
- Avertir en cas de modification des fichiers `appsettings*.json` pour que des secrets ne soient pas commités
