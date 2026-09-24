---
paths:
  - "**/*.go"
  - "**/go.mod"
  - "**/go.sum"
---
# Tests Go

> Ce fichier étend [common/testing.md](../common/testing.md) avec du contenu spécifique à Go.

## Framework

Utiliser le `go test` standard avec des **tests pilotés par table** (table-driven tests).

## Détection des races

Toujours exécuter avec l'option `-race` :

```bash
go test -race ./...
```

## Couverture

```bash
go test -cover ./...
```

## Référence

Voir le skill : `golang-testing` pour des patterns et helpers de test Go détaillés.
