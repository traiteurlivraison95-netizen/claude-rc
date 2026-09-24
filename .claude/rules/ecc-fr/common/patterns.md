# Patterns courants

## Projets squelettes

Lors de l'implémentation d'une nouvelle fonctionnalité :
1. Rechercher des projets squelettes éprouvés
2. Utiliser des agents en parallèle pour évaluer les options :
   - Évaluation de sécurité
   - Analyse d'extensibilité
   - Score de pertinence
   - Planification de l'implémentation
3. Cloner la meilleure correspondance comme base
4. Itérer dans une structure éprouvée

## Design patterns

### Pattern Repository

Encapsuler l'accès aux données derrière une interface cohérente :
- Définir des opérations standard : findAll, findById, create, update, delete
- Les implémentations concrètes gèrent les détails de stockage (base de données, API, fichier, etc.)
- La logique métier dépend de l'interface abstraite, pas du mécanisme de stockage
- Permet de changer facilement de source de données et simplifie les tests avec des mocks

### Format de réponse API

Utiliser une enveloppe cohérente pour toutes les réponses API :
- Inclure un indicateur de succès/statut
- Inclure le payload de données (nullable en cas d'erreur)
- Inclure un champ de message d'erreur (nullable en cas de succès)
- Inclure des métadonnées pour les réponses paginées (total, page, limite)
