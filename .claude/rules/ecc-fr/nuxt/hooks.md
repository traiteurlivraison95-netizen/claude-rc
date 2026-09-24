---
paths:
  - "**/nuxt.config.*"
  - "**/app.config.*"
  - "**/server/**/*.ts"
  - "**/*.vue"
---

# Hooks Nuxt

> Ce fichier étend [common/hooks.md](../common/hooks.md) avec du contenu spécifique à Nuxt.

Ce sont des hooks du harnais Claude Code pour le travail Nuxt. Ils s'exécutent via le harnais, pas Claude.

## Vérification de types (Typecheck)

- `nuxi typecheck` encapsule `vue-tsc`. Nécessite les dépendances de développement `vue-tsc` + `typescript`.
- À exécuter sur édition de `.vue` / `.ts` ou en pre-commit. Le typecheck porte sur tout le projet, donc il faut le débouncer et l'envelopper dans un timeout (à l'image de `web/hooks.md`, par exemple `timeout 60 nuxi typecheck`) afin qu'un typecheck bloqué soit interrompu au lieu de s'accumuler à travers des éditions rapides.

## Lint

- Utiliser le module `@nuxt/eslint` (flat-config, conscient du projet, génère `.nuxt/eslint.config.mjs`).
- Exécuter `eslint .` ou `eslint --fix`. C'est l'intégration ESLint officielle de Nuxt, à préférer aux configurations écrites à la main.

## Format

- `prettier --write`, ou activer les règles stylistiques dans `@nuxt/eslint` pour éviter un conflit Prettier/ESLint.
- Choisir une seule autorité de formatage. Ne pas exécuter Prettier et le stylistique ESLint en même temps.

## Chaîne PostToolUse suggérée

- Sur édition de `app/**` et `server/**` : exécuter `eslint --fix` puis `timeout 60 nuxi typecheck`.
- L'ordre compte : d'abord le lint-fix (mute le fichier), ensuite le typecheck chronométré (vérifie le résultat). Le debouncing s'applique toujours.

## Référence

- Skills ECC : `nuxt4-patterns`, `vite-patterns`.
- [Module @nuxt/eslint](https://eslint.nuxt.com/)
- [nuxi typecheck](https://nuxt.com/docs/api/commands/typecheck)
