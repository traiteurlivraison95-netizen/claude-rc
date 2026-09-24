# Workflow de développement

> Ce fichier étend [common/git-workflow.md](./git-workflow.md) avec le processus complet de développement de fonctionnalités qui a lieu avant les opérations git.

Le workflow d'implémentation de fonctionnalité décrit le pipeline de développement : recherche, planification, TDD, revue de code, puis commit vers git.

## Workflow d'implémentation de fonctionnalité

0. **Recherche & réutilisation** _(obligatoire avant toute nouvelle implémentation)_
   - **Recherche de code GitHub en premier :** lancer `gh search repos` et `gh search code` pour trouver des implémentations, modèles et patterns existants avant d'écrire quoi que ce soit de nouveau.
   - **Documentation de bibliothèque en second :** utiliser Context7 ou la documentation officielle de l'éditeur pour confirmer le comportement de l'API, l'utilisation du package et les détails spécifiques à la version avant d'implémenter.
   - **Exa uniquement quand les deux premiers sont insuffisants :** utiliser Exa pour une recherche web plus large ou de la découverte après la recherche GitHub et la documentation officielle.
   - **Vérifier les registres de paquets :** rechercher dans npm, PyPI, crates.io et d'autres registres avant d'écrire du code utilitaire. Préférer des bibliothèques éprouvées à des solutions faites maison.
   - **Rechercher des implémentations adaptables :** chercher des projets open source qui résolvent 80 %+ du problème et qui peuvent être forkés, portés ou enveloppés (wrapped).
   - Préférer adopter ou porter une approche éprouvée plutôt qu'écrire du code entièrement nouveau lorsque cela répond à l'exigence.

1. **Planifier d'abord**
   - Utiliser l'agent **planner** pour créer un plan d'implémentation
   - Générer des documents de planification avant de coder : PRD, architecture, system_design, tech_doc, task_list
   - Identifier les dépendances et les risques
   - Découper en phases

2. **Approche TDD**
   - Utiliser l'agent **tdd-guide**
   - Écrire les tests d'abord (ROUGE)
   - Implémenter pour faire passer les tests (VERT)
   - Refactoriser (AMÉLIORER)
   - Vérifier une couverture de 80 %+

3. **Revue de code**
   - Utiliser l'agent **code-reviewer** immédiatement après avoir écrit le code
   - Traiter les problèmes CRITICAL et HIGH
   - Corriger les problèmes MEDIUM quand possible

4. **Commit & Push**
   - Messages de commit détaillés
   - Suivre le format conventional commits
   - Voir [git-workflow.md](./git-workflow.md) pour le format des messages de commit et le processus de PR

5. **Vérifications avant revue**
   - Vérifier que toutes les vérifications automatisées (CI/CD) passent
   - Résoudre tout conflit de fusion
   - S'assurer que la branche est à jour avec la branche cible
   - Ne demander une revue qu'une fois ces vérifications passées
