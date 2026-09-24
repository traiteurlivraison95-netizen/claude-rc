---
paths:
  - "**/*.fs"
  - "**/*.fsx"
  - "**/*.fsproj"
  - "**/appsettings*.json"
---
# Sécurité F#

> Ce fichier étend [common/security.md](../common/security.md) avec du contenu spécifique à F#.

## Gestion des secrets

- Ne jamais coder en dur des clés API, des jetons ou des chaînes de connexion dans le code source
- Utiliser des variables d'environnement, les user secrets pour le développement local, et un gestionnaire de secrets en production
- Garder les fichiers `appsettings.*.json` exempts de véritables identifiants

```fsharp
// MAUVAIS
let apiKey = "sk-live-123"

// BON
let apiKey =
    configuration["OpenAI:ApiKey"]
    |> Option.ofObj
    |> Option.defaultWith (fun () -> failwith "OpenAI:ApiKey is not configured.")
```

## Prévention des injections SQL

- Toujours utiliser des requêtes paramétrées avec ADO.NET, Dapper ou EF Core
- Ne jamais concaténer des entrées utilisateur dans des chaînes SQL
- Valider les champs de tri et les opérateurs de filtre avant d'utiliser une composition de requête dynamique

```fsharp
let findByCustomer (connection: IDbConnection) customerId =
    task {
        let sql = "SELECT * FROM Orders WHERE CustomerId = @customerId"
        return! connection.QueryAsync<Order>(sql, {| customerId = customerId |})
    }
```

## Validation des entrées

- Valider les entrées à la frontière de l'application en utilisant les types
- Utiliser des unions discriminées à cas unique pour les valeurs validées
- Rejeter les entrées invalides avant qu'elles n'entrent dans la logique de domaine

```fsharp
type ValidatedEmail = private ValidatedEmail of string

module ValidatedEmail =
    let create (input: string) =
        if System.Text.RegularExpressions.Regex.IsMatch(input, @"^[^@]+@[^@]+\.[^@]+$") then
            Ok(ValidatedEmail input)
        else
            Error "Invalid email address"

    let value (ValidatedEmail v) = v
```

## Authentification et autorisation

- Préférer les gestionnaires d'authentification du framework à l'analyse de jetons personnalisée
- Appliquer les politiques d'autorisation aux frontières des endpoints ou des handlers
- Ne jamais logger les jetons bruts, les mots de passe ou les données personnelles (PII)

## Gestion des erreurs

- Retourner des messages sûrs orientés client
- Logger les exceptions détaillées avec un contexte structuré côté serveur
- Ne pas exposer les traces de pile, le texte SQL ou les chemins de fichiers dans les réponses API

## Références

Voir le skill : `security-review` pour des checklists de sécurité applicative plus larges.
