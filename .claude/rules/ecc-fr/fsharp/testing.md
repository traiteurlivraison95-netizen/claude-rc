---
paths:
  - "**/*.fs"
  - "**/*.fsx"
  - "**/*.fsproj"
---
# Tests F#

> Ce fichier étend [common/testing.md](../common/testing.md) avec du contenu spécifique à F#.

## Framework de test

- Préférer **xUnit** avec **FsUnit.xUnit** pour des assertions adaptées à F#
- Utiliser **Unquote** pour des assertions basées sur les quotations avec des messages d'échec clairs
- Utiliser **FsCheck.xUnit** pour les tests basés sur les propriétés (property-based testing)
- Utiliser **NSubstitute** ou des stubs de fonctions pour simuler les dépendances
- Utiliser **Testcontainers** lorsque les tests d'intégration nécessitent une infrastructure réelle

## Organisation des tests

- Reproduire la structure de `src/` sous `tests/`
- Séparer clairement la couverture unitaire, d'intégration et de bout en bout
- Nommer les tests selon le comportement, pas les détails d'implémentation

```fsharp
open Xunit
open Swensen.Unquote

[<Fact>]
let ``PlaceOrder returns success when request is valid`` () =
    let request = { CustomerId = "cust-123"; Items = [ validItem ] }
    let result = OrderService.placeOrder request
    test <@ Result.isOk result @>

[<Fact>]
let ``PlaceOrder returns error when items are empty`` () =
    let request = { CustomerId = "cust-123"; Items = [] }
    let result = OrderService.placeOrder request
    test <@ Result.isError result @>
```

## Tests basés sur les propriétés avec FsCheck

```fsharp
open FsCheck.Xunit

[<Property>]
let ``order total is never negative`` (items: OrderItem list) =
    let total = Order.calculateTotal items
    total >= 0m
```

## Tests d'intégration ASP.NET Core

- Utiliser `WebApplicationFactory<TEntryPoint>` pour la couverture d'intégration de l'API
- Tester l'authentification, la validation et la sérialisation via HTTP, sans contourner le middleware

## Couverture

- Viser 80 %+ de couverture de lignes
- Concentrer la couverture sur la logique de domaine, la validation, l'authentification et les chemins d'échec
- Exécuter `dotnet test` en CI avec la collecte de couverture activée lorsque disponible
