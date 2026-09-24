---
paths:
  - "**/*.swift"
  - "**/Package.swift"
---
# Hooks Swift

> Ce fichier étend [common/hooks.md](../common/hooks.md) avec du contenu spécifique à Swift.

## Hooks PostToolUse

Configurer dans `~/.claude/settings.json` :

- **SwiftFormat** : formater automatiquement les fichiers `.swift` après modification
- **SwiftLint** : exécuter les vérifications de lint après modification des fichiers `.swift`
- **swift build** : vérifier les types des packages modifiés après édition

## Avertissement

Signaler les instructions `print()` — utiliser `os.Logger` ou une journalisation structurée à la place pour le code de production.
