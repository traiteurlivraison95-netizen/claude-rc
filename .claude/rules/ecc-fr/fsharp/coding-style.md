---
paths:
  - "**/*.fs"
  - "**/*.fsx"
---
# Style de code F#

> Ce fichier étend [common/coding-style.md](../common/coding-style.md) avec du contenu spécifique à F#.

## Standards

- Suivre les conventions F# standard et exploiter le système de types pour garantir la correction
- Préférer l'immutabilité par défaut ; n'utiliser `mutable` que lorsque c'est justifié par la performance
- Garder les modules ciblés et cohérents

## Types et modèles

- Préférer les unions discriminées pour la modélisation de domaine plutôt que les hiérarchies de classes
- Utiliser des records pour les données avec des champs nommés
- Utiliser des unions à cas unique pour des wrappers de type sécurisé autour des types primitifs
- Éviter les classes sauf si l'interopérabilité ou un état mutable l'exige

```fsharp
type EmailAddress = EmailAddress of string

type OrderStatus =
    | Pending
    | Confirmed of confirmedAt: DateTimeOffset
    | Shipped of trackingNumber: string
    | Cancelled of reason: string

type Order =
    { Id: Guid
      CustomerId: string
      Status: OrderStatus
      Items: OrderItem list }
```

## Immutabilité

- Les records sont immuables par défaut ; utiliser les expressions `with` pour les mises à jour
- Préférer `list`, `map`, `set` aux collections mutables
- Éviter les cellules `ref` et les champs mutables dans la logique de domaine

```fsharp
let rename (profile: UserProfile) newName =
    { profile with Name = newName }
```

## Style de fonction

- Préférer des fonctions petites et composables plutôt que de grandes méthodes
- Utiliser l'opérateur pipe `|>` pour construire des pipelines de données lisibles
- Préférer le pattern matching aux chaînes if/else
- Utiliser `Option` au lieu de null ; utiliser `Result` pour les opérations qui peuvent échouer

```fsharp
let processOrder order =
    order
    |> validateItems
    |> Result.bind calculateTotal
    |> Result.map applyDiscount
    |> Result.mapError OrderError
```

## Asynchrone et gestion des erreurs

- Utiliser `task { }` pour l'interopérabilité avec les API async .NET
- Utiliser `async { }` pour les workflows async natifs F#
- Propager `CancellationToken` à travers les API async publiques
- Préférer `Result` et la programmation orientée rails (railway-oriented programming) aux exceptions pour les échecs attendus

```fsharp
let loadOrderAsync (orderId: Guid) (ct: CancellationToken) =
    task {
        let! order = repository.FindAsync(orderId, ct)
        return
            order
            |> Option.defaultWith (fun () ->
                failwith $"Order {orderId} was not found.")
    }
```

## Formatage

- Utiliser `fantomas` pour le formatage automatique
- Préférer les espaces significatifs ; éviter les parenthèses inutiles
- Supprimer les déclarations `open` inutilisées

### Ordre des déclarations Open

Regrouper les instructions `open` en quatre sections séparées par une ligne vide, chaque section triée lexicalement en son sein :

1. `System.*`
2. `Microsoft.*`
3. Espaces de noms tiers
4. Espaces de noms de première partie / du projet

```fsharp
open System
open System.Collections.Generic
open System.Threading.Tasks

open Microsoft.AspNetCore.Http
open Microsoft.Extensions.Logging

open FsCheck.Xunit
open Swensen.Unquote

open MyApp.Domain
open MyApp.Infrastructure
```
