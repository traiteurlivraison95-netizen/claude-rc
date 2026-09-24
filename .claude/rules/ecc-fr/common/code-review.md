# Standards de revue de code

## Objectif

La revue de code garantit la qualité, la sécurité et la maintenabilité avant que le code ne soit fusionné. Cette règle définit quand et comment mener des revues de code.

## Quand faire une revue

**Déclencheurs de revue OBLIGATOIRES :**

- Après avoir écrit ou modifié du code
- Avant tout commit sur des branches partagées
- Quand du code sensible pour la sécurité est modifié (auth, paiements, données utilisateur)
- Quand des changements architecturaux sont effectués
- Avant de fusionner des pull requests

**Exigences avant revue :**

Avant de demander une revue, s'assurer que :

- Toutes les vérifications automatisées (CI/CD) passent
- Les conflits de fusion sont résolus
- La branche est à jour avec la branche cible

## Checklist de revue

Avant de considérer le code terminé :

- [ ] Le code est lisible et bien nommé
- [ ] Les fonctions sont ciblées (<50 lignes)
- [ ] Les fichiers source sont cohésifs (sous le plafond souple de maintenabilité de 800 lignes, ou avec une raison pour une exception délibérée)
- [ ] Pas d'imbrication profonde (>4 niveaux)
- [ ] Les erreurs sont gérées explicitement
- [ ] Pas de secrets ou identifiants en dur
- [ ] Pas de console.log ou d'instructions de débogage
- [ ] Des tests existent pour les nouvelles fonctionnalités
- [ ] La couverture de test atteint le minimum de 80 %

## Déclencheurs de revue de sécurité

**ARRÊTER et utiliser l'agent security-reviewer quand :**

- Code d'authentification ou d'autorisation
- Traitement des entrées utilisateur
- Requêtes de base de données
- Opérations sur le système de fichiers
- Appels API externes
- Opérations cryptographiques
- Code de paiement ou financier

## Niveaux de sévérité de revue

| Niveau | Signification | Action |
|-------|---------|--------|
| CRITICAL | Vulnérabilité de sécurité ou risque de perte de données | **BLOQUER** - Doit être corrigé avant fusion |
| HIGH | Bug ou problème de qualité significatif | **AVERTIR** - Devrait être corrigé avant fusion |
| MEDIUM | Préoccupation de maintenabilité, y compris un fichier source non justifié dépassant le plafond souple de 800 lignes | **INFO** - À envisager de corriger |
| LOW | Style ou suggestion mineure | **NOTE** - Optionnel |

## Utilisation des agents

Utiliser ces agents pour la revue de code :

| Agent | Objectif |
|-------|---------|
| **code-reviewer** | Qualité de code générale, patterns, bonnes pratiques |
| **security-reviewer** | Vulnérabilités de sécurité, OWASP Top 10 |
| **typescript-reviewer** | Problèmes spécifiques à TypeScript/JavaScript |
| **python-reviewer** | Problèmes spécifiques à Python |
| **go-reviewer** | Problèmes spécifiques à Go |
| **rust-reviewer** | Problèmes spécifiques à Rust |

## Workflow de revue

```
1. Lancer git diff pour comprendre les changements
2. Vérifier d'abord la checklist de sécurité
3. Vérifier la checklist de qualité de code
4. Lancer les tests pertinents
5. Vérifier que la couverture >= 80 %
6. Utiliser l'agent approprié pour une revue détaillée
```

## Problèmes courants à détecter

### Sécurité

- Identifiants en dur (clés API, mots de passe, tokens)
- Injection SQL (concaténation de chaînes dans les requêtes)
- Vulnérabilités XSS (entrée utilisateur non échappée)
- Traversée de chemin (chemins de fichiers non assainis)
- Protection CSRF manquante
- Contournements d'authentification

### Qualité de code

- Fonctions volumineuses (>50 lignes) - à diviser
- Fichiers volumineux (>800 lignes) - extraire des modules
- Imbrication profonde (>4 niveaux) - utiliser des retours anticipés
- Gestion d'erreurs manquante - gérer explicitement
- Patterns de mutation - préférer les opérations immuables
- Tests manquants - ajouter de la couverture de test

### Performance

- Requêtes N+1 - utiliser des JOIN ou du batching
- Pagination manquante - ajouter une LIMIT aux requêtes
- Requêtes non bornées - ajouter des contraintes
- Cache manquant - mettre en cache les opérations coûteuses

## Critères d'approbation

- **Approuver** : aucun problème CRITICAL ou HIGH
- **Avertissement** : seulement des problèmes HIGH (fusionner avec prudence)
- **Bloquer** : problèmes CRITICAL trouvés

## Intégration avec les autres règles

Cette règle fonctionne avec :

- [testing.md](testing.md) - Exigences de couverture de test
- [security.md](security.md) - Checklist de sécurité
- [git-workflow.md](git-workflow.md) - Standards de commit
- [agents.md](agents.md) - Délégation d'agents
