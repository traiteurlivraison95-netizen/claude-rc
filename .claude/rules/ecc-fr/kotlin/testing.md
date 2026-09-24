---
paths:
  - "**/*.kt"
  - "**/*.kts"
---
# Tests Kotlin

> Ce fichier étend [common/testing.md](../common/testing.md) avec du contenu spécifique à Kotlin et Android/KMP.

## Framework de test

- **kotlin.test** pour le multiplateforme (KMP) — `@Test`, `assertEquals`, `assertTrue`
- **JUnit 4/5** pour les tests spécifiques à Android
- **Turbine** pour tester les Flows et StateFlow
- **kotlinx-coroutines-test** pour tester les coroutines (`runTest`, `TestDispatcher`)

## Tests de ViewModel avec Turbine

```kotlin
@Test
fun `loading state emitted then data`() = runTest {
    val repo = FakeItemRepository()
    repo.addItem(testItem)
    val viewModel = ItemListViewModel(GetItemsUseCase(repo))

    viewModel.state.test {
        assertEquals(ItemListState(), awaitItem())     // état initial
        viewModel.onEvent(ItemListEvent.Load)
        assertTrue(awaitItem().isLoading)               // chargement
        assertEquals(listOf(testItem), awaitItem().items) // chargé
    }
}
```

## Fakes plutôt que Mocks

Préférer des fakes écrits à la main aux frameworks de mocking :

```kotlin
class FakeItemRepository : ItemRepository {
    private val items = mutableListOf<Item>()
    var fetchError: Throwable? = null

    override suspend fun getAll(): Result<List<Item>> {
        fetchError?.let { return Result.failure(it) }
        return Result.success(items.toList())
    }

    override fun observeAll(): Flow<List<Item>> = flowOf(items.toList())

    fun addItem(item: Item) { items.add(item) }
}
```

## Tests de coroutines

```kotlin
@Test
fun `parallel operations complete`() = runTest {
    val repo = FakeRepository()
    val result = loadDashboard(repo)
    advanceUntilIdle()
    assertNotNull(result.items)
    assertNotNull(result.stats)
}
```

Utiliser `runTest` — il fait automatiquement avancer le temps virtuel et fournit un `TestScope`.

## Ktor MockEngine

```kotlin
val mockEngine = MockEngine { request ->
    when (request.url.encodedPath) {
        "/api/items" -> respond(
            content = Json.encodeToString(testItems),
            headers = headersOf(HttpHeaders.ContentType, ContentType.Application.Json.toString())
        )
        else -> respondError(HttpStatusCode.NotFound)
    }
}

val client = HttpClient(mockEngine) {
    install(ContentNegotiation) { json() }
}
```

## Tests Room/SQLDelight

- Room : utiliser `Room.inMemoryDatabaseBuilder()` pour les tests en mémoire
- SQLDelight : utiliser `JdbcSqliteDriver(JdbcSqliteDriver.IN_MEMORY)` pour les tests JVM

```kotlin
@Test
fun `insert and query items`() = runTest {
    val driver = JdbcSqliteDriver(JdbcSqliteDriver.IN_MEMORY)
    Database.Schema.create(driver)
    val db = Database(driver)

    db.itemQueries.insert("1", "Sample Item", "description")
    val items = db.itemQueries.getAll().executeAsList()
    assertEquals(1, items.size)
}
```

## Nommage des tests

Utiliser des noms descriptifs entre backticks :

```kotlin
@Test
fun `search with empty query returns all items`() = runTest { }

@Test
fun `delete item emits updated list without deleted item`() = runTest { }
```

## Organisation des tests

```
src/
├── commonTest/kotlin/     # Tests partagés (ViewModel, UseCase, Repository)
├── androidUnitTest/kotlin/ # Tests unitaires Android (JUnit)
├── androidInstrumentedTest/kotlin/  # Tests instrumentés (Room, UI)
└── iosTest/kotlin/        # Tests spécifiques à iOS
```

Couverture de test minimale : ViewModel + UseCase pour chaque fonctionnalité.
