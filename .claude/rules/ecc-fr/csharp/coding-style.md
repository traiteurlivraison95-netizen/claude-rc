---
paths:
  - "**/*.cs"
  - "**/*.csx"
---
# Style de code C#

> Ce fichier étend [common/coding-style.md](../common/coding-style.md) avec du contenu spécifique à C#.

## Standards

- Suis les conventions .NET actuelles et active les types de référence nullable
- Privilégie les modificateurs d'accès explicites sur les API publiques et internes
- Garde les fichiers alignés avec le type principal qu'ils définissent

## Types et modèles

- Privilégie `record` ou `record struct` pour les modèles immuables de type valeur
- Utilise `class` pour les entités ou les types avec identité et cycle de vie
- Utilise `interface` pour les frontières de service et les abstractions
- Évite `dynamic` dans le code applicatif ; privilégie les génériques ou des modèles explicites

```csharp
public sealed record UserDto(Guid Id, string Email);

public interface IUserRepository
{
    Task<UserDto?> FindByIdAsync(Guid id, CancellationToken cancellationToken);
}
```

## Immutabilité

- Privilégie les setters `init`, les paramètres de constructeur et les collections immuables pour l'état partagé
- Ne mute jamais les modèles d'entrée en place lors de la production d'un état mis à jour

```csharp
public sealed record UserProfile(string Name, string Email);

public static UserProfile Rename(UserProfile profile, string name) =>
    profile with { Name = name };
```

## Asynchronisme et gestion des erreurs

- Privilégie `async`/`await` aux appels bloquants comme `.Result` ou `.Wait()`
- Propage `CancellationToken` à travers les API asynchrones publiques
- Lève des exceptions spécifiques et journalise avec des propriétés structurées

```csharp
public async Task<Order> LoadOrderAsync(
    Guid orderId,
    CancellationToken cancellationToken)
{
    try
    {
        return await repository.FindAsync(orderId, cancellationToken)
            ?? throw new InvalidOperationException($"Order {orderId} was not found.");
    }
    catch (Exception ex)
    {
        logger.LogError(ex, "Failed to load order {OrderId}", orderId);
        throw;
    }
}
```

## Formatage

- Utilise `dotnet format` pour le formatage et les corrections d'analyseur
- Garde les directives `using` organisées et supprime les imports inutilisés
- Privilégie les membres à corps d'expression uniquement quand ils restent lisibles
