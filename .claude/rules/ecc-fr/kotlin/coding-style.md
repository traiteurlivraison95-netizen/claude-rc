---
paths:
  - "**/*.kt"
  - "**/*.kts"
---
# Style de code Kotlin

> Ce fichier étend [common/coding-style.md](../common/coding-style.md) avec du contenu spécifique à Kotlin.

## Formatage

- **ktlint** ou **Detekt** pour l'application du style
- Style de code Kotlin officiel (`kotlin.code.style=official` dans `gradle.properties`)

## Immutabilité

- Préférer `val` à `var` — utiliser `val` par défaut et n'utiliser `var` que lorsque la mutation est requise
- Utiliser `data class` pour les types de valeur ; utiliser des collections immuables (`List`, `Map`, `Set`) dans les API publiques
- Copy-on-write pour les mises à jour d'état : `state.copy(field = newValue)`

## Nommage

Suivre les conventions Kotlin :
- `camelCase` pour les fonctions et propriétés
- `PascalCase` pour les classes, interfaces, objects et alias de type
- `SCREAMING_SNAKE_CASE` pour les constantes (`const val` ou `@JvmStatic`)
- Préfixer les interfaces par un comportement, pas par `I` : `Clickable` et non `IClickable`

## Sécurité des null (Null Safety)

- Ne jamais utiliser `!!` — préférer `?.`, `?:`, `requireNotNull()`, ou `checkNotNull()`
- Utiliser `?.let {}` pour les opérations null-safe scopées
- Retourner des types nullables depuis les fonctions qui peuvent légitimement n'avoir aucun résultat

```kotlin
// MAUVAIS
val name = user!!.name

// BON
val name = user?.name ?: "Unknown"
val name = requireNotNull(user) { "User must be set before accessing name" }.name
```

## Types Sealed

Utiliser des classes/interfaces sealed pour modéliser des hiérarchies d'état fermées :

```kotlin
sealed interface UiState<out T> {
    data object Loading : UiState<Nothing>
    data class Success<T>(val data: T) : UiState<T>
    data class Error(val message: String) : UiState<Nothing>
}
```

Toujours utiliser un `when` exhaustif avec les types sealed — pas de branche `else`.

## Fonctions d'extension

Utiliser des fonctions d'extension pour les opérations utilitaires, mais les garder découvrables :
- Les placer dans un fichier nommé d'après le type récepteur (`StringExt.kt`, `FlowExt.kt`)
- Garder la portée limitée — ne pas ajouter d'extensions à `Any` ou à des types trop génériques

## Fonctions de portée (Scope Functions)

Utiliser la bonne fonction de portée :
- `let` — vérification de nullité + transformation : `user?.let { greet(it) }`
- `run` — calculer un résultat en utilisant le récepteur : `service.run { fetch(config) }`
- `apply` — configurer un objet : `builder.apply { timeout = 30 }`
- `also` — effets de bord : `result.also { log(it) }`
- Éviter l'imbrication profonde de fonctions de portée (2 niveaux max)

## Gestion des erreurs

- Utiliser `Result<T>` ou des types sealed personnalisés
- Utiliser `runCatching {}` pour envelopper du code pouvant lever une exception
- Ne jamais intercepter `CancellationException` — toujours la relancer (rethrow)
- Éviter `try-catch` pour le flux de contrôle

```kotlin
// MAUVAIS — utiliser des exceptions pour le flux de contrôle
val user = try { repository.getUser(id) } catch (e: NotFoundException) { null }

// BON — retour nullable
val user: User? = repository.findUser(id)
```
