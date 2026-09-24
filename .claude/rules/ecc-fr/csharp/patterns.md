---
paths:
  - "**/*.cs"
  - "**/*.csx"
---
# Patterns C#

> Ce fichier étend [common/patterns.md](../common/patterns.md) avec du contenu spécifique à C#.

## Pattern de réponse API

```csharp
public sealed record ApiResponse<T>(
    bool Success,
    T? Data = default,
    string? Error = null,
    object? Meta = null);
```

## Pattern Repository

```csharp
public interface IRepository<T>
{
    Task<IReadOnlyList<T>> FindAllAsync(CancellationToken cancellationToken);
    Task<T?> FindByIdAsync(Guid id, CancellationToken cancellationToken);
    Task<T> CreateAsync(T entity, CancellationToken cancellationToken);
    Task<T> UpdateAsync(T entity, CancellationToken cancellationToken);
    Task DeleteAsync(Guid id, CancellationToken cancellationToken);
}
```

## Pattern Options

Utilise des options fortement typées pour la configuration plutôt que de lire des chaînes brutes dans toute la base de code.

```csharp
public sealed class PaymentsOptions
{
    public const string SectionName = "Payments";
    public required string BaseUrl { get; init; }
    public required string ApiKeySecretName { get; init; }
}
```

## Injection de dépendances

- Dépends des interfaces au niveau des frontières de service
- Garde les constructeurs ciblés ; si un service nécessite trop de dépendances, sépare les responsabilités
- Enregistre les durées de vie intentionnellement : singleton pour les services sans état/partagés, scoped pour les données de requête, transient pour les workers purs et légers
