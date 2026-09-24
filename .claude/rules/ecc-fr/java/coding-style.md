---
paths:
  - "**/*.java"
---
# Style de code Java

> Ce fichier étend [common/coding-style.md](../common/coding-style.md) avec du contenu spécifique à Java.

## Formatage

- **google-java-format** ou **Checkstyle** (style Google ou Sun) pour l'application
- Un seul type public de premier niveau par fichier
- Indentation cohérente : 2 ou 4 espaces (selon le standard du projet)
- Ordre des membres : constantes, champs, constructeurs, méthodes publiques, protégées, privées

## Immutabilité

- Préférer `record` pour les types de valeur (Java 16+)
- Marquer les champs `final` par défaut — n'utiliser un état mutable que lorsque nécessaire
- Retourner des copies défensives depuis les API publiques : `List.copyOf()`, `Map.copyOf()`, `Set.copyOf()`
- Copy-on-write : retourner de nouvelles instances plutôt que de muter les existantes

```java
// BON — type de valeur immuable
public record OrderSummary(Long id, String customerName, BigDecimal total) {}

// BON — champs final, pas de setters
public class Order {
    private final Long id;
    private final List<LineItem> items;

    public List<LineItem> getItems() {
        return List.copyOf(items);
    }
}
```

## Nommage

Suivre les conventions Java standard :
- `PascalCase` pour les classes, interfaces, records, enums
- `camelCase` pour les méthodes, champs, paramètres, variables locales
- `SCREAMING_SNAKE_CASE` pour les constantes `static final`
- Paquets : tout en minuscules, domaine inversé (`com.example.app.service`)

## Fonctionnalités Java modernes

Utiliser les fonctionnalités modernes du langage lorsqu'elles améliorent la clarté :
- **Records** pour les DTO et types de valeur (Java 16+)
- **Sealed classes** pour les hiérarchies de types fermées (Java 17+)
- **Pattern matching** avec `instanceof` — sans cast explicite (Java 16+)
- **Text blocks** pour les chaînes multi-lignes — SQL, gabarits JSON (Java 15+)
- **Switch expressions** avec syntaxe fléchée (Java 14+)
- **Pattern matching dans switch** — gestion exhaustive des types sealed (Java 21+)

```java
// Pattern matching instanceof
if (shape instanceof Circle c) {
    return Math.PI * c.radius() * c.radius();
}

// Hiérarchie de type sealed
public sealed interface PaymentMethod permits CreditCard, BankTransfer, Wallet {}

// Switch expression
String label = switch (status) {
    case ACTIVE -> "Active";
    case SUSPENDED -> "Suspended";
    case CLOSED -> "Closed";
};
```

## Utilisation d'Optional

- Retourner `Optional<T>` depuis les méthodes de recherche pouvant n'avoir aucun résultat
- Utiliser `map()`, `flatMap()`, `orElseThrow()` — ne jamais appeler `get()` sans `isPresent()`
- Ne jamais utiliser `Optional` comme type de champ ou paramètre de méthode

```java
// BON
return repository.findById(id)
    .map(ResponseDto::from)
    .orElseThrow(() -> new OrderNotFoundException(id));

// MAUVAIS — Optional en paramètre
public void process(Optional<String> name) {}
```

## Gestion des erreurs

- Préférer les exceptions non contrôlées pour les erreurs de domaine
- Créer des exceptions spécifiques au domaine étendant `RuntimeException`
- Éviter les `catch (Exception e)` larges sauf dans les gestionnaires de haut niveau
- Inclure du contexte dans les messages d'exception

```java
public class OrderNotFoundException extends RuntimeException {
    public OrderNotFoundException(Long id) {
        super("Order not found: id=" + id);
    }
}
```

## Streams

- Utiliser les streams pour les transformations ; garder les pipelines courts (3-4 opérations max)
- Préférer les références de méthodes quand c'est lisible : `.map(Order::getTotal)`
- Éviter les effets de bord dans les opérations de stream
- Pour une logique complexe, préférer une boucle à un pipeline de stream alambiqué

## Références

Voir le skill : `java-coding-standards` pour des standards de codage complets avec exemples.
Voir le skill : `jpa-patterns` pour des patterns de conception d'entités JPA/Hibernate.
