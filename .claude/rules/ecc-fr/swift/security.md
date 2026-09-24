---
paths:
  - "**/*.swift"
  - "**/Package.swift"
---
# Sécurité Swift

> Ce fichier étend [common/security.md](../common/security.md) avec du contenu spécifique à Swift.

## Gestion des secrets

- Utiliser **Keychain Services** pour les données sensibles (tokens, mots de passe, clés) — jamais `UserDefaults`
- Utiliser des variables d'environnement ou des fichiers `.xcconfig` pour les secrets au moment du build
- Ne jamais coder en dur des secrets dans le code source — les outils de décompilation les extraient trivialement

```swift
let apiKey = ProcessInfo.processInfo.environment["API_KEY"]
guard let apiKey, !apiKey.isEmpty else {
    fatalError("API_KEY not configured")
}
```

## Sécurité du transport

- App Transport Security (ATS) est appliqué par défaut — ne pas le désactiver
- Utiliser le certificate pinning pour les endpoints critiques
- Valider tous les certificats serveur

## Validation des entrées

- Assainir toute entrée utilisateur avant affichage pour prévenir l'injection
- Utiliser `URL(string:)` avec validation plutôt que le force-unwrapping
- Valider les données provenant de sources externes (API, deep links, presse-papiers) avant traitement
