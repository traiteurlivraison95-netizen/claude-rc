---
paths:
  - "**/*.java"
  - "**/pom.xml"
  - "**/build.gradle"
  - "**/build.gradle.kts"
---
# Hooks Java

> Ce fichier étend [common/hooks.md](../common/hooks.md) avec du contenu spécifique à Java.

## Hooks PostToolUse

À configurer dans `~/.claude/settings.json` :

- **google-java-format** : formate automatiquement les fichiers `.java` après édition
- **checkstyle** : exécute des vérifications de style après édition des fichiers Java
- **./mvnw compile** ou **./gradlew compileJava** : vérifie la compilation après les modifications
