---
paths:
  - "**/*.py"
  - "**/*.pyi"
---
# Style de code Python

> Ce fichier étend [common/coding-style.md](../common/coding-style.md) avec du contenu spécifique à Python.

## Standards

- Suivre les conventions **PEP 8**
- Utiliser des **annotations de type** sur toutes les signatures de fonction

## Immutabilité

Préférer les structures de données immuables :

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class User:
    name: str
    email: str

from typing import NamedTuple

class Point(NamedTuple):
    x: float
    y: float
```

## Formatage

- **black** pour le formatage du code
- **isort** pour le tri des imports
- **ruff** pour le linting

## Référence

Voir la compétence : `python-patterns` pour des idiomes et patterns Python complets.
