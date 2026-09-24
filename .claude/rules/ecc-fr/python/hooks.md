---
paths:
  - "**/*.py"
  - "**/*.pyi"
---
# Hooks Python

> Ce fichier étend [common/hooks.md](../common/hooks.md) avec du contenu spécifique à Python.

## Hooks PostToolUse

À configurer dans `~/.claude/settings.json` :

- **black/ruff** : formate automatiquement les fichiers `.py` après modification
- **mypy/pyright** : exécute la vérification de type après modification des fichiers `.py`

## Avertissements

- Avertir en cas d'instructions `print()` dans les fichiers modifiés (utiliser le module `logging` à la place)
