---
paths:
  - "**/*.ts"
  - "**/*.tsx"
---
# Tests React Native / Expo

> Ce fichier étend [common/testing.md](../common/testing.md) avec du contenu spécifique à React Native / Expo.
> L'objectif de couverture et le workflow TDD sont hérités de common/testing.md (minimum 80 %, RED-GREEN-REFACTOR).

## Outillage

| Couche | Outil |
|-------|------|
| Unitaire / composant | Jest + `@testing-library/react-native` (via le preset `jest-expo`) |
| Hooks | `@testing-library/react-native` `renderHook` |
| E2E | Maestro (recommandé, flux YAML simples) ou Detox |
| Sécurité des types | `tsc --noEmit` en CI |

## Tests de composants

- Interroger par rôle/label/texte accessible, pas par `testID` sauf nécessité — cela impose aussi l'accessibilité.
- Faire des assertions sur le comportement visible par l'utilisateur, pas sur les détails d'implémentation.
- Suivre le schéma Arrange-Act-Assert.

```tsx
import { render, screen, fireEvent } from '@testing-library/react-native'

test('calls onSelect with the user id when pressed', () => {
  const onSelect = jest.fn()
  render(<UserCard user={{ id: '1', email: 'a@b.com' }} onSelect={onSelect} />)

  fireEvent.press(screen.getByText('a@b.com'))

  expect(onSelect).toHaveBeenCalledWith('1')
})
```

## Mocking

- Mocker les modules du SDK Expo (caméra, localisation, notifications, secure-store) à la frontière du test.
- Envelopper les composants qui utilisent TanStack Query dans un `QueryClientProvider` avec un client neuf par test.
- Mocker la navigation (`expo-router`) afin que les écrans se rendent en isolation.

## E2E

- Couvrir uniquement les flux critiques : authentification, navigation principale, transactions essentielles.
- Exécuter l'E2E en CI contre une application buildée (EAS Build) avant la mise en production.

## Que tester en premier

Utiliser l'agent `tdd-guide` de manière proactive pour les nouvelles fonctionnalités : écrire un test qui échoue et qui capture le comportement, puis implémenter.
