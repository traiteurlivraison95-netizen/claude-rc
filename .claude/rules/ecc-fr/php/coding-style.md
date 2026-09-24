---
paths:
  - "**/*.php"
  - "**/composer.json"
---
# Style de code PHP

> Ce fichier étend [common/coding-style.md](../common/coding-style.md) avec du contenu spécifique à PHP.

## Standards

- Suivre les conventions de formatage et de nommage **PSR-12**.
- Préférer `declare(strict_types=1);` dans le code applicatif.
- Utiliser des indications de type scalaire, des types de retour et des propriétés typées partout où le nouveau code le permet.

## Immutabilité

- Préférer les DTO et objets-valeur immuables pour les données traversant les frontières de service.
- Utiliser des propriétés `readonly` ou des constructeurs immuables pour les payloads de requête/réponse lorsque possible.
- Conserver des tableaux pour les correspondances simples ; faire évoluer les structures critiques pour l'activité en classes explicites.

## Formatage

- Utiliser **PHP-CS-Fixer** ou **Laravel Pint** pour le formatage.
- Utiliser **PHPStan** ou **Psalm** pour l'analyse statique.
- Conserver les scripts Composer versionnés afin que les mêmes commandes s'exécutent en local et en CI.

## Imports

- Ajouter des déclarations `use` pour toutes les classes, interfaces et traits référencés.
- Éviter de s'appuyer sur l'espace de noms global sauf si le projet préfère explicitement les noms pleinement qualifiés.

## Gestion des erreurs

- Lever des exceptions pour les états exceptionnels ; éviter de retourner `false`/`null` comme canal d'erreur caché dans le nouveau code.
- Convertir les entrées du framework/de la requête en DTO validés avant qu'elles n'atteignent la logique métier.

## Référence

Voir la compétence : `backend-patterns` pour des conseils plus larges sur la stratification service/repository.
