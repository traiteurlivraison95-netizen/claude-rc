---
paths:
  - "**/*.java"
---
# Patterns Java

> Ce fichier étend [common/patterns.md](../common/patterns.md) avec du contenu spécifique à Java.

## Pattern Repository

Encapsuler l'accès aux données derrière une interface :

```java
public interface OrderRepository {
    Optional<Order> findById(Long id);
    List<Order> findAll();
    Order save(Order order);
    void deleteById(Long id);
}
```

Les implémentations concrètes gèrent les détails de stockage (JPA, JDBC, en mémoire pour les tests).

## Couche de service

Logique métier dans des classes de service ; garder les contrôleurs et les repositories légers :

```java
public class OrderService {
    private final OrderRepository orderRepository;
    private final PaymentGateway paymentGateway;

    public OrderService(OrderRepository orderRepository, PaymentGateway paymentGateway) {
        this.orderRepository = orderRepository;
        this.paymentGateway = paymentGateway;
    }

    public OrderSummary placeOrder(CreateOrderRequest request) {
        var order = Order.from(request);
        paymentGateway.charge(order.total());
        var saved = orderRepository.save(order);
        return OrderSummary.from(saved);
    }
}
```

## Injection par constructeur

Toujours utiliser l'injection par constructeur — jamais l'injection par champ :

```java
// BON — injection par constructeur (testable, immuable)
public class NotificationService {
    private final EmailSender emailSender;

    public NotificationService(EmailSender emailSender) {
        this.emailSender = emailSender;
    }
}

// MAUVAIS — injection par champ (non testable sans réflexion, nécessite la magie du framework)
public class NotificationService {
    @Inject // ou @Autowired
    private EmailSender emailSender;
}
```

## Mapping DTO

Utiliser des records pour les DTO. Mapper aux frontières service/contrôleur :

```java
public record OrderResponse(Long id, String customer, BigDecimal total) {
    public static OrderResponse from(Order order) {
        return new OrderResponse(order.getId(), order.getCustomerName(), order.getTotal());
    }
}
```

## Pattern Builder

Utiliser pour les objets ayant de nombreux paramètres optionnels :

```java
public class SearchCriteria {
    private final String query;
    private final int page;
    private final int size;
    private final String sortBy;

    private SearchCriteria(Builder builder) {
        this.query = builder.query;
        this.page = builder.page;
        this.size = builder.size;
        this.sortBy = builder.sortBy;
    }

    public static class Builder {
        private String query = "";
        private int page = 0;
        private int size = 20;
        private String sortBy = "id";

        public Builder query(String query) { this.query = query; return this; }
        public Builder page(int page) { this.page = page; return this; }
        public Builder size(int size) { this.size = size; return this; }
        public Builder sortBy(String sortBy) { this.sortBy = sortBy; return this; }
        public SearchCriteria build() { return new SearchCriteria(this); }
    }
}
```

## Types Sealed pour les modèles de domaine

```java
public sealed interface PaymentResult permits PaymentSuccess, PaymentFailure {
    record PaymentSuccess(String transactionId, BigDecimal amount) implements PaymentResult {}
    record PaymentFailure(String errorCode, String message) implements PaymentResult {}
}

// Gestion exhaustive (Java 21+)
String message = switch (result) {
    case PaymentSuccess s -> "Paid: " + s.transactionId();
    case PaymentFailure f -> "Failed: " + f.errorCode();
};
```

## Enveloppe de réponse API

Réponses API cohérentes :

```java
public record ApiResponse<T>(boolean success, T data, String error) {
    public static <T> ApiResponse<T> ok(T data) {
        return new ApiResponse<>(true, data, null);
    }
    public static <T> ApiResponse<T> error(String message) {
        return new ApiResponse<>(false, null, message);
    }
}
```

## Références

Voir le skill : `springboot-patterns` pour les patterns d'architecture Spring Boot.
Voir le skill : `quarkus-patterns` pour les patterns d'architecture Quarkus avec REST, Panache et messagerie.
Voir le skill : `jpa-patterns` pour la conception d'entités et l'optimisation des requêtes.
