---
paths:
  - "**/*.go"
  - "**/go.mod"
  - "**/go.sum"
---
# Style de code Go

> Ce fichier étend [common/coding-style.md](../common/coding-style.md) avec du contenu spécifique à Go.

## Formatage

- **gofmt** et **goimports** sont obligatoires — aucun débat de style

## Principes de conception

- Accepter des interfaces, retourner des structs
- Garder les interfaces petites (1 à 3 méthodes)

## Gestion des erreurs

Toujours envelopper les erreurs avec du contexte :

```go
if err != nil {
    return fmt.Errorf("failed to create user: %w", err)
}
```

## Référence

Voir le skill : `golang-patterns` pour des idiomes et patterns Go complets.
