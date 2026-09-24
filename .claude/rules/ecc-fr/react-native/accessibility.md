---
paths:
  - "**/*.ts"
  - "**/*.tsx"
---
# Accessibilité React Native / Expo

> Étend le niveau d'exigence de qualité ECC à l'accessibilité (a11y). Traiter l'a11y comme une exigence de mise en production, pas comme une réflexion après coup.
> Objectif : utilisable avec les lecteurs d'écran (VoiceOver sur iOS, TalkBack sur Android) et avec de grandes tailles de police.

## Étiquetage

- Chaque élément interactif a un `accessibilityRole` et un `accessibilityLabel` (ou un texte enfant lisible).
- Les boutons uniquement composés d'une icône DOIVENT avoir un `accessibilityLabel` — il n'y a pas de texte visible que le lecteur puisse annoncer.
- Utiliser `accessibilityHint` uniquement lorsque l'action n'est pas évidente ; le garder court.
- Regrouper les éléments liés avec `accessible` sur le conteneur afin qu'ils soient annoncés comme une seule unité lorsque c'est pertinent.

```tsx
<Pressable
  accessibilityRole="button"
  accessibilityLabel="Delete item"
  onPress={onDelete}
>
  <TrashIcon />
</Pressable>
```

## État et régions dynamiques (live regions)

- Communiquer l'état avec `accessibilityState` (ex. `{ disabled, selected, checked, expanded }`).
- Annoncer les changements asynchrones/transitoires (toasts, erreurs de validation) via `accessibilityLiveRegion` (Android) et `AccessibilityInfo.announceForAccessibility` lorsque nécessaire.
- Refléter les états de chargement/erreur/vide dans un texte accessible au lecteur — pas seulement des spinners ou de la couleur.

## Cibles tactiles et mise en page

- Cible tactile minimale d'environ 44x44pt (iOS) / 48x48dp (Android) ; utiliser `hitSlop` pour agrandir les petits contrôles.
- Respecter le Dynamic Type / la mise à l'échelle des polices — éviter les hauteurs fixes qui tronquent le texte agrandi ; tester avec la plus grande taille de police d'accessibilité.
- Respecter `prefers-reduced-motion` (`AccessibilityInfo.isReduceMotionEnabled`) — conditionner les animations non essentielles.

## Couleur et contraste

- Ne pas transmettre de sens par la seule couleur ; l'associer à du texte, une icône, ou une forme.
- Respecter le contraste WCAG AA : 4,5:1 pour le texte courant, 3:1 pour le grand texte et les éléments UI/graphiques significatifs.
- Vérifier à la fois les thèmes clair et sombre.

## Focus et navigation

- Ordre de focus logique ; déplacer le focus vers le nouveau contenu (modales, écrans) à l'ouverture et le restaurer à la fermeture.
- S'assurer que les composants personnalisés sont accessibles et utilisables par le lecteur d'écran, pas seulement au toucher.

## Tests

- Tester manuellement avec VoiceOver et TalkBack sur des appareils réels — les vérifications automatisées ne détectent pas tout.
- Dans les tests de composants, interroger par rôle/label (voir testing.md) afin que l'a11y et les tests se renforcent mutuellement.
- Ajouter l'a11y à la porte de pré-release : les flux clés passent un parcours au lecteur d'écran.
