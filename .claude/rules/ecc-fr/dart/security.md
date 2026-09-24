---
paths:
  - "**/*.dart"
  - "**/pubspec.yaml"
  - "**/AndroidManifest.xml"
  - "**/Info.plist"
---
# Sécurité Dart/Flutter

> Ce fichier étend [common/security.md](../common/security.md) avec du contenu spécifique à Dart, Flutter et au mobile.

## Gestion des secrets

- Ne fige jamais en dur de clés d'API, jetons, ou identifiants dans le code source Dart
- Utilise `--dart-define` ou `--dart-define-from-file` pour la configuration au moment de la compilation (ces valeurs ne sont pas véritablement secrètes — utilise un proxy backend pour les secrets côté serveur)
- Utilise `flutter_dotenv` ou équivalent, avec des fichiers `.env` listés dans `.gitignore`
- Stocke les secrets d'exécution dans un stockage sécurisé de la plateforme : `flutter_secure_storage` (Keychain sur iOS, EncryptedSharedPreferences sur Android)

```dart
// MAUVAIS
const apiKey = 'sk-abc123...';

// BON — configuration au moment de la compilation (pas secrète, juste configurable)
const apiKey = String.fromEnvironment('API_KEY');

// BON — secret d'exécution depuis un stockage sécurisé
final token = await secureStorage.read(key: 'auth_token');
```

## Sécurité réseau

- Impose HTTPS — pas d'appels `http://` en production
- Configure `network_security_config.xml` sous Android pour bloquer le trafic en clair
- Définis `NSAppTransportSecurity` dans `Info.plist` pour interdire les chargements arbitraires
- Définis des timeouts de requête sur tous les clients HTTP — ne laisse jamais les valeurs par défaut
- Envisage le certificate pinning pour les endpoints à haute sécurité

```dart
// Dio avec timeout et imposition HTTPS
final dio = Dio(BaseOptions(
  baseUrl: 'https://api.example.com',
  connectTimeout: const Duration(seconds: 10),
  receiveTimeout: const Duration(seconds: 30),
));
```

## Validation des entrées

- Valide et sanitise toutes les entrées utilisateur avant de les envoyer à l'API ou au stockage
- Ne passe jamais d'entrée non sanitisée dans des requêtes SQL — utilise des requêtes paramétrées (sqflite, drift)
- Sanitise les URLs de deep link avant la navigation — valide le schéma, l'hôte et les paramètres de chemin
- Utilise `Uri.tryParse` et valide avant de naviguer

```dart
// MAUVAIS — injection SQL
await db.rawQuery("SELECT * FROM users WHERE email = '$userInput'");

// BON — paramétré
await db.query('users', where: 'email = ?', whereArgs: [userInput]);

// MAUVAIS — deep link non validé
final uri = Uri.parse(incomingLink);
context.go(uri.path); // pourrait naviguer vers n'importe quelle route

// BON — deep link validé
final uri = Uri.tryParse(incomingLink);
if (uri != null && uri.host == 'myapp.com' && _allowedPaths.contains(uri.path)) {
  context.go(uri.path);
}
```

## Protection des données

- Stocke les jetons, PII, et identifiants uniquement dans `flutter_secure_storage`
- N'écris jamais de données sensibles en clair dans `SharedPreferences` ou des fichiers locaux
- Efface l'état d'authentification à la déconnexion : jetons, données utilisateur en cache, cookies
- Utilise l'authentification biométrique (`local_auth`) pour les opérations sensibles
- Évite de journaliser des données sensibles — pas de `print(token)` ni `debugPrint(password)`

## Spécifique à Android

- Déclare uniquement les permissions requises dans `AndroidManifest.xml`
- N'exporte les composants Android (`Activity`, `Service`, `BroadcastReceiver`) que lorsque c'est nécessaire ; ajoute `android:exported="false"` là où ce n'est pas nécessaire
- Revois les filtres d'intent — les composants exportés avec des filtres d'intent implicites sont accessibles par n'importe quelle application
- Utilise `FLAG_SECURE` pour les écrans affichant des données sensibles (empêche les captures d'écran)

```xml
<!-- AndroidManifest.xml — restreindre les composants exportés -->
<activity android:name=".MainActivity" android:exported="true">
    <!-- Seule l'activité de lancement a besoin de exported=true -->
</activity>
<activity android:name=".SensitiveActivity" android:exported="false" />
```

## Spécifique à iOS

- Déclare uniquement les descriptions d'usage requises dans `Info.plist` (`NSCameraUsageDescription`, etc.)
- Stocke les secrets dans le Keychain — `flutter_secure_storage` utilise le Keychain sur iOS
- Utilise App Transport Security (ATS) — interdis les chargements arbitraires
- Active l'entitlement de protection des données pour les fichiers sensibles

## Sécurité des WebViews

- Utilise `webview_flutter` v4+ (`WebViewController` / `WebViewWidget`) — le widget `WebView` historique est supprimé
- Désactive JavaScript sauf si explicitement requis (`JavaScriptMode.disabled`)
- Valide les URLs avant chargement — ne charge jamais d'URLs arbitraires provenant de deep links
- N'expose jamais de callbacks Dart à JavaScript sauf absolue nécessité et isolation soigneuse
- Utilise `NavigationDelegate.onNavigationRequest` pour intercepter et valider les requêtes de navigation

```dart
// API webview_flutter v4+ (WebViewController + WebViewWidget)
final controller = WebViewController()
  ..setJavaScriptMode(JavaScriptMode.disabled) // désactivé sauf si requis
  ..setNavigationDelegate(
    NavigationDelegate(
      onNavigationRequest: (request) {
        final uri = Uri.tryParse(request.url);
        if (uri == null || uri.host != 'trusted.example.com') {
          return NavigationDecision.prevent;
        }
        return NavigationDecision.navigate;
      },
    ),
  );

// Dans ton arbre de widgets :
WebViewWidget(controller: controller)
```

## Obfuscation et sécurité du build

- Active l'obfuscation dans les builds de release : `flutter build apk --obfuscate --split-debug-info=./debug-info/`
- Garde la sortie de `--split-debug-info` hors du contrôle de version (utilisée uniquement pour la symbolisation des crashs)
- Assure-toi que les règles ProGuard/R8 n'exposent pas involontairement des classes sérialisées
- Exécute `flutter analyze` et traite tous les avertissements avant la release
