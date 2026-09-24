---
paths:
  - "**/*.py"
  - "**/*.pyi"
---
# Tests Python

> Ce fichier étend [common/testing.md](../common/testing.md) avec du contenu spécifique à Python.

## Framework

Utiliser **pytest** comme framework de test.

## Couverture

```bash
pytest --cov=src --cov-report=term-missing
```

## Organisation des tests

Utiliser `pytest.mark` pour la catégorisation des tests :

```python
import pytest

@pytest.mark.unit
def test_calculate_total():
    ...

@pytest.mark.integration
def test_database_connection():
    ...
```

## Référence

Voir la compétence : `python-testing` pour des patterns pytest détaillés et des fixtures.
