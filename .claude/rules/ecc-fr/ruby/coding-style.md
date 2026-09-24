---
paths:
  - "**/*.rb"
  - "**/*.rake"
  - "**/Gemfile"
  - "**/*.gemspec"
  - "**/config.ru"
---
# Style de code Ruby

> Ce fichier étend [common/coding-style.md](../common/coding-style.md) avec du contenu spécifique à Ruby et Rails.

## Standards

- Viser **Ruby 3.3+** pour tout nouveau travail Rails, sauf si le projet fixe déjà un runtime supporté plus ancien.
- N'activer **YJIT** en production qu'après avoir mesuré le temps de démarrage, la mémoire et le débit des requêtes/jobs.
- Ajouter `# frozen_string_literal: true` aux nouveaux fichiers Ruby lorsque le projet utilise cette convention.
- Préférer un Ruby clair à une métaprogrammation astucieuse ; isoler le code fortement DSL derrière des frontières étroites et testées.

## Formatage et linting

- Utiliser la configuration RuboCop versionnée du projet. Pour les applications Rails 8+, partir de `rubocop-rails-omakase` et ne personnaliser que là où la base de code a une convention réelle.
- Garder les commandes de formateur/linter derrière des binstubs ou des scripts pour que la CI et les exécutions locales correspondent :

```bash
bundle exec rubocop
bundle exec rubocop -A
```

- Ne pas désactiver les cops en ligne sauf si l'exception est étroite, documentée, et plus difficile à exprimer proprement dans le code.

## Style Rails

- Suivre les conventions de nommage et de répertoires Rails avant d'ajouter une structure personnalisée.
- Garder les contrôleurs centrés sur le transport : authentification, autorisation, gestion des paramètres, forme de la réponse.
- Placer le comportement métier réutilisable dans des modèles, concerns, service objects, query objects ou form objects selon la complexité réelle, pas comme une cérémonie par défaut.
- Préférer `bin/rails`, `bin/rake`, et les binstubs versionnés aux commandes installées globalement.

## Gestion des erreurs

- Capturer des exceptions spécifiques. Éviter les blocs `rescue StandardError` larges sauf s'ils relancent l'exception ou préservent assez de contexte pour les opérateurs.
- Utiliser `ActiveSupport::Notifications` ou le logger de l'application pour les événements opérationnels ; ne pas laisser `puts`, `pp`, ou `debugger` dans le code d'application commité.

## Référence

Voir la skill : `backend-patterns` pour des conseils plus larges sur la stratification service/repository.
