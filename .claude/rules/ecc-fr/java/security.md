---
paths:
  - "**/*.java"
---
# Sécurité Java

> Ce fichier étend [common/security.md](../common/security.md) avec du contenu spécifique à Java.

## Gestion des secrets

- Ne jamais coder en dur des clés API, jetons ou identifiants dans le code source
- Utiliser des variables d'environnement : `System.getenv("API_KEY")`
- Utiliser un gestionnaire de secrets (Vault, AWS Secrets Manager) pour les secrets de production
- Garder les fichiers de configuration locaux contenant des secrets dans `.gitignore`

```java
// MAUVAIS
private static final String API_KEY = "sk-abc123...";

// BON — variable d'environnement
String apiKey = System.getenv("PAYMENT_API_KEY");
Objects.requireNonNull(apiKey, "PAYMENT_API_KEY must be set");
```

## Prévention des injections SQL

- Toujours utiliser des requêtes paramétrées — ne jamais concaténer d'entrées utilisateur dans du SQL
- Utiliser `PreparedStatement` ou l'API de requête paramétrée de votre framework
- Valider et assainir toute entrée utilisée dans des requêtes natives

```java
// MAUVAIS — injection SQL par concaténation de chaînes
Statement stmt = conn.createStatement();
String sql = "SELECT * FROM orders WHERE name = '" + name + "'";
stmt.executeQuery(sql);

// BON — PreparedStatement avec requête paramétrée
PreparedStatement ps = conn.prepareStatement("SELECT * FROM orders WHERE name = ?");
ps.setString(1, name);

// BON — template JDBC
jdbcTemplate.query("SELECT * FROM orders WHERE name = ?", mapper, name);
```

## Validation des entrées

- Valider toutes les entrées utilisateur aux frontières du système avant traitement
- Utiliser Bean Validation (`@NotNull`, `@NotBlank`, `@Size`) sur les DTO lors de l'utilisation d'un framework de validation
- Assainir les chemins de fichiers et les chaînes fournies par l'utilisateur avant utilisation
- Rejeter les entrées qui échouent à la validation avec des messages d'erreur clairs

```java
// Validation manuelle en Java pur
public Order createOrder(String customerName, BigDecimal amount) {
    if (customerName == null || customerName.isBlank()) {
        throw new IllegalArgumentException("Customer name is required");
    }
    if (amount == null || amount.compareTo(BigDecimal.ZERO) <= 0) {
        throw new IllegalArgumentException("Amount must be positive");
    }
    return new Order(customerName, amount);
}
```

## Authentification et autorisation

- Ne jamais implémenter de cryptographie d'authentification personnalisée — utiliser des bibliothèques établies
- Stocker les mots de passe avec bcrypt ou Argon2, jamais MD5/SHA1
- Appliquer les vérifications d'autorisation aux frontières des services
- Effacer les données sensibles des logs — ne jamais logger de mots de passe, jetons ou PII

## Sécurité des dépendances

- Exécuter `mvn dependency:tree` ou `./gradlew dependencies` pour auditer les dépendances transitives
- Utiliser OWASP Dependency-Check ou Snyk pour scanner les CVE connues
- Garder les dépendances à jour — configurer Dependabot ou Renovate

## Messages d'erreur

- Ne jamais exposer les traces de pile, les chemins internes ou les erreurs SQL dans les réponses API
- Mapper les exceptions vers des messages génériques et sûrs aux frontières des handlers
- Logger les erreurs détaillées côté serveur ; retourner des messages génériques aux clients

```java
// Logger le détail, retourner un message générique
try {
    return orderService.findById(id);
} catch (OrderNotFoundException ex) {
    log.warn("Order not found: id={}", id);
    return ApiResponse.error("Resource not found");  // générique, sans détails internes
} catch (Exception ex) {
    log.error("Unexpected error processing order id={}", id, ex);
    return ApiResponse.error("Internal server error");  // ne jamais exposer ex.getMessage()
}
```

## Références

Voir le skill : `springboot-security` pour les patterns d'authentification et d'autorisation Spring Security.
Voir le skill : `quarkus-security` pour la sécurité Quarkus avec JWT/OIDC, RBAC et CDI.
Voir le skill : `security-review` pour des checklists de sécurité générales.
