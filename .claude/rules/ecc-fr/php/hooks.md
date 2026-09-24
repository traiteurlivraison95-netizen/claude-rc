---
paths:
  - "**/*.php"
  - "**/composer.json"
  - "**/phpstan.neon"
  - "**/phpstan.neon.dist"
  - "**/psalm.xml"
---
# Hooks PHP

> Ce fichier étend [common/hooks.md](../common/hooks.md) avec du contenu spécifique à PHP.

## Hooks PostToolUse

À configurer dans `~/.claude/settings.json` :

- **Pint / PHP-CS-Fixer** : formate automatiquement les fichiers `.php` modifiés.
- **PHPStan / Psalm** : exécute une analyse statique après les modifications PHP dans les bases de code typées.
- **PHPUnit / Pest** : exécute les tests ciblés pour les fichiers ou modules touchés lorsque les modifications affectent le comportement.

## Avertissements

- Avertir en cas de `var_dump`, `dd`, `dump` ou `die()` laissés dans les fichiers modifiés.
- Avertir lorsque les fichiers PHP modifiés ajoutent du SQL brut ou désactivent les protections CSRF/session.
