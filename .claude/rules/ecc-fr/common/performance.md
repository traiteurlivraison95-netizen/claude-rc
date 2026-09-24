# Optimisation des performances

## Stratégie de sélection de modèle

**Haiku** (90 % des capacités de Sonnet, 3x moins cher) :
- Agents légers invoqués fréquemment
- Programmation en binôme et génération de code
- Agents « worker » dans les systèmes multi-agents

**Sonnet** (meilleur modèle de codage) :
- Travail de développement principal
- Orchestration de workflows multi-agents
- Tâches de codage complexes

**Opus** (raisonnement le plus profond) :
- Décisions architecturales complexes
- Exigences de raisonnement maximales
- Tâches de recherche et d'analyse

## Gestion de la fenêtre de contexte

Éviter les 20 derniers % de la fenêtre de contexte pour :
- Le refactoring à grande échelle
- L'implémentation de fonctionnalités s'étalant sur plusieurs fichiers
- Le débogage d'interactions complexes

Tâches à plus faible sensibilité au contexte :
- Modifications sur un seul fichier
- Création d'utilitaires indépendants
- Mises à jour de documentation
- Corrections de bugs simples

## Extended Thinking + Mode Plan

L'Extended Thinking est activé par défaut, réservant jusqu'à 31 999 tokens pour le raisonnement interne.

Contrôler l'Extended Thinking via :
- **Bascule** : Option+T (macOS) / Alt+T (Windows/Linux)
- **Config** : définir `alwaysThinkingEnabled` dans `~/.claude/settings.json`
- **Plafond de budget** : `export MAX_THINKING_TOKENS=10000` (bash) ou `$env:MAX_THINKING_TOKENS = "10000"` (PowerShell)
- **Mode verbeux** : Ctrl+O pour voir la sortie du raisonnement

Pour les tâches complexes nécessitant un raisonnement profond :
1. S'assurer que l'Extended Thinking est activé (activé par défaut)
2. Activer le **mode Plan** pour une approche structurée
3. Utiliser plusieurs cycles de critique pour une analyse approfondie
4. Utiliser des sous-agents à rôles répartis pour des perspectives diverses

## Dépannage de build

En cas d'échec de build :
1. Utiliser l'agent **build-error-resolver**
2. Analyser les messages d'erreur
3. Corriger progressivement
4. Vérifier après chaque correction
