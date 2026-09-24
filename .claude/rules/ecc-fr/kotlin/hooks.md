---
paths:
  - "**/*.kt"
  - "**/*.kts"
  - "**/build.gradle.kts"
---
# Hooks Kotlin

> Ce fichier étend [common/hooks.md](../common/hooks.md) avec du contenu spécifique à Kotlin.

## Hooks PostToolUse

À configurer dans `~/.claude/settings.json` :

- **ktfmt/ktlint** : formate automatiquement les fichiers `.kt` et `.kts` après édition
- **detekt** : exécute une analyse statique après édition des fichiers Kotlin
- **./gradlew build** : vérifie la compilation après les modifications
