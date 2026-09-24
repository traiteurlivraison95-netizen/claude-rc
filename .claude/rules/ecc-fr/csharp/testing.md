---
paths:
  - "**/*.cs"
  - "**/*.csx"
  - "**/*.csproj"
---
# Tests C#

> Ce fichier étend [common/testing.md](../common/testing.md) avec du contenu spécifique à C#.

## Framework de test

- Privilégie **xUnit** pour les tests unitaires et d'intégration
- Utilise **FluentAssertions** pour des assertions lisibles
- Utilise **Moq** ou **NSubstitute** pour simuler les dépendances
- Utilise **Testcontainers** quand les tests d'intégration nécessitent une infrastructure réelle

## Organisation des tests

- Reflète la structure de `src/` sous `tests/`
- Sépare clairement la couverture unitaire, d'intégration et de bout en bout
- Nomme les tests par comportement, pas par détails d'implémentation

```csharp
public sealed class OrderServiceTests
{
    [Fact]
    public async Task FindByIdAsync_ReturnsOrder_WhenOrderExists()
    {
        // Arrange
        // Act
        // Assert
    }
}
```

## Tests d'intégration ASP.NET Core

- Utilise `WebApplicationFactory<TEntryPoint>` pour la couverture d'intégration d'API
- Teste l'authentification, la validation et la sérialisation via HTTP, pas en contournant le middleware

## Couverture

- Vise 80%+ de couverture de lignes
- Concentre la couverture sur la logique métier, la validation, l'authentification et les chemins d'échec
- Exécute `dotnet test` en CI avec la collecte de couverture activée quand disponible
