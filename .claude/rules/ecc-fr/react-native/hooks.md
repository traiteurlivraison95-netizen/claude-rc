---
paths:
  - "**/*.ts"
  - "**/*.tsx"
---
# Hooks React Native / Expo

> Ce fichier étend [common/hooks.md](../common/hooks.md) avec des directives d'automatisation spécifiques à React Native / Expo.

Voici des automatisations PostToolUse recommandées pour garder le code RN/Expo sain. Les brancher dans votre système d'exécution de hooks (ou les exécuter manuellement) ; adapter les commandes à votre gestionnaire de paquets.

## Vérifications PostToolUse suggérées (lors de la modification de *.ts/*.tsx)

- **Vérification de type :** `tsc --noEmit` — détecter les erreurs de type tôt.
- **Lint :** `npx expo lint` (utilise `eslint-config-expo` ; la configuration plate `eslint.config.js` est la valeur par défaut à partir du SDK 53+).
- **Formatage :** `prettier --write` sur les fichiers modifiés.

## Pré-release / périodique

- `npx expo-doctor` — valide la santé et la configuration des dépendances Expo/natives.
- `npx expo install --check` — garde les dépendances natives alignées avec le SDK Expo installé.
- `npm audit` — analyse de vulnérabilités des dépendances.

## Remarques

- Ne pas exécuter de builds natifs lourds dans des hooks d'édition rapides ; limiter les hooks au moment de l'édition au typecheck/lint/formatage.
- Réserver `eas build` / l'E2E aux commandes explicites ou à la CI, pas à l'automatisation par édition.
- Garder cela cohérent avec les contrôles d'exécution des hooks ECC (`ECC_HOOK_PROFILE`, `ECC_DISABLED_HOOKS`).
