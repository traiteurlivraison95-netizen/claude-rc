---
paths:
  - "**/*.dart"
  - "**/pubspec.yaml"
  - "**/analysis_options.yaml"
---
# Hooks Dart/Flutter

> Ce fichier étend [common/hooks.md](../common/hooks.md) avec du contenu spécifique à Dart et Flutter.

## Hooks PostToolUse

À configurer dans `~/.claude/settings.json` :

- **dart format** : formate automatiquement les fichiers `.dart` après édition
- **dart analyze** : exécute l'analyse statique après édition des fichiers Dart et fait remonter les avertissements
- **flutter test** : exécute optionnellement les tests concernés après des changements significatifs

## Configuration de hook recommandée

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": { "tool_name": "Edit", "file_paths": ["**/*.dart"] },
        "hooks": [
          { "type": "command", "command": "dart format $CLAUDE_FILE_PATHS" }
        ]
      }
    ]
  }
}
```

## Vérifications avant commit

À exécuter avant de commiter des changements Dart/Flutter :

```bash
dart format --set-exit-if-changed .
dart analyze --fatal-infos
flutter test
```

## Commandes utiles

```bash
# Formater tous les fichiers Dart
dart format .

# Analyser et signaler les problèmes
dart analyze

# Exécuter tous les tests avec couverture
flutter test --coverage

# Régénérer les fichiers de génération de code
dart run build_runner build --delete-conflicting-outputs

# Vérifier les paquets obsolètes
flutter pub outdated

# Mettre à jour les paquets dans les contraintes
flutter pub upgrade
```
