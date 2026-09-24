---
paths:
  - "**/*.swift"
  - "**/Package.swift"
---
# Style de code Swift

> Ce fichier étend [common/coding-style.md](../common/coding-style.md) avec du contenu spécifique à Swift.

## Formatage

- **SwiftFormat** pour le formatage automatique, **SwiftLint** pour l'application du style
- `swift-format` est fourni avec Xcode 16+ comme alternative

## Immutabilité

- Préférer `let` à `var` — définir tout en `let` et ne passer à `var` que si le compilateur l'exige
- Utiliser `struct` avec la sémantique de valeur par défaut ; n'utiliser `class` que lorsque l'identité ou la sémantique de référence est nécessaire

## Nommage

Suivre les [Apple API Design Guidelines](https://www.swift.org/documentation/api-design-guidelines/) :

- Clarté au point d'utilisation — omettre les mots inutiles
- Nommer les méthodes et propriétés d'après leur rôle, pas d'après leur type
- Utiliser `static let` pour les constantes plutôt que des constantes globales

## Gestion des erreurs

Utiliser les throws typés (Swift 6+) et le pattern matching :

```swift
func load(id: String) throws(LoadError) -> Item {
    guard let data = try? read(from: path) else {
        throw .fileNotFound(id)
    }
    return try decode(data)
}
```

## Concurrence

Activer la vérification stricte de la concurrence Swift 6. Préférer :

- Les types valeur `Sendable` pour les données traversant les frontières d'isolation
- Les acteurs (actors) pour l'état mutable partagé
- La concurrence structurée (`async let`, `TaskGroup`) plutôt que `Task {}` non structuré
