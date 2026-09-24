---
paths:
  - "**/*.test.tsx"
  - "**/*.test.jsx"
  - "**/*.spec.tsx"
  - "**/*.spec.jsx"
  - "**/__tests__/**/*.ts"
  - "**/__tests__/**/*.tsx"
---
# Tests React

> Ce fichier étend [typescript/testing.md](../typescript/testing.md) et [common/testing.md](../common/testing.md) avec du contenu spécifique à React.

## Choix de la bibliothèque

- **React Testing Library (RTL)** — la référence pour les tests de composants. Teste le comportement via le DOM rendu.
- **Vitest** — lanceur de test préféré pour les nouveaux projets basés sur Vite. Plus rapide que Jest, ESM natif, même API.
- **Jest** — toujours la valeur par défaut pour les projets Next.js / CRA. RTL fonctionne de manière identique.
- **Playwright Component Testing** — lorsque les tests de composants nécessitent un vrai moteur de navigateur (animation, mise en page, événements complexes)
- **Cypress Component Testing** — lanceur de tests de composants alternatif en navigateur réel

Choisir un seul lanceur de tests de composants par projet — ne pas mélanger RTL + Playwright CT dans le même dépôt.

## Principe fondamental

Tester ce que l'utilisateur voit et fait, pas les détails d'implémentation.

- Interroger d'abord par rôle accessible, puis par label, puis par texte — se replier sur `data-testid` uniquement lorsque rien d'autre ne convient
- Ne jamais faire d'assertion sur l'état interne, les props passées aux enfants, ou les hooks appelés
- Refactoriser sans casser les tests = le test testait le comportement ; c'est l'objectif

## Priorité des requêtes

RTL expose des requêtes en trois familles. Utiliser cet ordre de priorité de haut en bas :

1. **Accessible à tous**
   - `getByRole(role, { name })` — choix principal
   - `getByLabelText` — pour les champs de formulaire
   - `getByPlaceholderText` — lorsqu'aucun label n'est disponible (et ajouter un label)
   - `getByText` — pour du texte non interactif
   - `getByDisplayValue` — pour les champs de formulaire avec une valeur actuelle

2. **Requêtes sémantiques**
   - `getByAltText` — pour les images
   - `getByTitle` — dernier recours, faible valeur d'accessibilité

3. **Identifiants de test**
   - `getByTestId("some-id")` — échappatoire uniquement, quand rien de ce qui précède ne fonctionne

`getBy*` lève une exception en l'absence de correspondance. `queryBy*` retourne null (à utiliser pour affirmer une absence). `findBy*` retourne une promesse (à utiliser pour l'asynchrone).

## Interaction utilisateur

Préférer `userEvent` à `fireEvent`. `userEvent` simule de vraies séquences navigateur (focus, keydown, beforeinput, input, keyup) — `fireEvent` déclenche un seul événement synthétique.

```tsx
import userEvent from "@testing-library/user-event";

test("submits the form", async () => {
  const user = userEvent.setup();
  render(<UserForm onSubmit={handleSubmit} />);

  await user.type(screen.getByLabelText("Email"), "user@example.com");
  await user.click(screen.getByRole("button", { name: /save/i }));

  expect(handleSubmit).toHaveBeenCalledWith({ email: "user@example.com" });
});
```

- Toujours `await` les appels à `userEvent` — ils sont asynchrones
- Appeler `userEvent.setup()` une fois en haut de chaque test, puis réutiliser le `user` retourné

## Assertions asynchrones

```tsx
// INCORRECT : requête synchrone pour un contenu rendu de façon asynchrone
expect(screen.getByText("Loaded")).toBeInTheDocument();   // lève une exception — pas encore dans le DOM

// CORRECT : findBy* (retourne une promesse, réessaie)
expect(await screen.findByText("Loaded")).toBeInTheDocument();

// CORRECT : waitFor pour les assertions non liées à un élément
await waitFor(() => expect(saveSpy).toHaveBeenCalled());
```

- `findBy*` pour l'apparition asynchrone d'un élément
- `waitFor` pour les attentes asynchrones sur des effets de bord ou d'autres matchers
- Jamais `setTimeout` + assertion — instable (flaky)

## Simulation réseau avec MSW

Utiliser Mock Service Worker pour tout test qui atteint une frontière réseau. MSW s'exécute au niveau de la couche réseau, donc le composant, les hooks et la bibliothèque de fetch se comportent tous comme en production.

```tsx
// configuration du test
import { setupServer } from "msw/node";
import { http, HttpResponse } from "msw";

const server = setupServer(
  http.get("/api/users/:id", ({ params }) =>
    HttpResponse.json({ id: params.id, name: "Alice" }),
  ),
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

Surcharge par test :

```tsx
test("renders error on 500", async () => {
  server.use(http.get("/api/users/:id", () => new HttpResponse(null, { status: 500 })));
  render(<UserPage id="1" />);
  expect(await screen.findByText(/something went wrong/i)).toBeInTheDocument();
});
```

## Éviter les tests de snapshot pour les composants

Les snapshots de la sortie rendue sont fragiles, difficiles à relire, et approuvés sans examen réel par les relecteurs. Ne les utiliser que pour :

- La sérialisation pure de données (ex. un transformateur qui produit une chaîne stable)
- Détecter des régressions involontaires dans une sortie non visuelle

Pour la régression visuelle de composants, utiliser des captures d'écran Playwright / Cypress / Percy — de véritables diffs visuels, pas des diffs de DOM.

## Utilitaires de configuration de test

Envelopper les providers une seule fois :

```tsx
function renderWithProviders(ui: React.ReactElement) {
  return render(
    <QueryClientProvider client={new QueryClient()}>
      <ThemeProvider theme={lightTheme}>
        <Router>{ui}</Router>
      </ThemeProvider>
    </QueryClientProvider>,
  );
}
```

Exporter depuis `test-utils.tsx` et utiliser partout.

## Tests de hooks personnalisés

Utiliser `renderHook` de RTL :

```tsx
import { renderHook, act } from "@testing-library/react";

test("useCounter increments", () => {
  const { result } = renderHook(() => useCounter());
  act(() => result.current.increment());
  expect(result.current.count).toBe(1);
});
```

- Toujours envelopper les appels qui modifient l'état dans `act`
- Toujours tester via l'API publique du hook, pas l'implémentation interne

## Assertions d'accessibilité

```tsx
import { axe } from "vitest-axe";   // ou jest-axe

test("UserCard has no a11y violations", async () => {
  const { container } = render(<UserCard user={mockUser} />);
  expect(await axe(container)).toHaveNoViolations();
});
```

Exécuter des assertions axe dans les tests de composants — détecte les labels manquants, la mauvaise utilisation d'ARIA, le contraste de couleur (de façon limitée).

## Quand se tourner vers Playwright / Cypress

Un test de composant avec RTL + JSDOM ne peut pas :

- Tester la mise en page réelle (flexbox, grid, rendu dépendant du viewport)
- Tester le défilement, le glisser-déposer, le collage depuis le presse-papiers
- Tester les animations natives du navigateur, les transitions CSS
- Tester les interactions inter-frames (iframes, popups)

Pour cela, utiliser Playwright Component Testing ou des exécutions Playwright/Cypress de bout en bout. Voir la compétence [e2e-testing](../../../skills/e2e-testing/SKILL.md).

## Objectifs de couverture

| Couche | Objectif |
|---|---|
| Fonctions utilitaires pures | ≥90% |
| Hooks personnalisés | ≥85% |
| Composants (présentation) | ≥80% — comportement, pas lignes |
| Composants Container | ≥70% — chemins nominaux + états d'erreur |
| Pages (couvertes séparément par l'E2E) | Test de fumée minimum par route |

## Anti-patterns

- Faire des assertions sur `container.querySelector` — contourne les requêtes d'accessibilité
- Faire des assertions sur le nombre de rendus — détail d'implémentation
- Mocker les hooks React (`jest.mock("react", ...)`) — refactoriser le composant à la place
- Mocker les composants enfants par défaut — teste l'intégration, pas le parent isolé
- Ignorer les avertissements manuels `act()` — ils indiquent de vrais bugs

## Référence des compétences

Voir `skills/react-testing/SKILL.md` pour des exemples de tests de bout en bout, des patterns MSW, et une base pour les tests d'accessibilité.
