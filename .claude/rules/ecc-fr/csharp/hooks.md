---
paths:
  - "**/*.cs"
  - "**/*.csx"
  - "**/*.csproj"
  - "**/*.sln"
  - "**/Directory.Build.props"
  - "**/Directory.Build.targets"
---
# Hooks C#

> Ce fichier étend [common/hooks.md](../common/hooks.md) avec du contenu spécifique à C#.

## Hooks PostToolUse

À configurer dans `~/.claude/settings.json` :

- **dotnet format** : formate automatiquement les fichiers C# édités et applique les corrections d'analyseur
- **dotnet build** : vérifie que la solution ou le projet compile toujours après les modifications
- **dotnet test --no-build** : relance le projet de tests pertinent le plus proche après des changements de comportement

## Hooks Stop

- Exécute un `dotnet build` final avant de terminer une session comportant des changements C# étendus
- Avertit sur les fichiers `appsettings*.json` modifiés pour que des secrets ne soient pas commités
