---
paths:
  - "**/*.go"
  - "**/go.mod"
  - "**/go.sum"
---
# Sécurité Go

> Ce fichier étend [common/security.md](../common/security.md) avec du contenu spécifique à Go.

## Gestion des secrets

```go
apiKey := os.Getenv("OPENAI_API_KEY")
if apiKey == "" {
    log.Fatal("OPENAI_API_KEY not configured")
}
```

## Analyse de sécurité

- Utiliser **gosec** pour l'analyse de sécurité statique :
  ```bash
  gosec ./...
  ```

## Context et délais d'expiration

Toujours utiliser `context.Context` pour le contrôle des délais :

```go
ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
defer cancel()
```
