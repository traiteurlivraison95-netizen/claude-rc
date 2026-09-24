---
paths:
  - "**/*.rs"
  - "**/Cargo.toml"
---
# Hooks Rust

> Ce fichier étend [common/hooks.md](../common/hooks.md) avec du contenu spécifique à Rust.

## Hooks PostToolUse

Configurer dans `~/.claude/settings.json` :

- **cargo fmt** : formater automatiquement les fichiers `.rs` après modification
- **cargo clippy** : exécuter les vérifications de lint après modification de fichiers Rust
- **cargo check** : vérifier la compilation après les changements (plus rapide que `cargo build`)
