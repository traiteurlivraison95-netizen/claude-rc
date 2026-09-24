---
paths:
  - "**/*.vue"
---

# Patterns Vue

> Ce fichier étend [common/patterns.md](../common/patterns.md) avec du contenu spécifique à Vue.

## Composables

- Le composable (`useXxx`) est l'unité de logique réutilisable. En Feature-Sliced Design, il vit dans le segment `model` de la tranche.
- Accepter des entrées `MaybeRefOrGetter<T>` et normaliser avec `toValue`, pour que les appelants puissent passer un ref, un getter, ou une valeur brute.
- Retourner `toRefs(reactive(...))` pour que les consommateurs puissent déstructurer sans perdre la réactivité.
- Un composable qui utilise des hooks de cycle de vie ou `provide` / `inject` doit être appelé à l'intérieur d'un `setup` de composant, pas paresseusement ou conditionnellement.

## Props, Emits, v-model

- `defineProps<Props>()` basé sur les types et `defineEmits<{ change: [id: number] }>()` en forme tuple.
- `defineModel<T>('name', { default })` pour le binding bidirectionnel. Cela se compile en une prop plus un emit `update:*`.

## Provide / Inject

- Utiliser `provide` / `inject` pour les données à portée d'arbre sans prop drilling.
- Clés type-safe sans collision : `const key = Symbol() as InjectionKey<T>`.
- Le provider possède les mutations. Exposer un ref `readonly` plus une fonction de mise à jour explicite, jamais un ref mutable brut.

## Pinia (segment model FSD)

- Préférer les setup stores : `ref` est l'état, `computed` sont les getters, `function` sont les actions.
- Les setup stores n'ont pas `$reset` gratuitement. Définir le vôtre.
- Utiliser `storeToRefs` pour l'état et les getters. Déstructurer les actions directement depuis le store.
- Ne jamais persister les tokens d'authentification bruts dans `localStorage`.

## vue-router

- Charger paresseusement les composants de route avec `import()` dynamique.
- Un garde d'authentification global `beforeEach` basé sur `meta.requiresAuth`. Les gardes retournent `false` (annuler), un emplacement de route (rediriger), ou `undefined` / `true` (continuer).
- Observer `() => route.params.id`, pas l'objet `route` entier.

## vue-query (cache serveur)

- `@tanstack/vue-query` possède l'état de cache serveur. Pinia possède l'état client.
- Placer les fonctions de requête plus les factories `queryOptions` dans le segment `api` FSD.
- Critique : mettre le ref ou computed LUI-MÊME dans la clé de requête, jamais `.value`. Passer `.value` fige la clé et tue le refetch réactif.

```ts
useQuery({ queryKey: ['auction', id], queryFn: () => fetchAuction(toValue(id)) })
// après une mutation
queryClient.invalidateQueries({ queryKey: ['auction', id] })
```

## Référence

- Skills ECC : `frontend-patterns`, `vite-patterns`.
- Docs : <https://pinia.vuejs.org/> · <https://router.vuejs.org/> · <https://tanstack.com/query/latest/docs/framework/vue/overview> · <https://vuejs.org/guide/reusability/composables.html>
