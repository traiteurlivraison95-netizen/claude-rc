---
paths:
  - "**/*.php"
  - "**/phpunit.xml"
  - "**/phpunit.xml.dist"
  - "**/composer.json"
---
# Tests PHP

> Ce fichier étend [common/testing.md](../common/testing.md) avec du contenu spécifique à PHP.

## Framework

Utiliser **PHPUnit** comme framework de test par défaut. Si **Pest** est configuré dans le projet, préférer Pest pour les nouveaux tests et éviter de mélanger les frameworks.

## Couverture

```bash
vendor/bin/phpunit --coverage-text
# ou
vendor/bin/pest --coverage
```

Préférer **pcov** ou **Xdebug** en CI, et conserver les seuils de couverture en CI plutôt que comme connaissance tribale.

## Organisation des tests

- Séparer les tests unitaires rapides des tests d'intégration framework/base de données.
- Utiliser des factories/builders pour les fixtures plutôt que de larges tableaux écrits à la main.
- Garder les tests HTTP/contrôleur concentrés sur le transport et la validation ; déplacer les règles métier vers des tests au niveau service.

## Inertia

Si le projet utilise Inertia.js, préférer `assertInertia` avec `AssertableInertia` pour vérifier les noms de composants et les props plutôt que des assertions JSON brutes.

## Référence

Voir la compétence : `tdd-workflow` pour la boucle RED -> GREEN -> REFACTOR à l'échelle du dépôt.
Voir la compétence : `laravel-tdd` pour des patterns de test spécifiques à Laravel (PHPUnit et Pest).
