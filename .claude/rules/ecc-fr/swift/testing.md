---
paths:
  - "**/*.swift"
  - "**/Package.swift"
---
# Tests Swift

> Ce fichier étend [common/testing.md](../common/testing.md) avec du contenu spécifique à Swift.

## Framework

Utiliser **Swift Testing** (`import Testing`) pour les nouveaux tests. Utiliser `@Test` et `#expect` :

```swift
@Test("User creation validates email")
func userCreationValidatesEmail() throws {
    #expect(throws: ValidationError.invalidEmail) {
        try User(email: "not-an-email")
    }
}
```

## Isolation des tests

Chaque test obtient une instance fraîche — initialiser dans `init`, nettoyer dans `deinit`. Pas d'état mutable partagé entre les tests.

## Tests paramétrés

```swift
@Test("Validates formats", arguments: ["json", "xml", "csv"])
func validatesFormat(format: String) throws {
    let parser = try Parser(format: format)
    #expect(parser.isValid)
}
```

## Couverture

```bash
swift test --enable-code-coverage
```

## Référence

Voir la skill : `swift-protocol-di-testing` pour l'injection de dépendances basée sur les protocoles et les patterns de mock avec Swift Testing.
