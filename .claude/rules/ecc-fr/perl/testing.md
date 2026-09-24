---
paths:
  - "**/*.pl"
  - "**/*.pm"
  - "**/*.t"
  - "**/*.psgi"
  - "**/*.cgi"
---
# Tests Perl

> Ce fichier étend [common/testing.md](../common/testing.md) avec du contenu spécifique à Perl.

## Framework

Utiliser **Test2::V0** pour les nouveaux projets (et non Test::More) :

```perl
use Test2::V0;

is($result, 42, 'answer is correct');

done_testing;
```

## Lanceur de tests

```bash
prove -l t/              # ajoute lib/ à @INC
prove -lr -j8 t/         # récursif, 8 jobs en parallèle
```

Toujours utiliser `-l` pour s'assurer que `lib/` est dans `@INC`.

## Couverture

Utiliser **Devel::Cover** — viser 80 % ou plus :

```bash
cover -test
```

## Mocking

- **Test::MockModule** — mocker des méthodes sur des modules existants
- **Test::MockObject** — créer des doublures de test à partir de rien

## Pièges

- Toujours terminer les fichiers de test par `done_testing`
- Ne jamais oublier l'option `-l` avec `prove`

## Référence

Voir la compétence : `perl-testing` pour des patterns détaillés de TDD Perl avec Test2::V0, prove et Devel::Cover.
