---
paths:
  - "**/*.rb"
  - "**/*.rake"
  - "**/Gemfile"
  - "**/app/**/*.erb"
  - "**/config/routes.rb"
---
# Patterns Ruby

> Ce fichier étend [common/patterns.md](../common/patterns.md) avec du contenu spécifique à Ruby et Rails.

## Priorité à la manière Rails

- Commencer avec le MVC Rails classique et les conventions Active Record pour les fonctionnalités petites et moyennes.
- Introduire des service objects, query objects, form objects, decorators ou presenters lorsque la frontière modèle/contrôleur porte plusieurs responsabilités.
- Nommer les objets extraits d'après l'opération métier qu'ils réalisent, pas d'après des couches génériques comme `Manager` ou `Processor`.

## Persistance

- Préférer PostgreSQL pour les applications Rails en production multi-hôtes, sauf si la plateforme existante a une raison claire d'utiliser MySQL ou SQLite.
- Considérer les valeurs par défaut Rails 8 basées sur SQLite comme viables pour un déploiement mono-hôte ou modeste, pas comme une solution automatique pour des systèmes multi-services partagés.
- Garder le SQL brut derrière des query objects ou des scopes de modèle, et paramétrer chaque valeur dynamique.

## Jobs en arrière-plan et services runtime

- Utiliser **Solid Queue** pour les nouvelles applications Rails 8 avec un débit modeste et des besoins de déploiement simples.
- Utiliser **Sidekiq** quand l'application a besoin d'une observabilité mature, d'un débit élevé, d'une infrastructure Redis existante, ou de fonctionnalités Pro/Enterprise.
- Utiliser **Solid Cache** et **Solid Cable** quand leur modèle de déploiement correspond à l'application ; utiliser Redis quand le comportement partagé inter-services, le fanout élevé, ou des structures de données avancées comptent.

## Frontend

- Préférer **Hotwire** avec Turbo, Stimulus, Importmap et Propshaft pour les applications Rails rendues côté serveur.
- Utiliser React, Vue, Inertia.js, ou une SPA séparée quand la complexité d'interaction, l'architecture produit existante, ou la propriété d'équipe justifie la surface client supplémentaire.
- Garder les view components, partials et presenters concentrés sur les décisions de rendu ; garder la persistance et l'autorisation hors des templates.

## Authentification

- Utiliser le générateur d'authentification Rails 8 pour les besoins simples d'authentification par session et de réinitialisation de mot de passe.
- Utiliser Devise ou un autre système d'authentification établi quand les exigences incluent OAuth, MFA, des flux confirmable/lockable, une authentification multi-modèle, ou une base Devise existante importante.

## Référence

Voir la skill : `backend-patterns` pour les frontières de service et les patterns d'adaptateur.
