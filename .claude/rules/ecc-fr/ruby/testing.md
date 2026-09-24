---
paths:
  - "**/*.rb"
  - "**/*.rake"
  - "**/Gemfile"
  - "**/test/**/*.rb"
  - "**/spec/**/*.rb"
  - "**/config/routes.rb"
---
# Tests Ruby

> Ce fichier étend [common/testing.md](../common/testing.md) avec du contenu spécifique à Ruby et Rails.

## Framework

- Utiliser **Minitest** quand l'application Rails suit la pile de tests Rails par défaut.
- Utiliser **RSpec** quand il est déjà établi dans le projet ou que l'équipe a des conventions de production explicites à son sujet.
- Ne pas mélanger Minitest et RSpec au sein de la même zone fonctionnelle sans raison de migration.

## Pyramide de tests

- Placer le comportement de domaine rapide dans les tests de modèle, service, query, policy, et job.
- Utiliser les tests request/controller pour les contrats HTTP, le comportement d'authentification, les redirections, les codes de statut, et les formes de réponse.
- Utiliser les tests système avec Capybara uniquement pour les flux critiques dans le navigateur ; les garder ciblés et stables.
- Couvrir les jobs en arrière-plan avec des tests unitaires pour le comportement et des tests d'intégration pour les contrats de queue/enqueue.

## Fixtures et factories

- Utiliser les fixtures Rails quand elles sont le défaut du projet et que le graphe de données est petit.
- Utiliser `factory_bot` quand les scénarios nécessitent une construction d'objet explicite ou des traits complexes.
- Garder les données de test proches du comportement testé ; éviter les fixtures globales qui masquent le coût de la configuration.

## Commandes

Préférer les commandes locales au projet :

```bash
bin/rails test
bin/rails test test/models/user_test.rb
bundle exec rspec
bundle exec rspec spec/models/user_spec.rb
```

## Couverture

- Utiliser SimpleCov quand la couverture est imposée ; garder les seuils en CI et éviter de tricher sur la couverture de branches avec des tests à faible valeur.
- Ajouter des tests de non-régression pour les corrections de bugs avant de modifier le code de production.

## Référence

Voir la skill : `tdd-workflow` pour la boucle RED -> GREEN -> REFACTOR à l'échelle du dépôt.
