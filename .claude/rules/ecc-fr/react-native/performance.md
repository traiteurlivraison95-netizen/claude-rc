---
paths:
  - "**/*.ts"
  - "**/*.tsx"
---
# Performance React Native / Expo

> Ce fichier étend [common/performance.md](../common/performance.md) avec du contenu spécifique à React Native / Expo.

## Rendu

- Mémoïser les composants coûteux avec `React.memo` ; mémoïser les callbacks/valeurs passés aux enfants avec `useCallback`/`useMemo` uniquement là où cela évite de vrais re-rendus.
- Garder l'état du composant local et restreint — remonter l'état trop haut provoque le re-rendu de grands sous-arbres.
- Éviter de créer de nouveaux objets/tableaux/fonctions en ligne dans les props sur les chemins critiques ; cela casse la mémoïsation.
- Diviser les grands écrans afin qu'un changement d'état re-rende le plus petit sous-arbre possible.

## Listes

- Utiliser `FlatList`/`SectionList`, ou `FlashList` (Shopify) pour les listes volumineuses ou hétérogènes.
- Fournir `keyExtractor`, un `renderItem` mémoïsé, et des hauteurs d'éléments stables lorsque c'est possible (`getItemLayout`).
- Ajuster `initialNumToRender`, `windowSize`, `maxToRenderPerBatch` pour les lignes lourdes.
- Ne jamais rendre de grands jeux de données avec `.map()` à l'intérieur d'un `ScrollView`.

## Images et assets

- Utiliser `expo-image` pour la mise en cache, la priorité, et les placeholders ; servir des images de taille appropriée.
- Éviter de charger des images en pleine résolution dans de petites miniatures.

## Animations

- Préférer `react-native-reanimated` (s'exécute sur le thread UI) à l'API `Animated` pilotée par JS.
- Pour l'ancien `Animated`, définir `useNativeDriver: true` lorsque c'est pris en charge.
- Garder les calculs lourds hors du thread JS ; les décharger vers des worklets Reanimated ou des modules natifs.

## Runtime et build

- Construire sur la **New Architecture** (Fabric + TurboModules). C'est la valeur par défaut dans les SDK Expo récents (désactivation encore possible sur SDK 53–54) et c'est obligatoire — impossible à désactiver — à partir du SDK 55+. Vérifier que chaque dépendance native est compatible New-Arch avant la mise en production.
- S'assurer que **Hermes** est activé (par défaut dans Expo moderne) pour un démarrage plus rapide et une mémoire réduite.
- Différer le travail non critique après le premier affichage ; charger paresseusement les écrans/modules lourds.
- Utiliser `InteractionManager.runAfterInteractions` pour le travail qui peut attendre la fin des animations.

## Mesure

- Profiler avec le profileur React DevTools, le profileur d'échantillonnage Hermes, et le moniteur de performance intégré à l'application. (Éviter Flipper — il est déprécié et non pris en charge sur la New Architecture.)
- Surveiller : les longues listes sans virtualisation, les images surdimensionnées, les re-rendus fréquents de l'arbre complet, et le travail synchrone sur le thread JS.
