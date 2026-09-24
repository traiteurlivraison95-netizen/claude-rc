---
paths:
  - "**/*.php"
  - "**/composer.json"
---
# Patterns PHP

> Ce fichier étend [common/patterns.md](../common/patterns.md) avec du contenu spécifique à PHP.

## Contrôleurs minces, services explicites

- Garder les contrôleurs concentrés sur le transport : authentification, validation, sérialisation, codes de statut.
- Déplacer les règles métier vers des services applicatifs/de domaine faciles à tester sans amorçage HTTP.

## DTO et objets-valeur

- Remplacer les tableaux associatifs à structure lourde par des DTO pour les requêtes, commandes et payloads d'API externes.
- Utiliser des objets-valeur pour l'argent, les identifiants, les plages de dates et autres concepts contraints.

## Injection de dépendances

- Dépendre d'interfaces ou de contrats de service étroits, pas des globales du framework.
- Passer les collaborateurs via les constructeurs afin que les services soient testables sans recherches de type service-locator.

## Frontières

- Isoler les modèles ORM des décisions de domaine lorsque la couche modèle fait plus que de la persistance.
- Envelopper les SDK tiers derrière de petits adaptateurs afin que le reste de la base de code dépende de votre contrat, pas du leur.

## Référence

Voir la compétence : `api-design` pour les conventions de endpoints et les conseils sur la forme des réponses.
Voir la compétence : `laravel-patterns` pour des conseils d'architecture spécifiques à Laravel.
