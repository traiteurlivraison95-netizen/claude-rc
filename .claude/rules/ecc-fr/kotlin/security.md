---
paths:
  - "**/*.kt"
  - "**/*.kts"
---
# Sécurité Kotlin

> Ce fichier étend [common/security.md](../common/security.md) avec du contenu spécifique à Kotlin et Android/KMP.

## Gestion des secrets

- Ne jamais coder en dur des clés API, jetons ou identifiants dans le code source
- Utiliser `local.properties` (ignoré par git) pour les secrets de développement local
- Utiliser des champs `BuildConfig` générés à partir des secrets CI pour les builds de release
- Utiliser `EncryptedSharedPreferences` (Android) ou Keychain (iOS) pour le stockage de secrets à l'exécution

```kotlin
// MAUVAIS
val apiKey = "sk-abc123..."

// BON — depuis BuildConfig (généré au moment du build)
val apiKey = BuildConfig.API_KEY

// BON — depuis un stockage sécurisé à l'exécution
val token = secureStorage.get("auth_token")
```

## Sécurité réseau

- Utiliser exclusivement HTTPS — configurer `network_security_config.xml` pour bloquer le trafic en clair
- Épingler les certificats pour les endpoints sensibles avec `CertificatePinner` d'OkHttp ou l'équivalent Ktor
- Définir des délais d'expiration sur tous les clients HTTP — ne jamais laisser les valeurs par défaut (qui peuvent être infinies)
- Valider et assainir toutes les réponses serveur avant utilisation

```xml
<!-- res/xml/network_security_config.xml -->
<network-security-config>
    <base-config cleartextTrafficPermitted="false" />
</network-security-config>
```

## Validation des entrées

- Valider toutes les entrées utilisateur avant traitement ou envoi à l'API
- Utiliser des requêtes paramétrées pour Room/SQLDelight — ne jamais concaténer d'entrées utilisateur dans du SQL
- Assainir les chemins de fichiers provenant des entrées utilisateur pour éviter le path traversal

```kotlin
// MAUVAIS — injection SQL
@Query("SELECT * FROM items WHERE name = '$input'")

// BON — paramétré
@Query("SELECT * FROM items WHERE name = :input")
fun findByName(input: String): List<ItemEntity>
```

## Protection des données

- Utiliser `EncryptedSharedPreferences` pour les données clé-valeur sensibles sur Android
- Utiliser `@Serializable` avec des noms de champs explicites — ne pas exposer les noms de propriétés internes
- Effacer les données sensibles de la mémoire lorsqu'elles ne sont plus nécessaires
- Utiliser `@Keep` ou des règles ProGuard pour les classes sérialisées afin d'éviter le name mangling

## Authentification

- Stocker les jetons dans un stockage sécurisé, pas dans de simples SharedPreferences
- Implémenter le rafraîchissement de jeton avec une gestion appropriée des 401/403
- Effacer tout l'état d'authentification à la déconnexion (jetons, données utilisateur en cache, cookies)
- Utiliser l'authentification biométrique (`BiometricPrompt`) pour les opérations sensibles

## ProGuard / R8

- Règles de conservation pour tous les modèles sérialisés (`@Serializable`, Gson, Moshi)
- Règles de conservation pour les bibliothèques basées sur la réflexion (Koin, Retrofit)
- Tester les builds de release — l'obfuscation peut casser la sérialisation silencieusement

## Sécurité WebView

- Désactiver JavaScript sauf si explicitement nécessaire : `settings.javaScriptEnabled = false`
- Valider les URL avant chargement dans une WebView
- Ne jamais exposer de méthodes `@JavascriptInterface` accédant à des données sensibles
- Utiliser `WebViewClient.shouldOverrideUrlLoading()` pour contrôler la navigation
