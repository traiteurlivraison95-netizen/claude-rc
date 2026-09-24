---
paths:
  - "**/*.vue"
---

# Tests Vue

> Ce fichier étend [common/testing.md](../common/testing.md) avec du contenu spécifique à Vue.

## Pile technique

- Vitest (runner natif Vite) plus `@vue/test-utils`. `create-vue` génère `@vitejs/plugin-vue`.
- Environnement DOM via `happy-dom` ou `jsdom`, configuré dans `vite.config.ts` sous `test.environment`.

## Rendu et async

- `mount` pour un rendu complet. `shallowMount` pour stubber tous les composants enfants.
- `trigger` et `setValue` retournent des promesses, les `await`.
- `flushPromises` vide les handlers de promesses résolues. `nextTick` stabilise le DOM après un changement d'état.

## Que tester

- Tester uniquement l'interface publique : props, événements émis, slots, sortie rendue.
- Ne pas asserter sur l'état privé ou les méthodes internes, et ne pas se reposer uniquement sur des snapshots.

## Composables

- Les composables qui n'utilisent que des API de réactivité se testent unitairement directement : appeler la fonction, asserter sur les refs retournés.
- Les composables qui utilisent des hooks de cycle de vie ou `inject` doivent être testés via un composant hôte.

## Pinia

- Dans les composants : `createTestingPinia()` depuis `@pinia/testing`, passé via `global.plugins`. Les actions sont stubbées par défaut, mettre `stubActions: false` pour les exécuter. `createSpy: vi.fn` est requis sous Vitest (pas de globals Jest).
- En isolation : `beforeEach(() => setActivePinia(createPinia()))` donne un store frais par test et évite les fuites d'état.

## Configuration de mount

- `global.plugins`, `global.stubs` (stubbe `Transition` / `TransitionGroup` par défaut), `global.mocks` (par ex. `$router`), `global.provide` (pour `inject`, clés Symbol supportées).
- `RouterLinkStub` stubbe `router-link` sans monter un routeur complet.

```ts
const wrapper = mount(AuctionCard, {
  props: { id: 1 },
  global: { plugins: [createTestingPinia({ createSpy: vi.fn })] },
})
await wrapper.find('button').trigger('click')
expect(wrapper.emitted('bid')).toBeTruthy()
```

## Référence

- Skills ECC : `frontend-patterns`, `vite-patterns`.
- Docs : <https://test-utils.vuejs.org/api/> · <https://pinia.vuejs.org/cookbook/testing.html> · <https://vitest.dev/>
