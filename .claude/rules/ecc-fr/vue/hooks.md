---
paths:
  - "**/*.vue"
  - "**/*.ts"
  - "**/*.tsx"
---

# Hooks Vue

> Ce fichier étend [common/hooks.md](../common/hooks.md) avec du contenu spécifique à Vue.

## Cibles PostToolUse

Exécuter sur `*.vue`, `*.ts`, et `*.tsx` après modifications. Cibler les fichiers modifiés quand c'est possible.

## Vérification de types

- Utiliser `vue-tsc --noEmit` pour la vérification SFC plus TypeScript. `tsc` seul ne peut pas lire les composants monofichiers `.vue`, il ne doit donc pas être le hook de vérification de types pour ce projet.
- La vérification de types est à l'échelle du projet. Debouncer ou cibler pour qu'une boucle de sauvegarde à chaque frappe ne bloque pas l'éditeur.

## Lint et formatage

- `eslint --fix` avec `eslint-plugin-vue` (flat-config `vue/vue3-recommended`) couvre le lint du template et du script.
- `prettier --write` pour le formatage. Préférer Prettier via ESLint à une passe Prettier séparée pour éviter le double formatage et les boucles de conflit.

## Frontières d'architecture

- Optionnel : imposer les frontières de tranches Feature-Sliced Design avec `@feature-sliced/steiger` ou `eslint-plugin-boundaries` pour bloquer les imports profonds inter-tranches.

## Séquencement

```bash
# fichiers modifiés uniquement
eslint --fix "$FILE"
prettier --write "$FILE"
# à l'échelle du projet, debouncé
vue-tsc --noEmit
```

- Exécuter le lint et le formatage par fichier d'abord, puis la vérification de types à l'échelle du projet en dernier pour que les erreurs de type reflètent le code source formaté.

## Référence

- Skills ECC : `frontend-patterns`, `vite-patterns`.
- Docs : <https://github.com/vuejs/language-tools> (vue-tsc) · <https://eslint.vuejs.org/> · <https://github.com/feature-sliced/steiger>
