---
paths:
  - "**/*.pl"
  - "**/*.pm"
  - "**/*.t"
  - "**/*.psgi"
  - "**/*.cgi"
---
# Patterns Perl

> Ce fichier étend [common/patterns.md](../common/patterns.md) avec du contenu spécifique à Perl.

## Pattern Repository

Utiliser **DBI** ou **DBIx::Class** derrière une interface :

```perl
package MyApp::Repo::User;
use Moo;

has dbh => (is => 'ro', required => 1);

sub find_by_id ($self, $id) {
    my $sth = $self->dbh->prepare('SELECT * FROM users WHERE id = ?');
    $sth->execute($id);
    return $sth->fetchrow_hashref;
}
```

## DTO / Objets-valeur

Utiliser des classes **Moo** avec **Types::Standard** (équivalent des dataclasses Python) :

```perl
package MyApp::DTO::User;
use Moo;
use Types::Standard qw(Str Int);

has name  => (is => 'ro', isa => Str, required => 1);
has email => (is => 'ro', isa => Str, required => 1);
has age   => (is => 'ro', isa => Int);
```

## Gestion des ressources

- Toujours utiliser **open à trois arguments** avec `autodie`
- Utiliser **Path::Tiny** pour les opérations sur les fichiers

```perl
use autodie;
use Path::Tiny;

my $content = path('config.json')->slurp_utf8;
```

## Interface de module

Utiliser `Exporter 'import'` avec `@EXPORT_OK` — jamais `@EXPORT` :

```perl
use Exporter 'import';
our @EXPORT_OK = qw(parse_config validate_input);
```

## Gestion des dépendances

Utiliser **cpanfile** + **carton** pour des installations reproductibles :

```bash
carton install
carton exec prove -lr t/
```

## Référence

Voir la compétence : `perl-patterns` pour des patterns et idiomes Perl modernes complets.
