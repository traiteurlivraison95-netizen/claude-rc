---
paths:
  - "**/*.cs"
  - "**/*.csx"
  - "**/*.csproj"
  - "**/appsettings*.json"
---
# Sécurité C#

> Ce fichier étend [common/security.md](../common/security.md) avec du contenu spécifique à C#.

## Gestion des secrets

- Ne fige jamais en dur des clés d'API, jetons ou chaînes de connexion dans le code source
- Utilise des variables d'environnement, les user secrets pour le développement local, et un gestionnaire de secrets en production
- Garde `appsettings.*.json` exempt de vrais identifiants

```csharp
// MAUVAIS
const string ApiKey = "sk-live-123";

// BON
var apiKey = builder.Configuration["OpenAI:ApiKey"]
    ?? throw new InvalidOperationException("OpenAI:ApiKey is not configured.");
```

## Prévention de l'injection SQL

- Utilise toujours des requêtes paramétrées avec ADO.NET, Dapper, ou EF Core
- Ne concatène jamais d'entrées utilisateur dans des chaînes SQL
- Valide les champs de tri et les opérateurs de filtre avant d'utiliser une composition dynamique de requête

```csharp
const string sql = "SELECT * FROM Orders WHERE CustomerId = @customerId";
await connection.QueryAsync<Order>(sql, new { customerId });
```

## Validation des entrées

- Valide les DTO à la frontière applicative
- Utilise les annotations de données, FluentValidation, ou des clauses de garde explicites
- Rejette un état de modèle invalide avant d'exécuter la logique métier

## Authentification et autorisation

- Privilégie les gestionnaires d'authentification du framework plutôt que l'analyse manuelle de jetons
- Applique les politiques d'autorisation aux frontières des endpoints ou des handlers
- Ne journalise jamais de jetons bruts, mots de passe, ou PII

## Gestion des erreurs

- Retourne des messages sûrs côté client
- Journalise les exceptions détaillées avec un contexte structuré côté serveur
- N'expose pas de traces de pile, de texte SQL, ou de chemins du système de fichiers dans les réponses d'API

## Références

Voir le skill : `security-review` pour des checklists de revue de sécurité applicative plus larges.
