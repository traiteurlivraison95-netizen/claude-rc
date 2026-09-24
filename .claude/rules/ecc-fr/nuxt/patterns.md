---
paths:
  - "**/nuxt.config.*"
  - "**/app.config.*"
  - "**/app.vue"
  - "**/server/**/*.ts"
  - "**/pages/**"
  - "**/middleware/**"
---

# Patterns Nuxt

> Ce fichier étend [common/patterns.md](../common/patterns.md) avec du contenu spécifique à Nuxt.

## Sélection de la récupération de données

Déterminant. Choisir selon le timing de rendu, pas par habitude.

- `useFetch(url)` = sûr en SSR, données initiales/premier rendu orientées URL. Le choix par défaut. Transmet le résultat serveur à travers le payload, donc pas de double récupération à l'hydratation.
- `useAsyncData(key, fn)` = sûr en SSR, logique async personnalisée (SDK / GraphQL / appels combinés). La clé explicite partage le résultat entre composants.
- `$fetch` = interactions client uniquement (soumission de formulaire, clic de bouton, POST/PUT/DELETE). Pas sûr en SSR, provoque une double récupération si utilisé pour le premier rendu.
- Règle : `useFetch` / `useAsyncData` pour tout ce qui est rendu au premier affichage, `$fetch` uniquement pour les mutations déclenchées par événement.

## État partagé

- `useState('key', () => init)` pour un état partagé sûr en SSR. Les valeurs doivent être sérialisables en JSON.
- NE JAMAIS faire `export const x = ref()` à l'échelle du module. Une instance partagée unique fuit entre les requêtes SSR concurrentes et cause une fuite mémoire.
- Avec `@pinia/nuxt` : Pinia pour l'état de domaine, `useState` pour les primitives inter-composants de petite taille.
- L'initialisation asynchrone côté serveur va dans `callOnce(async () => {...})`, pas comme effet de bord à l'intérieur de `useAsyncData`.

## Routes serveur Nitro

- `server/api/*.{get,post}.ts` s'enregistrent automatiquement par chemin + méthode. Le handler est `defineEventHandler((event) => ...)`.
- Erreurs via `throw createError({ status, statusText })`. Préférer les `status` / `statusText` de la Web-API aux `statusCode` / `statusMessage` dépréciés.
- `server/middleware/` ne DOIT PAS retourner de réponse. Uniquement muter `event.context` ou définir des en-têtes.

## Middleware de route

- `app/middleware/*.ts` avec `defineNuxtRouteMiddleware((to, from) => ...)`.
- Utiliser les arguments `to` / `from`. Ne PAS appeler `useRoute()` à l'intérieur d'un middleware.
- Le suffixe `.global` s'exécute sur chaque route. Retourner `navigateTo()` pour rediriger, `abortNavigation()` pour arrêter.

## Rendu sûr pour l'hydratation

- Router selon `status` (`idle | pending | success | error`) pour les récupérations paresseuses (lazy).
- Le payload de `useAsyncData` utilise `devalue` (Date/Map/Set/refs survivent). Une réponse `server/api` est uniquement `JSON.stringify`, il faut donc définir `toJSON()` pour les types non-JSON.
- Réduire le payload avec `pick` / `transform`. Cela réduit la taille sérialisée, cela ne saute pas la récupération.

## Référence

- Skills ECC : `nuxt4-patterns`, `vite-patterns`, `frontend-patterns`.
- [Récupération de données Nuxt](https://nuxt.com/docs/getting-started/data-fetching)
- [Gestion d'état Nuxt](https://nuxt.com/docs/getting-started/state-management)
- [Moteur serveur Nuxt (Nitro)](https://nuxt.com/docs/guide/directory-structure/server)
