---
paths:
  - "**/nuxt.config.*"
  - "**/app.config.*"
  - "**/app.vue"
  - "**/pages/**"
  - "**/layouts/**"
  - "**/middleware/**"
---

# Style de code Nuxt

> Ce fichier étend [common/coding-style.md](../common/coding-style.md) avec du contenu spécifique à Nuxt.

## Disposition des répertoires

- Le `srcDir` par défaut est `app/`. Les fichiers du framework se trouvent dans `app/pages/`, `app/layouts/`, `app/middleware/`, `app/plugins/`, `app/app.config.ts`. `nuxt.config.ts` et `server/` restent à la racine du projet.
- Certains projets redéfinissent `srcDir` vers `src/` pour une disposition Feature-Sliced Design, en remappant `dir.pages` (par exemple vers `src/app/routes`), `dir.layouts`, et les alias `@`/`~`. Toujours vérifier `nuxt.config.ts` avant de présumer un chemin.

## Discipline des auto-imports

- Les composables dans `app/composables/` et `server/utils/` sont auto-importés. Ne PAS importer manuellement les composables Nuxt (`useFetch`, `useState`, `navigateTo`) ou `defineStore` / `storeToRefs`.
- Ne PAS ajouter une dépendance `vue-router` autonome (Nuxt intègre v5) ni monter manuellement `createApp` / `createPinia` / `createRouter`. Le framework s'en charge.

## Macros du compilateur

- `definePageMeta` est une macro à la compilation. Valeurs statiques uniquement, pas de données réactives ni d'appels à effet de bord à l'intérieur.
- Augmenter le `PageMeta` typé via `declare module '#app'` plutôt que par un cast.

## Séparation des fichiers de configuration

Trois fichiers distincts, à ne pas confondre.

- `nuxt.config.ts` = build-time uniquement (`routeRules`, `modules`, `nitro`, l'indicateur `ssr`). Non réactif.
- `runtimeConfig` (dans nuxt.config) = valeurs runtime par environnement, surchargeables via `NUXT_*`. Les clés racine sont réservées au serveur, les clés `public` sont visibles côté client.
- `app/app.config.ts` = paramètres réactifs publics fixés au build (tokens de thème, feature flags). Pas de surcharge par environnement. JAMAIS de secrets.

## Head et meta

- `app.head` dans `nuxt.config.ts` n'accepte que des valeurs statiques.
- Les meta réactives passent par `useHead` / `useSeoMeta` dans le setup du composant, jamais via `app.head`.

## Référence

- Skills ECC : `nuxt4-patterns`, `vite-patterns`, `frontend-patterns`.
- [Structure des répertoires Nuxt](https://nuxt.com/docs/guide/directory-structure/app)
- [Configuration Nuxt](https://nuxt.com/docs/api/nuxt-config)
