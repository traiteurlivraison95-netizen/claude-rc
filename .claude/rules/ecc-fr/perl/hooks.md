---
paths:
  - "**/*.pl"
  - "**/*.pm"
  - "**/*.t"
  - "**/*.psgi"
  - "**/*.cgi"
---
# Hooks Perl

> Ce fichier étend [common/hooks.md](../common/hooks.md) avec du contenu spécifique à Perl.

## Hooks PostToolUse

À configurer dans `~/.claude/settings.json` :

- **perltidy** : formate automatiquement les fichiers `.pl` et `.pm` après modification
- **perlcritic** : exécute une vérification de linting après modification des fichiers `.pm`

## Avertissements

- Avertir en cas d'utilisation de `print` dans des fichiers `.pm` non-scripts — utiliser `say` ou un module de journalisation (ex. `Log::Any`)
