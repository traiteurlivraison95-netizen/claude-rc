---
paths:
  - "**/*.php"
  - "**/composer.lock"
  - "**/composer.json"
---
# Sécurité PHP

> Ce fichier étend [common/security.md](../common/security.md) avec du contenu spécifique à PHP.

## Entrée et sortie

- Valider les entrées de requête à la frontière du framework (`FormRequest`, Symfony Validator, ou validation explicite par DTO).
- Échapper la sortie dans les templates par défaut ; traiter le rendu HTML brut comme une exception qui doit être justifiée.
- Ne jamais faire confiance aux paramètres de requête, cookies, en-têtes ou métadonnées de fichiers téléversés sans validation.

## Sécurité de la base de données

- Utiliser des requêtes préparées (`PDO`, Doctrine, query builder Eloquent) pour toutes les requêtes dynamiques.
- Éviter la construction de SQL par concaténation de chaînes dans les contrôleurs/vues.
- Délimiter soigneusement l'assignation de masse (mass-assignment) de l'ORM et mettre en liste blanche les champs modifiables.

## Secrets et dépendances

- Charger les secrets depuis des variables d'environnement ou un gestionnaire de secrets, jamais depuis des fichiers de configuration versionnés.
- Exécuter `composer audit` en CI et vérifier la confiance envers le mainteneur du nouveau package avant d'ajouter une dépendance.
- Épingler les versions majeures délibérément et retirer rapidement les packages abandonnés.

## Sécurité de l'authentification et des sessions

- Utiliser `password_hash()` / `password_verify()` pour le stockage des mots de passe.
- Régénérer les identifiants de session après l'authentification et les changements de privilèges.
- Appliquer la protection CSRF sur les requêtes web qui modifient l'état.

## Référence

Voir la compétence : `laravel-security` pour des conseils de sécurité spécifiques à Laravel.
