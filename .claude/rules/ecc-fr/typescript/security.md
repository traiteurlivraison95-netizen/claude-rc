---
paths:
  - "**/*.ts"
  - "**/*.tsx"
  - "**/*.js"
  - "**/*.jsx"
---
# Sécurité TypeScript/JavaScript

> Ce fichier étend [common/security.md](../common/security.md) avec du contenu spécifique à TypeScript/JavaScript.

## Gestion des secrets

```typescript
// JAMAIS : secrets codés en dur
const apiKey = "sk-proj-xxxxx"

// TOUJOURS : variables d'environnement
const apiKey = process.env.API_KEY

if (!apiKey) {
  throw new Error('API_KEY not configured')
}
```

## Support par agent

- Utiliser la skill **security-reviewer** pour des audits de sécurité complets
