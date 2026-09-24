---
paths:
  - "**/*.rs"
---
# Style de code Rust

> Ce fichier étend [common/coding-style.md](../common/coding-style.md) avec du contenu spécifique à Rust.

## Formatage

- **rustfmt** pour l'application — toujours exécuter `cargo fmt` avant de commiter
- **clippy** pour les lints — `cargo clippy -- -D warnings` (traiter les avertissements comme des erreurs)
- Indentation de 4 espaces (défaut rustfmt)
- Largeur de ligne max : 100 caractères (défaut rustfmt)

## Immutabilité

Les variables Rust sont immuables par défaut — adoptez cela :

- Utiliser `let` par défaut ; n'utiliser `let mut` que lorsque la mutation est nécessaire
- Préférer retourner de nouvelles valeurs plutôt que muter sur place
- Utiliser `Cow<'_, T>` quand une fonction peut ou non avoir besoin d'allouer

```rust
use std::borrow::Cow;

// BON — immuable par défaut, nouvelle valeur retournée
fn normalize(input: &str) -> Cow<'_, str> {
    if input.contains(' ') {
        Cow::Owned(input.replace(' ', "_"))
    } else {
        Cow::Borrowed(input)
    }
}

// MAUVAIS — mutation inutile
fn normalize_bad(input: &mut String) {
    *input = input.replace(' ', "_");
}
```

## Nommage

Suivre les conventions Rust standard :
- `snake_case` pour les fonctions, méthodes, variables, modules, crates
- `PascalCase` (UpperCamelCase) pour les types, traits, enums, paramètres de type
- `SCREAMING_SNAKE_CASE` pour les constantes et statics
- Lifetimes : minuscules courtes (`'a`, `'de`) — noms descriptifs pour les cas complexes (`'input`)

## Ownership et emprunt (Borrowing)

- Emprunter (`&T`) par défaut ; ne prendre possession que lorsqu'il faut stocker ou consommer
- Ne jamais cloner juste pour satisfaire le borrow checker sans comprendre la cause racine
- Accepter `&str` plutôt que `String`, `&[T]` plutôt que `Vec<T>` dans les paramètres de fonction
- Utiliser `impl Into<String>` pour les constructeurs qui doivent posséder une `String`

```rust
// BON — emprunte quand la possession n'est pas nécessaire
fn word_count(text: &str) -> usize {
    text.split_whitespace().count()
}

// BON — prend possession dans le constructeur via Into
fn new(name: impl Into<String>) -> Self {
    Self { name: name.into() }
}

// MAUVAIS — prend une String alors que &str suffit
fn word_count_bad(text: String) -> usize {
    text.split_whitespace().count()
}
```

## Gestion des erreurs

- Utiliser `Result<T, E>` et `?` pour la propagation — jamais `unwrap()` en code de production
- **Bibliothèques** : définir des erreurs typées avec `thiserror`
- **Applications** : utiliser `anyhow` pour un contexte d'erreur flexible
- Ajouter du contexte avec `.with_context(|| format!("failed to ..."))?`
- Réserver `unwrap()` / `expect()` aux tests et aux états véritablement inatteignables

```rust
// BON — erreur de bibliothèque avec thiserror
#[derive(Debug, thiserror::Error)]
pub enum ConfigError {
    #[error("failed to read config: {0}")]
    Io(#[from] std::io::Error),
    #[error("invalid config format: {0}")]
    Parse(String),
}

// BON — erreur d'application avec anyhow
use anyhow::Context;

fn load_config(path: &str) -> anyhow::Result<Config> {
    let content = std::fs::read_to_string(path)
        .with_context(|| format!("failed to read {path}"))?;
    toml::from_str(&content)
        .with_context(|| format!("failed to parse {path}"))
}
```

## Itérateurs plutôt que boucles

Préférer les chaînes d'itérateurs pour les transformations ; utiliser les boucles pour le flux de contrôle complexe :

```rust
// BON — déclaratif et composable
let active_emails: Vec<&str> = users.iter()
    .filter(|u| u.is_active)
    .map(|u| u.email.as_str())
    .collect();

// BON — boucle pour une logique complexe avec retours anticipés
for user in &users {
    if let Some(verified) = verify_email(&user.email)? {
        send_welcome(&verified)?;
    }
}
```

## Organisation des modules

Organiser par domaine, pas par type :

```text
src/
├── main.rs
├── lib.rs
├── auth/           # Module de domaine
│   ├── mod.rs
│   ├── token.rs
│   └── middleware.rs
├── orders/         # Module de domaine
│   ├── mod.rs
│   ├── model.rs
│   └── service.rs
└── db/             # Infrastructure
    ├── mod.rs
    └── pool.rs
```

## Visibilité

- Privé par défaut ; utiliser `pub(crate)` pour le partage interne
- Ne marquer `pub` que ce qui fait partie de l'API publique de la crate
- Ré-exporter l'API publique depuis `lib.rs`

## Références

Voir la skill : `rust-patterns` pour des idiomes et patterns Rust complets.
