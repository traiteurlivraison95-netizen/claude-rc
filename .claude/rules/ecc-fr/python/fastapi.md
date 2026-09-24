---
paths:
  - "**/app/**/*.py"
  - "**/fastapi/**/*.py"
  - "**/*_api.py"
---
# Règles FastAPI

Utiliser ces règles pour les projets FastAPI en complément des règles Python générales.

## Structure

- Placer la construction de l'application dans `create_app()`.
- Garder les routeurs minces ; déplacer la persistance et le comportement métier vers des services ou des helpers CRUD.
- Garder séparés les schémas de requête, de mise à jour et de réponse.
- Garder les sessions de base de données et l'authentification dans les dépendances.

## Async

- Utiliser `async def` pour les endpoints qui effectuent des E/S.
- Utiliser des clients de base de données et HTTP asynchrones depuis les endpoints async.
- Ne pas appeler `requests`, des sessions SQLAlchemy synchrones, ou des opérations fichier/réseau bloquantes depuis des routes async.

## Injection de dépendances

```python
@router.get("/users/{user_id}")
async def get_user(
    user_id: str,
    db: AsyncSession = Depends(get_db),
    current_user: User = Depends(get_current_user),
):
    ...
```

Ne pas créer `SessionLocal()` ou des clients à durée de vie longue à l'intérieur des gestionnaires de route.

## Schémas

- Ne jamais inclure de mots de passe, hachages de mots de passe, jetons d'accès, jetons de rafraîchissement ou état d'authentification interne dans les modèles de réponse.
- Utiliser `response_model` sur les endpoints qui retournent des données applicatives.
- Utiliser des contraintes de champ plutôt qu'une validation écrite à la main lorsque Pydantic peut exprimer la règle.

## Sécurité

- Garder les origines CORS spécifiques à l'environnement.
- Ne pas combiner des origines génériques (wildcard) avec un CORS avec identifiants.
- Valider l'expiration, l'émetteur, l'audience et l'algorithme du JWT.
- Limiter le débit (rate-limit) des endpoints d'authentification et à forte écriture.
- Rédiger (masquer) les identifiants, cookies, en-têtes d'autorisation et jetons dans les logs.

## Tests

- Surcharger la dépendance exacte utilisée par `Depends`.
- Nettoyer `app.dependency_overrides` après les tests.
- Préférer des clients de test asynchrones pour les applications asynchrones.

Voir la compétence : `fastapi-patterns`.
