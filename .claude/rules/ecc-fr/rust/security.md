---
paths:
  - "**/*.rs"
---
# Sécurité Rust

> Ce fichier étend [common/security.md](../common/security.md) avec du contenu spécifique à Rust.

## Gestion des secrets

- Ne jamais coder en dur les clés API, tokens, ou credentials dans le code source
- Utiliser des variables d'environnement : `std::env::var("API_KEY")`
- Échouer rapidement si des secrets requis manquent au démarrage
- Garder les fichiers `.env` dans `.gitignore`

```rust
// MAUVAIS
const API_KEY: &str = "sk-abc123...";

// BON — variable d'environnement avec validation précoce
fn load_api_key() -> anyhow::Result<String> {
    std::env::var("PAYMENT_API_KEY")
        .context("PAYMENT_API_KEY must be set")
}
```

## Prévention de l'injection SQL

- Toujours utiliser des requêtes paramétrées — ne jamais formater l'entrée utilisateur dans des chaînes SQL
- Utiliser un query builder ou un ORM (sqlx, diesel, sea-orm) avec des paramètres liés

```rust
// MAUVAIS — injection SQL via chaîne de format
let query = format!("SELECT * FROM users WHERE name = '{name}'");
sqlx::query(&query).fetch_one(&pool).await?;

// BON — requête paramétrée avec sqlx
// La syntaxe des placeholders varie selon le backend : Postgres : $1 | MySQL : ? | SQLite : $1
sqlx::query("SELECT * FROM users WHERE name = $1")
    .bind(&name)
    .fetch_one(&pool)
    .await?;
```

## Validation des entrées

- Valider toute entrée utilisateur aux frontières du système avant traitement
- Utiliser le système de types pour imposer les invariants (pattern newtype)
- Parser, ne pas valider — convertir les données non structurées en structs typées à la frontière
- Rejeter les entrées invalides avec des messages d'erreur clairs

```rust
// Parser, ne pas valider — les états invalides sont irreprésentables
pub struct Email(String);

impl Email {
    pub fn parse(input: &str) -> Result<Self, ValidationError> {
        let trimmed = input.trim();
        let at_pos = trimmed.find('@')
            .filter(|&p| p > 0 && p < trimmed.len() - 1)
            .ok_or_else(|| ValidationError::InvalidEmail(input.to_string()))?;
        let domain = &trimmed[at_pos + 1..];
        if trimmed.len() > 254 || !domain.contains('.') {
            return Err(ValidationError::InvalidEmail(input.to_string()));
        }
        // Pour un usage en production, préférer une crate email validée (par ex. `email_address`)
        Ok(Self(trimmed.to_string()))
    }

    pub fn as_str(&self) -> &str {
        &self.0
    }
}
```

## Code unsafe

- Minimiser les blocs `unsafe` — préférer les abstractions sûres
- Chaque bloc `unsafe` doit avoir un commentaire `// SAFETY:` expliquant l'invariant
- Ne jamais utiliser `unsafe` pour contourner le borrow checker par commodité
- Auditer tout code `unsafe` en revue — c'est un signal d'alarme sans justification
- Préférer des wrappers FFI `safe` autour des bibliothèques C

```rust
// BON — le commentaire de sécurité documente TOUS les invariants requis
let widget: &Widget = {
    // SAFETY: `ptr` est non-null, aligné, pointe vers un Widget initialisé,
    // et aucune référence mutable ni mutation n'existe pendant sa durée de vie.
    unsafe { &*ptr }
};

// MAUVAIS — aucune justification de sécurité
unsafe { &*ptr }
```

## Sécurité des dépendances

- Exécuter `cargo audit` pour scanner les CVE connues dans les dépendances
- Exécuter `cargo deny check` pour la conformité des licences et des avis
- Utiliser `cargo tree` pour auditer les dépendances transitives
- Garder les dépendances à jour — mettre en place Dependabot ou Renovate
- Minimiser le nombre de dépendances — évaluer avant d'ajouter de nouvelles crates

```bash
# Audit de sécurité
cargo audit

# Refuser les avis, versions dupliquées, et licences restreintes
cargo deny check

# Inspecter l'arbre de dépendances
cargo tree
cargo tree -d  # N'afficher que les doublons
```

## Messages d'erreur

- Ne jamais exposer les chemins internes, traces de pile, ou erreurs de base de données dans les réponses API
- Journaliser les erreurs détaillées côté serveur ; retourner des messages génériques aux clients
- Utiliser `tracing` ou `log` pour la journalisation structurée côté serveur

```rust
// Mapper les erreurs vers des codes de statut appropriés et des messages génériques
// (l'exemple utilise axum ; adapter le type de réponse à votre framework)
match order_service.find_by_id(id) {
    Ok(order) => Ok((StatusCode::OK, Json(order))),
    Err(ServiceError::NotFound(_)) => {
        tracing::info!(order_id = id, "order not found");
        Err((StatusCode::NOT_FOUND, "Resource not found"))
    }
    Err(e) => {
        tracing::error!(order_id = id, error = %e, "unexpected error");
        Err((StatusCode::INTERNAL_SERVER_ERROR, "Internal server error"))
    }
}
```

## Références

Voir la skill : `rust-patterns` pour les directives sur le code unsafe et les patterns d'ownership.
Voir la skill : `security-review` pour des checklists de sécurité générales.
