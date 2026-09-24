---
paths:
  - "**/*.pl"
  - "**/*.pm"
  - "**/*.t"
  - "**/*.psgi"
  - "**/*.cgi"
---
# Sécurité Perl

> Ce fichier étend [common/security.md](../common/security.md) avec du contenu spécifique à Perl.

## Mode Taint

- Utiliser l'option `-T` sur tous les scripts CGI/web
- Nettoyer `%ENV` (`$ENV{PATH}`, `$ENV{CDPATH}`, etc.) avant tout appel à une commande externe

## Validation des entrées

- Utiliser une regex de liste blanche pour l'untainting — jamais `/(.*)/s`
- Valider toutes les entrées utilisateur avec des patterns explicites :

```perl
if ($input =~ /\A([a-zA-Z0-9_-]+)\z/) {
    my $clean = $1;
}
```

## E/S de fichiers

- **Open à trois arguments uniquement** — jamais l'open à deux arguments
- Empêcher le path traversal avec `Cwd::realpath` :

```perl
use Cwd 'realpath';
my $safe_path = realpath($user_path);
die "Path traversal" unless $safe_path =~ m{\A/allowed/directory/};
```

## Exécution de processus

- Utiliser **`system()` en forme liste** — jamais la forme chaîne unique
- Utiliser **IPC::Run3** pour capturer la sortie
- Ne jamais utiliser de backticks avec interpolation de variable

```perl
system('grep', '-r', $pattern, $directory);  # sûr
```

## Prévention de l'injection SQL

Toujours utiliser les placeholders DBI — jamais interpoler dans le SQL :

```perl
my $sth = $dbh->prepare('SELECT * FROM users WHERE email = ?');
$sth->execute($email);
```

## Analyse de sécurité

Exécuter **perlcritic** avec le thème sécurité au niveau de sévérité 4+ :

```bash
perlcritic --severity 4 --theme security lib/
```

## Référence

Voir la compétence : `perl-security` pour des patterns de sécurité Perl complets, le mode taint et les E/S sûres.
