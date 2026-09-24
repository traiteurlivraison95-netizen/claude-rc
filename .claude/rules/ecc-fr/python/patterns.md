---
paths:
  - "**/*.py"
  - "**/*.pyi"
---
# Patterns Python

> Ce fichier étend [common/patterns.md](../common/patterns.md) avec du contenu spécifique à Python.

## Protocol (typage canard)

```python
from typing import Protocol

class Repository(Protocol):
    def find_by_id(self, id: str) -> dict | None: ...
    def save(self, entity: dict) -> dict: ...
```

## Dataclasses comme DTO

```python
from dataclasses import dataclass

@dataclass
class CreateUserRequest:
    name: str
    email: str
    age: int | None = None
```

## Gestionnaires de contexte et générateurs

- Utiliser les gestionnaires de contexte (instruction `with`) pour la gestion des ressources
- Utiliser des générateurs pour l'évaluation paresseuse et l'itération économe en mémoire

## Référence

Voir la compétence : `python-patterns` pour des patterns complets incluant les décorateurs, la concurrence et l'organisation des packages.
