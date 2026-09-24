---
paths:
  - "**/nuxt.config.*"
  - "**/server/**/*.ts"
  - "**/pages/**"
  - "**/layouts/**"
  - "**/middleware/**"
---

# Tests Nuxt

> Ce fichier étend [common/testing.md](../common/testing.md) avec du contenu spécifique à Nuxt.

Paquet : `@nuxt/test-utils`. Vitest en priorité pour les tests unitaires et de composants, avec un support intégré de Playwright pour les tests E2E dans le navigateur. nuxt-vitest et vitest-environment-nuxt sont supplantés et intégrés dedans.

## Mise en place

- Installer les dépendances de développement : `@nuxt/test-utils vitest @vue/test-utils happy-dom playwright-core`.
- Config : `defineVitestConfig({ test: { environment: 'nuxt' } })` depuis `@nuxt/test-utils/config`. Utiliser `defineVitestProject` pour le multi-projet (environnements séparés unit / nuxt / e2e).
- Ajouter `@nuxt/test-utils/module` à `nuxt.config`. Opt-in par fichier via `// @vitest-environment nuxt`.

## Helpers runtime

À importer depuis `@nuxt/test-utils/runtime`.

- `mountSuspended(component, opts)` monte dans l'environnement Nuxt avec un setup async + injection de plugins (accepte les options de mount de `@vue/test-utils` + `route`).
- `renderSuspended(component, opts)` est la variante Testing Library (nécessite `@testing-library/vue`).
- `mockNuxtImport(name, factory)` simule les auto-imports (par ex. `useState`). Une fois par import et par fichier, utiliser `vi.hoisted()`.
- `mockComponent(name, factory)` simule par nom PascalCase ou par chemin.
- `registerEndpoint(path, handler|opts)` simule un endpoint Nitro pour tester les routes serveur ou simuler le backend. Prend en charge la méthode + `once`.

## Helpers E2E

À importer depuis `@nuxt/test-utils/e2e`.

- `await setup({ rootDir, server, browser, ... })` à l'intérieur du bloc describe (gère beforeAll/afterAll).
- Ensuite `$fetch(url)` (HTML rendu), `fetch(url)` (objet réponse), `url(path)` (URL complète avec le port), `createPage(url)` (Playwright).
- Intégration Playwright : importer `expect` / `test` depuis `@nuxt/test-utils/playwright`.

## Quoi tester et comment

- Composables : simuler les auto-imports avec `mockNuxtImport`, monter un composant hôte via `mountSuspended` pour exercer `useState` / `useFetch` dans le runtime Nuxt.
- Routes serveur : `registerEndpoint` pour simuler, ou `$fetch` / `fetch` e2e contre le vrai serveur Nitro.

## Référence

- Skills ECC : `nuxt4-patterns`, `e2e-testing`, `vite-patterns`.
- [Documentation des tests Nuxt](https://nuxt.com/docs/getting-started/testing)
- [@nuxt/test-utils sur npm](https://www.npmjs.com/package/@nuxt/test-utils)
