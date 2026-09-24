---
paths:
  - "**/*.swift"
  - "**/Package.swift"
---
# Patterns Swift

> Ce fichier étend [common/patterns.md](../common/patterns.md) avec du contenu spécifique à Swift.

## Conception orientée protocole

Définir des protocoles petits et ciblés. Utiliser des extensions de protocole pour les comportements par défaut partagés :

```swift
protocol Repository: Sendable {
    associatedtype Item: Identifiable & Sendable
    func find(by id: Item.ID) async throws -> Item?
    func save(_ item: Item) async throws
}
```

## Types valeur

- Utiliser des structs pour les objets de transfert de données et les modèles
- Utiliser des enums avec valeurs associées pour modéliser des états distincts :

```swift
enum LoadState<T: Sendable>: Sendable {
    case idle
    case loading
    case loaded(T)
    case failed(Error)
}
```

## Pattern Actor

Utiliser des acteurs (actors) pour l'état mutable partagé plutôt que des verrous ou des dispatch queues :

```swift
actor Cache<Key: Hashable & Sendable, Value: Sendable> {
    private var storage: [Key: Value] = [:]

    func get(_ key: Key) -> Value? { storage[key] }
    func set(_ key: Key, value: Value) { storage[key] = value }
}
```

## Injection de dépendances

Injecter des protocoles avec des paramètres par défaut — la production utilise les valeurs par défaut, les tests injectent des mocks :

```swift
struct UserService {
    private let repository: any UserRepository

    init(repository: any UserRepository = DefaultUserRepository()) {
        self.repository = repository
    }
}
```

## Références

Voir la skill : `swift-actor-persistence` pour les patterns de persistance basés sur les acteurs.
Voir la skill : `swift-protocol-di-testing` pour l'injection de dépendances et les tests basés sur les protocoles.
