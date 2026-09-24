---
paths:
  - "**/*.java"
---
# Tests Java

> Ce fichier étend [common/testing.md](../common/testing.md) avec du contenu spécifique à Java.

## Framework de test

- **JUnit 5** (`@Test`, `@ParameterizedTest`, `@Nested`, `@DisplayName`)
- **AssertJ** pour des assertions fluides (`assertThat(result).isEqualTo(expected)`)
- **Mockito** pour simuler les dépendances
- **Testcontainers** pour les tests d'intégration nécessitant des bases de données ou services

## Organisation des tests

```
src/test/java/com/example/app/
  service/           # Tests unitaires pour la couche de service
  controller/        # Tests de la couche web / API
  repository/        # Tests d'accès aux données
  integration/       # Tests d'intégration inter-couches
```

Reproduire la structure du paquet `src/main/java` dans `src/test/java`.

## Pattern de test unitaire

```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock
    private OrderRepository orderRepository;

    private OrderService orderService;

    @BeforeEach
    void setUp() {
        orderService = new OrderService(orderRepository);
    }

    @Test
    @DisplayName("findById returns order when exists")
    void findById_existingOrder_returnsOrder() {
        var order = new Order(1L, "Alice", BigDecimal.TEN);
        when(orderRepository.findById(1L)).thenReturn(Optional.of(order));

        var result = orderService.findById(1L);

        assertThat(result.customerName()).isEqualTo("Alice");
        verify(orderRepository).findById(1L);
    }

    @Test
    @DisplayName("findById throws when order not found")
    void findById_missingOrder_throws() {
        when(orderRepository.findById(99L)).thenReturn(Optional.empty());

        assertThatThrownBy(() -> orderService.findById(99L))
            .isInstanceOf(OrderNotFoundException.class)
            .hasMessageContaining("99");
    }
}
```

## Tests paramétrés

```java
@ParameterizedTest
@CsvSource({
    "100.00, 10, 90.00",
    "50.00, 0, 50.00",
    "200.00, 25, 150.00"
})
@DisplayName("discount applied correctly")
void applyDiscount(BigDecimal price, int pct, BigDecimal expected) {
    assertThat(PricingUtils.discount(price, pct)).isEqualByComparingTo(expected);
}
```

## Tests d'intégration

Utiliser Testcontainers pour une véritable intégration avec base de données :

```java
@Testcontainers
class OrderRepositoryIT {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16");

    private OrderRepository repository;

    @BeforeEach
    void setUp() {
        var dataSource = new PGSimpleDataSource();
        dataSource.setUrl(postgres.getJdbcUrl());
        dataSource.setUser(postgres.getUsername());
        dataSource.setPassword(postgres.getPassword());
        repository = new JdbcOrderRepository(dataSource);
    }

    @Test
    void save_and_findById() {
        var saved = repository.save(new Order(null, "Bob", BigDecimal.ONE));
        var found = repository.findById(saved.getId());
        assertThat(found).isPresent();
    }
}
```

Pour les tests d'intégration Spring Boot, voir le skill : `springboot-tdd`.
Pour les tests d'intégration Quarkus, voir le skill : `quarkus-tdd`.

## Nommage des tests

Utiliser des noms descriptifs avec `@DisplayName` :
- `methodName_scenario_expectedBehavior()` pour les noms de méthode
- `@DisplayName("description lisible par un humain")` pour les rapports

## Couverture

- Viser 80 %+ de couverture de lignes
- Utiliser JaCoCo pour les rapports de couverture
- Se concentrer sur la logique de service et de domaine — ignorer les getters triviaux/classes de configuration

## Références

Voir le skill : `springboot-tdd` pour les patterns TDD Spring Boot avec MockMvc et Testcontainers.
Voir le skill : `quarkus-tdd` pour les patterns TDD Quarkus avec REST Assured et Dev Services.
Voir le skill : `java-coding-standards` pour les attentes en matière de tests.
