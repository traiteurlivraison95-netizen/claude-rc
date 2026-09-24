# Exigences de test

## Couverture de test minimale : 80 %

Types de tests (TOUS requis) :
1. **Tests unitaires** - Fonctions, utilitaires, composants individuels
2. **Tests d'intégration** - Endpoints API, opérations de base de données
3. **Tests E2E** - Parcours utilisateur critiques (framework choisi selon le langage)

## Développement piloté par les tests (TDD)

Workflow OBLIGATOIRE :
1. Écrire le test d'abord (ROUGE)
2. Lancer le test - il doit ÉCHOUER
3. Écrire l'implémentation minimale (VERT)
4. Lancer le test - il doit PASSER
5. Refactoriser (AMÉLIORER)
6. Vérifier la couverture (80 %+)

## Dépannage des échecs de test

1. Utiliser l'agent **tdd-guide**
2. Vérifier l'isolation des tests
3. Vérifier que les mocks sont corrects
4. Corriger l'implémentation, pas les tests (sauf si les tests sont erronés)

## Support par agent

- **tdd-guide** - À utiliser PROACTIVEMENT pour les nouvelles fonctionnalités, impose l'écriture des tests en premier

## Structure de test (pattern AAA)

Préférer la structure Arrange-Act-Assert pour les tests :

```typescript
test('calculates similarity correctly', () => {
  // Arrange
  const vector1 = [1, 0, 0]
  const vector2 = [0, 1, 0]

  // Act
  const similarity = calculateCosineSimilarity(vector1, vector2)

  // Assert
  expect(similarity).toBe(0)
})
```

### Nommage des tests

Utiliser des noms descriptifs qui expliquent le comportement testé :

```typescript
test('returns empty array when no markets match query', () => {})
test('throws error when API key is missing', () => {})
test('falls back to substring search when Redis is unavailable', () => {})
```
