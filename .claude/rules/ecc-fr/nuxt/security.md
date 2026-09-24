---
paths:
  - "**/nuxt.config.*"
  - "**/app.config.*"
  - "**/server/**/*.ts"
---

# Sécurité Nuxt

> Ce fichier étend [common/security.md](../common/security.md) avec du contenu spécifique à Nuxt.

## runtimeConfig public vs privé

- Les clés `runtimeConfig` racine sont réservées au serveur. `runtimeConfig.public` se sérialise dans CHAQUE payload de page (visible côté client).
- Les secrets vont uniquement à la racine. Ne jamais mettre de secrets dans `app.config.ts` ou `runtimeConfig.public`, tous deux expédiés dans le bundle client.
- Avertissement officiel : « Faites attention à ne pas exposer les clés de configuration runtime côté client en les affichant ou en les passant à `useState`. »

## Validation des entrées des routes serveur

- Utiliser les lecteurs validants de h3. Ne PAS faire confiance à `readBody` / `getQuery` / `getRouterParam` bruts.
  - `readValidatedBody(event, schema)` valide le corps.
  - `getValidatedQuery(event, schema)` valide la requête (query).
  - `getValidatedRouterParams(event, schema)` valide les paramètres de route.
- Tous acceptent une fonction de validation ou un schéma Zod et lèvent une exception en cas d'échec.

## Fuite du payload SSR

- Tout ce qui se trouve dans `useState`, les résultats de `useFetch` / `useAsyncData`, ou `runtimeConfig.public` est sérialisé dans le payload client. N'y écrivez jamais de secret.
- Utiliser `useServerSeoMeta` pour des meta réservées au serveur sans coût côté client.

## Passage des cookies et de l'authentification en SSR

- Nuxt n'attache PAS automatiquement les cookies de l'utilisateur entrant aux appels `$fetch` sortants côté serveur.
- Les transmettre explicitement avec `useRequestFetch()` (le plus propre, pré-lié aux en-têtes de la requête) ou `useRequestHeaders(['cookie'])`.
- Relayer un `Set-Cookie` du backend vers le navigateur via `$fetch.raw` + `appendResponseHeader(event, 'set-cookie', ...)`.
- socket.io est client uniquement (plugin `.client.ts`), jamais en SSR.

## SSRF sur $fetch côté serveur

- Les routes serveur s'exécutent avec un accès réseau sortant complet. Ne jamais passer une entrée contrôlée par l'utilisateur directement dans une URL ou un hôte `$fetch` côté serveur.
- Valider d'abord le paramètre (utilitaires h3 ci-dessus), mettre en liste blanche la cible, épingler sur `runtimeConfig.public.apiBase`, rejeter les URL absolues fournies par l'utilisateur.
- Déclencher automatiquement `/security-review` uniquement pour les routes qui effectuent des requêtes réseau externes (`$fetch` côté serveur), gèrent des jetons d'authentification ou identifiants, ou effectuent des mutations sensibles ou des vérifications d'autorisation. Exemples : endpoints proxy propices au SSRF, échange de jetons ou réinitialisation de mot de passe, actions d'administration. Ignorer les routes de lecture seule bénignes qui n'acceptent que des paramètres de requête validés.

## Référence

- Skills ECC : `security-review`, `nuxt4-patterns`.
- [Configuration runtime Nuxt](https://nuxt.com/docs/guide/going-further/runtime-config)
- [Utilitaires de requête h3](https://v1.h3.dev/utils/request)
