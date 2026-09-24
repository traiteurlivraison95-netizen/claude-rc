---
paths:
  - "**/*.rb"
  - "**/*.rake"
  - "**/Gemfile"
  - "**/Gemfile.lock"
  - "**/config/routes.rb"
---
# Hooks Ruby

> Ce fichier étend [common/hooks.md](../common/hooks.md) avec du contenu spécifique à Ruby et Rails.

## Hooks PostToolUse

Configurer des hooks locaux au projet pour privilégier les binstubs et l'outillage versionné :

- **RuboCop** : exécuter `bundle exec rubocop -A <file>` ou la commande de formateur plus sûre du projet après des modifications Ruby.
- **Brakeman** : exécuter `bundle exec brakeman --no-progress` après des changements Rails sensibles pour la sécurité.
- **Tests** : exécuter la commande `bin/rails test ...` ou `bundle exec rspec ...` la plus ciblée correspondant aux fichiers modifiés.
- **Bundler audit** : exécuter `bundle exec bundle-audit check --update` quand `Gemfile` ou `Gemfile.lock` change et que le projet a bundler-audit installé.

## Avertissements

- Avertir en cas d'appels `debugger`, `binding.irb`, `binding.pry`, `puts`, `pp`, ou `p` commités dans le code d'application.
- Avertir quand une modification désactive la protection CSRF, étend le mass-assignment, ou ajoute du SQL brut sans paramétrage.
- Avertir quand une migration modifie des données de façon destructive sans chemin réversible ni plan de déploiement documenté.

## Suggestions de portail CI

```bash
bundle exec rubocop
bundle exec brakeman --no-progress
bin/rails test
bundle exec rspec
```

N'utiliser que les commandes présentes dans le projet ; ne pas installer de nouvelles dépendances de hook sans l'approbation du mainteneur.
