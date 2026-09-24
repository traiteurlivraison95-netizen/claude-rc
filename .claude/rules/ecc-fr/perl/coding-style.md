---
paths:
  - "**/*.pl"
  - "**/*.pm"
  - "**/*.t"
  - "**/*.psgi"
  - "**/*.cgi"
---
# Style de code Perl

> Ce fichier étend [common/coding-style.md](../common/coding-style.md) avec du contenu spécifique à Perl.

## Standards

- Toujours `use v5.36` (active `strict`, `warnings`, `say`, les signatures de sous-routines)
- Utiliser les signatures de sous-routines — ne jamais décompresser `@_` manuellement
- Préférer `say` à `print` avec des retours à la ligne explicites

## Immutabilité

- Utiliser **Moo** avec `is => 'ro'` et `Types::Standard` pour tous les attributs
- Ne jamais utiliser directement des hashrefs bénis (blessed) — toujours utiliser les accesseurs Moo/Moose
- **Remarque sur la surcharge OO** : les attributs `has` de Moo avec `builder` ou `default` sont acceptables pour les valeurs calculées en lecture seule

## Formatage

Utiliser **perltidy** avec ces réglages :

```
-i=4    # indentation de 4 espaces
-l=100  # longueur de ligne de 100 caractères
-ce     # else en cascade (cuddled)
-bar    # accolade ouvrante toujours à droite
```

## Linting

Utiliser **perlcritic** au niveau de sévérité 3 avec les thèmes : `core`, `pbp`, `security`.

```bash
perlcritic --severity 3 --theme 'core || pbp || security' lib/
```

## Référence

Voir la compétence : `perl-patterns` pour des idiomes et bonnes pratiques Perl modernes complets.
