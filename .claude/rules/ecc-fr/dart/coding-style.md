---
paths:
  - "**/*.dart"
  - "**/pubspec.yaml"
  - "**/analysis_options.yaml"
---
# Style de code Dart/Flutter

> Ce fichier étend [common/coding-style.md](../common/coding-style.md) avec du contenu spécifique à Dart et Flutter.

## Formatage

- **dart format** pour tous les fichiers `.dart` — imposé en CI (`dart format --set-exit-if-changed .`)
- Longueur de ligne : 80 caractères (par défaut de dart format)
- Virgules finales sur les listes d'arguments/paramètres multi-lignes pour améliorer les diffs et le formatage

## Immutabilité

- Privilégie `final` pour les variables locales et `const` pour les constantes évaluées à la compilation
- Utilise des constructeurs `const` partout où tous les champs sont `final`
- Retourne des collections non modifiables depuis les API publiques (`List.unmodifiable`, `Map.unmodifiable`)
- Utilise `copyWith()` pour les mutations d'état dans les classes d'état immuables

```dart
// MAUVAIS
var count = 0;
List<String> items = ['a', 'b'];

// BON
final count = 0;
const items = ['a', 'b'];
```

## Nommage

Suis les conventions Dart :
- `camelCase` pour les variables, paramètres et constructeurs nommés
- `PascalCase` pour les classes, enums, typedefs et extensions
- `snake_case` pour les noms de fichiers et de bibliothèques
- `SCREAMING_SNAKE_CASE` pour les constantes déclarées avec `const` au niveau supérieur
- Préfixe les membres privés avec `_`
- Les noms d'extension décrivent le type qu'ils étendent : `StringExtensions`, pas `MyHelpers`

## Null safety

- Évite `!` (opérateur bang) — privilégie `?.`, `??`, `if (x != null)`, ou le pattern matching Dart 3 ; réserve `!` uniquement quand une valeur null est une erreur de programmation et qu'un plantage est le comportement attendu
- Évite `late` sauf si l'initialisation est garantie avant la première utilisation (privilégie nullable ou l'initialisation par constructeur)
- Utilise `required` pour les paramètres de constructeur qui doivent toujours être fournis

```dart
// MAUVAIS — plante à l'exécution si user est null
final name = user!.name;

// BON — opérateurs null-aware
final name = user?.name ?? 'Unknown';

// BON — pattern matching Dart 3 (exhaustif, vérifié par le compilateur)
final name = switch (user) {
  User(:final name) => name,
  null => 'Unknown',
};

// BON — garde de retour anticipé sur null
String getUserName(User? user) {
  if (user == null) return 'Unknown';
  return user.name; // promu en non-null après la garde
}
```

## Types scellés et pattern matching (Dart 3+)

Utilise des classes scellées pour modéliser des hiérarchies d'état fermées :

```dart
sealed class AsyncState<T> {
  const AsyncState();
}

final class Loading<T> extends AsyncState<T> {
  const Loading();
}

final class Success<T> extends AsyncState<T> {
  const Success(this.data);
  final T data;
}

final class Failure<T> extends AsyncState<T> {
  const Failure(this.error);
  final Object error;
}
```

Utilise toujours un `switch` exhaustif avec les types scellés — pas de default/wildcard :

```dart
// MAUVAIS
if (state is Loading) { ... }

// BON
return switch (state) {
  Loading() => const CircularProgressIndicator(),
  Success(:final data) => DataWidget(data),
  Failure(:final error) => ErrorWidget(error.toString()),
};
```

## Gestion des erreurs

- Précise les types d'exception dans les clauses `on` — n'utilise jamais de `catch (e)` nu
- Ne capture jamais les sous-types d'`Error` — ils indiquent des bugs de programmation
- Utilise des types de style `Result` ou des classes scellées pour les erreurs récupérables
- Évite d'utiliser les exceptions pour le contrôle de flux

```dart
// MAUVAIS
try {
  await fetchUser();
} catch (e) {
  log(e.toString());
}

// BON
try {
  await fetchUser();
} on NetworkException catch (e) {
  log('Network error: ${e.message}');
} on NotFoundException {
  handleNotFound();
}
```

## Async / Futures

- Fais toujours `await` sur les Futures ou appelle explicitement `unawaited()` pour signaler un fire-and-forget intentionnel
- Ne marque jamais une fonction `async` si elle ne fait jamais `await`
- Utilise `Future.wait` / `Future.any` pour les opérations concurrentes
- Vérifie `context.mounted` avant d'utiliser `BuildContext` après tout `await` (Flutter 3.7+)

```dart
// MAUVAIS — Future ignorée
fetchData(); // fire-and-forget sans marquer l'intention

// BON
unawaited(fetchData()); // fire-and-forget explicite
await fetchData();      // ou correctement attendue
```

## Imports

- Utilise les imports `package:` partout — jamais d'imports relatifs (`../`) pour du code inter-fonctionnalité ou inter-couche
- Ordre : `dart:` → `package:` externe → `package:` interne (même package)
- Aucun import inutilisé — `dart analyze` l'impose avec `unused_import`

## Génération de code

- Les fichiers générés (`.g.dart`, `.freezed.dart`, `.gr.dart`) doivent être commités ou ignorés de façon cohérente — choisis une stratégie par projet
- Ne modifie jamais manuellement les fichiers générés
- Garde les annotations du générateur (`@JsonSerializable`, `@freezed`, `@riverpod`, etc.) uniquement sur le fichier source canonique
