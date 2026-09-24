---
paths:
  - "**/*.fs"
  - "**/*.fsx"
---
# Patterns F#

> Ce fichier étend [common/patterns.md](../common/patterns.md) avec du contenu spécifique à F#.

## Type Result pour la gestion des erreurs

Utiliser `Result<'T, 'TError>` avec la programmation orientée rails (railway-oriented programming) plutôt que des exceptions pour les échecs attendus.

```fsharp
type OrderError =
    | InvalidCustomer of string
    | EmptyItems
    | ItemOutOfStock of sku: string

let validateOrder (request: CreateOrderRequest) : Result<ValidatedOrder, OrderError> =
    if String.IsNullOrWhiteSpace request.CustomerId then
        Error(InvalidCustomer "CustomerId is required")
    elif request.Items |> List.isEmpty then
        Error EmptyItems
    else
        Ok { CustomerId = request.CustomerId; Items = request.Items }
```

## Option pour les valeurs manquantes

Préférer `Option<'T>` à null. Utiliser `Option.map`, `Option.bind` et `Option.defaultValue` pour transformer.

```fsharp
let findUser (id: Guid) : User option =
    users |> Map.tryFind id

let getUserEmail userId =
    findUser userId
    |> Option.map (fun u -> u.Email)
    |> Option.defaultValue "unknown@example.com"
```

## Unions discriminées pour la modélisation de domaine

Modéliser explicitement les états métier. Le compilateur impose une gestion exhaustive.

```fsharp
type PaymentState =
    | AwaitingPayment of amount: decimal
    | Paid of paidAt: DateTimeOffset * transactionId: string
    | Refunded of refundedAt: DateTimeOffset * reason: string
    | Failed of error: string

let describePayment = function
    | AwaitingPayment amount -> $"Awaiting payment of {amount:C}"
    | Paid (at, txn) -> $"Paid at {at} (txn: {txn})"
    | Refunded (at, reason) -> $"Refunded at {at}: {reason}"
    | Failed error -> $"Payment failed: {error}"
```

## Expressions de calcul (Computation Expressions)

Utiliser les expressions de calcul pour simplifier les opérations séquentielles pouvant échouer.

```fsharp
let placeOrder request =
    result {
        let! validated = validateOrder request
        let! inventory = checkInventory validated.Items
        let! order = createOrder validated inventory
        return order
    }
```

## Organisation des modules

- Regrouper les fonctions liées dans des modules plutôt que des classes
- Utiliser `[<RequireQualifiedAccess>]` pour éviter les collisions de noms
- Garder les modules petits et centrés sur une seule responsabilité

```fsharp
[<RequireQualifiedAccess>]
module Order =
    let create customerId items = { Id = Guid.NewGuid(); CustomerId = customerId; Items = items; Status = Pending }
    let confirm order = { order with Status = Confirmed(DateTimeOffset.UtcNow) }
    let cancel reason order = { order with Status = Cancelled reason }
```

## Injection de dépendances

- Définir les dépendances comme des paramètres de fonction ou des records de fonctions
- Utiliser les interfaces avec parcimonie, principalement à la frontière avec les bibliothèques .NET
- Préférer l'application partielle pour injecter des dépendances dans les pipelines

```fsharp
type OrderDeps =
    { FindOrder: Guid -> Task<Order option>
      SaveOrder: Order -> Task<unit>
      SendNotification: Order -> Task<unit> }

let processOrder (deps: OrderDeps) orderId =
    task {
        match! deps.FindOrder orderId with
        | None -> return Error "Order not found"
        | Some order ->
            let confirmed = Order.confirm order
            do! deps.SaveOrder confirmed
            do! deps.SendNotification confirmed
            return Ok confirmed
    }
```
