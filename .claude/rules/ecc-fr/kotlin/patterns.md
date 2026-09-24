---
paths:
  - "**/*.kt"
  - "**/*.kts"
---
# Patterns Kotlin

> Ce fichier étend [common/patterns.md](../common/patterns.md) avec du contenu spécifique à Kotlin et Android/KMP.

## Injection de dépendances

Préférer l'injection par constructeur. Utiliser Koin (KMP) ou Hilt (Android uniquement) :

```kotlin
// Koin — déclarer des modules
val dataModule = module {
    single<ItemRepository> { ItemRepositoryImpl(get(), get()) }
    factory { GetItemsUseCase(get()) }
    viewModelOf(::ItemListViewModel)
}

// Hilt — annotations
@HiltViewModel
class ItemListViewModel @Inject constructor(
    private val getItems: GetItemsUseCase
) : ViewModel()
```

## Pattern ViewModel

Objet d'état unique, réceptacle d'événements, flux de données à sens unique :

```kotlin
data class ScreenState(
    val items: List<Item> = emptyList(),
    val isLoading: Boolean = false
)

class ScreenViewModel(private val useCase: GetItemsUseCase) : ViewModel() {
    private val _state = MutableStateFlow(ScreenState())
    val state = _state.asStateFlow()

    fun onEvent(event: ScreenEvent) {
        when (event) {
            is ScreenEvent.Load -> load()
            is ScreenEvent.Delete -> delete(event.id)
        }
    }
}
```

## Pattern Repository

- Les fonctions `suspend` retournent `Result<T>` ou un type d'erreur personnalisé
- `Flow` pour les flux réactifs
- Coordonner les sources de données locales + distantes

```kotlin
interface ItemRepository {
    suspend fun getById(id: String): Result<Item>
    suspend fun getAll(): Result<List<Item>>
    fun observeAll(): Flow<List<Item>>
}
```

## Pattern UseCase

Responsabilité unique, `operator fun invoke` :

```kotlin
class GetItemUseCase(private val repository: ItemRepository) {
    suspend operator fun invoke(id: String): Result<Item> {
        return repository.getById(id)
    }
}

class GetItemsUseCase(private val repository: ItemRepository) {
    suspend operator fun invoke(): Result<List<Item>> {
        return repository.getAll()
    }
}
```

## expect/actual (KMP)

Utiliser pour les implémentations spécifiques à la plateforme :

```kotlin
// commonMain
expect fun platformName(): String
expect class SecureStorage {
    fun save(key: String, value: String)
    fun get(key: String): String?
}

// androidMain
actual fun platformName(): String = "Android"
actual class SecureStorage {
    actual fun save(key: String, value: String) { /* EncryptedSharedPreferences */ }
    actual fun get(key: String): String? = null /* ... */
}

// iosMain
actual fun platformName(): String = "iOS"
actual class SecureStorage {
    actual fun save(key: String, value: String) { /* Keychain */ }
    actual fun get(key: String): String? = null /* ... */
}
```

## Patterns de coroutines

- Utiliser `viewModelScope` dans les ViewModels, `coroutineScope` pour le travail enfant structuré
- Utiliser `stateIn(viewModelScope, SharingStarted.WhileSubscribed(5_000), initialValue)` pour un StateFlow issu de Flows froids
- Utiliser `supervisorScope` lorsque les échecs enfants doivent être indépendants

## Pattern Builder avec DSL

```kotlin
class HttpClientConfig {
    var baseUrl: String = ""
    var timeout: Long = 30_000
    private val interceptors = mutableListOf<Interceptor>()

    fun interceptor(block: () -> Interceptor) {
        interceptors.add(block())
    }
}

fun httpClient(block: HttpClientConfig.() -> Unit): HttpClient {
    val config = HttpClientConfig().apply(block)
    return HttpClient(config)
}

// Utilisation
val client = httpClient {
    baseUrl = "https://api.example.com"
    timeout = 15_000
    interceptor { AuthInterceptor(tokenProvider) }
}
```

## Références

Voir le skill : `kotlin-coroutines-flows` pour des patterns de coroutines détaillés.
Voir le skill : `android-clean-architecture` pour les patterns de module et de couche.
