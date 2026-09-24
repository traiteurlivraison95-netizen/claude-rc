---
paths:
  - "**/*.rs"
---
# Tests Rust

> Ce fichier étend [common/testing.md](../common/testing.md) avec du contenu spécifique à Rust.

## Framework de test

- **`#[test]`** avec des modules `#[cfg(test)]` pour les tests unitaires
- **rstest** pour les tests paramétrés et les fixtures
- **proptest** pour les tests basés sur les propriétés
- **mockall** pour le mocking basé sur les traits
- **`#[tokio::test]`** pour les tests async

## Organisation des tests

```text
my_crate/
├── src/
│   ├── lib.rs           # Tests unitaires dans des modules #[cfg(test)]
│   ├── auth/
│   │   └── mod.rs       # #[cfg(test)] mod tests { ... }
│   └── orders/
│       └── service.rs   # #[cfg(test)] mod tests { ... }
├── tests/               # Tests d'intégration (chaque fichier = binaire séparé)
│   ├── api_test.rs
│   ├── db_test.rs
│   └── common/          # Utilitaires de test partagés
│       └── mod.rs
└── benches/             # Benchmarks Criterion
    └── benchmark.rs
```

Les tests unitaires vont dans des modules `#[cfg(test)]` dans le même fichier. Les tests d'intégration vont dans `tests/`.

## Pattern de test unitaire

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn creates_user_with_valid_email() {
        let user = User::new("Alice", "alice@example.com").unwrap();
        assert_eq!(user.name, "Alice");
    }

    #[test]
    fn rejects_invalid_email() {
        let result = User::new("Bob", "not-an-email");
        assert!(result.is_err());
        assert!(result.unwrap_err().to_string().contains("invalid email"));
    }
}
```

## Tests paramétrés

```rust
use rstest::rstest;

#[rstest]
#[case("hello", 5)]
#[case("", 0)]
#[case("rust", 4)]
fn test_string_length(#[case] input: &str, #[case] expected: usize) {
    assert_eq!(input.len(), expected);
}
```

## Tests async

```rust
#[tokio::test]
async fn fetches_data_successfully() {
    let client = TestClient::new().await;
    let result = client.get("/data").await;
    assert!(result.is_ok());
}
```

## Mocking avec mockall

Définir les traits dans le code de production ; générer les mocks dans les modules de test :

```rust
// Trait de production — pub pour que les tests d'intégration puissent l'importer
pub trait UserRepository {
    fn find_by_id(&self, id: u64) -> Option<User>;
}

#[cfg(test)]
mod tests {
    use super::*;
    use mockall::predicate::eq;

    mockall::mock! {
        pub Repo {}
        impl UserRepository for Repo {
            fn find_by_id(&self, id: u64) -> Option<User>;
        }
    }

    #[test]
    fn service_returns_user_when_found() {
        let mut mock = MockRepo::new();
        mock.expect_find_by_id()
            .with(eq(42))
            .times(1)
            .returning(|_| Some(User { id: 42, name: "Alice".into() }));

        let service = UserService::new(Box::new(mock));
        let user = service.get_user(42).unwrap();
        assert_eq!(user.name, "Alice");
    }
}
```

## Nommage des tests

Utiliser des noms descriptifs qui expliquent le scénario :
- `creates_user_with_valid_email()`
- `rejects_order_when_insufficient_stock()`
- `returns_none_when_not_found()`

## Couverture

- Viser 80%+ de couverture de lignes
- Utiliser **cargo-llvm-cov** pour le rapport de couverture
- Se concentrer sur la logique métier — exclure le code généré et les bindings FFI

```bash
cargo llvm-cov                       # Résumé
cargo llvm-cov --html                # Rapport HTML
cargo llvm-cov --fail-under-lines 80 # Échouer si en dessous du seuil
```

## Commandes de test

```bash
cargo test                       # Exécuter tous les tests
cargo test -- --nocapture        # Afficher la sortie println
cargo test test_name             # Exécuter les tests correspondant au motif
cargo test --lib                 # Tests unitaires uniquement
cargo test --test api_test       # Test d'intégration spécifique (tests/api_test.rs)
cargo test --doc                 # Doc tests uniquement
```

## Références

Voir la skill : `rust-testing` pour des patterns de test complets incluant les tests basés sur les propriétés, les fixtures, et le benchmarking avec Criterion.
