---
paths:
  - "**/*.py"
  - "**/*.pyi"
---
# Sécurité Python

> Ce fichier étend [common/security.md](../common/security.md) avec du contenu spécifique à Python.

## Gestion des secrets

```python
import os
from dotenv import load_dotenv

load_dotenv()

api_key = os.environ["OPENAI_API_KEY"]  # Lève une KeyError si absent
```

## Analyse de sécurité

- Utiliser **bandit** pour l'analyse de sécurité statique :
  ```bash
  bandit -r src/
  ```

## Référence

Voir la compétence : `django-security` pour les directives de sécurité spécifiques à Django (le cas échéant).
