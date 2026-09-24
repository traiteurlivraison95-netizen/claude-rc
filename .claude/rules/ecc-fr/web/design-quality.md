---
paths:
  - "**/*.css"
  - "**/*.scss"
  - "**/*.sass"
  - "**/*.less"
  - "**/*.html"
  - "**/*.tsx"
  - "**/*.jsx"
  - "**/*.vue"
  - "**/*.svelte"
---
> Ce fichier étend [common/patterns.md](../common/patterns.md) avec des conseils de qualité de design spécifiques au web.

# Standards de qualité de design Web

## Politique anti-template

Ne pas livrer d'UI à l'apparence générique et calquée sur un template. Le rendu frontend doit paraître intentionnel, marqué, et spécifique au produit.

### Patterns bannis

- Grilles de cartes par défaut avec espacement uniforme et sans hiérarchie
- Section hero standard avec titre centré, gradient blob, et CTA générique
- Valeurs par défaut de bibliothèque non modifiées présentées comme un design fini
- Mises en page plates sans stratification, profondeur, ou mouvement
- Rayon, espacement, et ombres uniformes sur tous les composants
- Style gris-sur-blanc sûr avec une seule couleur d'accent décorative
- Mises en page « dashboard par les chiffres » avec barre latérale + cartes + graphiques sans point de vue
- Empilements de polices par défaut utilisés sans raison délibérée

### Qualités requises

Chaque surface frontend significative devrait démontrer au moins quatre de ces éléments :

1. Hiérarchie claire par contraste d'échelle
2. Rythme intentionnel dans l'espacement, pas un padding uniforme partout
3. Profondeur ou stratification par chevauchement, ombres, surfaces, ou mouvement
4. Typographie avec du caractère et une vraie stratégie de pairing
5. Couleur utilisée sémantiquement, pas seulement de manière décorative
6. États hover, focus, et actif qui paraissent conçus
7. Composition éditoriale ou bento qui casse la grille quand c'est approprié
8. Texture, grain, ou atmosphère quand cela correspond à la direction visuelle
9. Mouvement qui clarifie le flux plutôt que de distraire
10. Visualisation de données traitée comme partie du système de design, pas comme un après-coup

## Avant d'écrire du code frontend

1. Choisir une direction stylistique spécifique. Éviter les valeurs par défaut vagues comme « clean minimal ».
2. Définir une palette de manière intentionnelle.
3. Choisir la typographie délibérément.
4. Rassembler au moins un petit ensemble de références réelles.
5. Utiliser les skills ECC design/frontend le cas échéant.

## Directions stylistiques valables

- Éditorial / magazine
- Néo-brutalisme
- Glassmorphisme avec une réelle profondeur
- Luxe sombre ou luxe clair avec un contraste discipliné
- Mises en page bento
- Scrollytelling
- Intégration 3D
- Swiss / International
- Rétro-futurisme

Ne pas passer automatiquement au mode sombre par défaut. Choisir la direction visuelle que le produit veut réellement.

## Checklist de composant

- [ ] Évite-t-il de ressembler à un template Tailwind ou shadcn par défaut ?
- [ ] A-t-il des états hover/focus/actif intentionnels ?
- [ ] Utilise-t-il la hiérarchie plutôt qu'une emphase uniforme ?
- [ ] Serait-il crédible dans une vraie capture d'écran de produit ?
- [ ] S'il supporte les deux thèmes, le clair et le sombre paraissent-ils tous deux intentionnels ?
