---
paths:
  - "**/*.ts"
  - "**/*.tsx"
---
# Sécurité React Native / Expo

> Ce fichier étend [common/security.md](../common/security.md) avec du contenu spécifique à React Native / Expo.
> La liste de vérification obligatoire avant commit et le protocole de réponse à un incident de sécurité de common/security.md s'appliquent toujours.

## Le bundle est public

Traiter tout ce qui est livré dans l'application comme lisible par un attaquant. Un binaire mobile peut être désassemblé.

- NE JAMAIS livrer de vrais secrets (clés API privées, clés de rôle de service, secrets de signature) dans le bundle JS ou `app.config`.
- Les clés publiques/anonymes (ex. clé anon Supabase, configuration Firebase) sont acceptables UNIQUEMENT lorsqu'elles sont protégées par des règles côté serveur (RLS, règles de sécurité). Imposer l'autorisation côté backend, jamais côté client.
- Garder les opérations privilégiées derrière votre propre serveur / vos propres edge functions.

## Stockage des secrets et jetons

- Stocker les jetons d'authentification et les valeurs sensibles dans `expo-secure-store` (Keychain / Keystore) — jamais dans `AsyncStorage` ou MMKV en clair.
- Ne pas persister de secrets dans un état Redux/Zustand pouvant être sérialisé sur disque.

## Configuration

- Lire l'environnement via `expo-constants` / `app.config.ts` `extra`, et `EXPO_PUBLIC_*` uniquement pour des valeurs réellement publiques.
- Garder les secrets de build dans EAS secrets, pas dans le dépôt.

## Réseau et données

- HTTPS uniquement ; rejeter le texte en clair. Envisager le certificate pinning pour les applications à haut risque.
- Valider TOUTES les données externes (réponses d'API, params de deep link, payloads de notifications push) avec Zod avant utilisation.
- Valider et nettoyer les deep links et universal links — ne jamais router ou accorder un accès sur la base de params non validés.

## Permissions et confidentialité

- Demander le minimum de permissions sur l'appareil, au moment où elles sont nécessaires, avec une justification claire.
- Déclarer avec précision la collecte de données pour les déclarations de confidentialité App Store / Play Store.

## Dépendances

- Exécuter régulièrement `expo-doctor` et `npm audit` ; garder le SDK Expo et les dépendances natives à jour.
- Utiliser `/security-scan` (AgentShield) sur la configuration de l'agent elle-même.
