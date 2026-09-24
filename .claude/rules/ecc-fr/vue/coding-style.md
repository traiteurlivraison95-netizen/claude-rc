---
paths:
  - "**/*.vue"
---

# Style de code Vue

> Ce fichier étend [common/coding-style.md](../common/coding-style.md) avec du contenu spécifique à Vue.

## Structure SFC

- Toujours `<script setup lang="ts">` avec la Composition API. Pas d'Options API dans le nouveau code.
- Ordre des blocs dans un fichier `.vue` : `<script setup>`, puis `<template>`, puis `<style scoped>`. Un composant par fichier.
- Nommage : fichiers de composants en PascalCase (`AuctionCard.vue`), composables en camelCase préfixés `useXxx` (`useAuctionTimer`).
- Formater avec Prettier plus la configuration flat ESLint utilisant `eslint-plugin-vue` (`vue/vue3-recommended`). Vérifier les types avec `vue-tsc`.

## Discipline de réactivité

- `ref` est l'API d'état principale. Muter via `.value` dans le script, auto-déballé uniquement au niveau supérieur du template.
- Un `ref` imbriqué dans un tableau, `Map`, ou `Set` nécessite toujours `.value` pour être lu.
- Ne recourir à `reactive` que pour l'état d'objet groupé. Ne jamais réassigner un objet `reactive` entier.
- Ne jamais déstructurer un objet `reactive` ou un store Pinia sans `toRefs` / `storeToRefs`. Une déstructuration simple supprime silencieusement la réactivité.

## Computed et watchers

- Les getters `computed` doivent être purs : pas d'effets de bord, pas d'async, pas d'accès au DOM.
- Depuis 3.4+, `computed` ne se déclenche que lorsque la valeur retournée change. Retourner l'objet précédent inchangé quand il est égal pour éviter les mises à jour en aval.
- `watch` est paresseux (lazy). Passer un getter pour une propriété réactive (`watch(() => x.value, ...)`), pas l'objet réactif brut.
- `watchEffect` est immédiat (eager) et arrête de traquer les dépendances après son premier `await`.

## Cycle de vie et DOM

- Enregistrer les hooks de cycle de vie de manière synchrone à l'intérieur de `setup` (`onMounted`, `onUnmounted`).
- Nettoyer les timers, écouteurs, et abonnements dans `onUnmounted`.
- Lire ou mesurer le DOM uniquement après `await nextTick()`.

## Macros et templates

- Macros : `defineProps` / `defineEmits` (forme tuple `change: [id: number]`), `defineModel` (3.4+) pour `v-model`, `withDefaults` ou la déstructuration réactive des props en 3.5+ pour les valeurs par défaut, `defineExpose` pour l'API ref publique.
- Mettre une `:key` sur chaque `v-for`, un primitif unique et stable. Jamais l'index du tableau, jamais un objet.
- Ne jamais mettre `v-if` et `v-for` sur le même élément. Envelopper avec `<template v-for>` plus un `v-if` interne, ou précalculer une liste filtrée.

```vue
<script setup lang="ts">
const props = defineProps<{ id: number }>()
const emit = defineEmits<{ change: [id: number] }>()
const open = defineModel<boolean>('open', { default: false })
</script>
```

## Référence

- Skills ECC : `frontend-patterns`, `vite-patterns`.
- Docs : <https://vuejs.org/api/sfc-script-setup.html> · <https://vuejs.org/guide/essentials/reactivity-fundamentals.html> · <https://eslint.vuejs.org/>
