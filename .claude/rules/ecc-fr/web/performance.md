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
> Ce fichier étend [common/performance.md](../common/performance.md) avec du contenu de performance spécifique au web.

# Règles de performance Web

## Cibles Core Web Vitals

| Métrique | Cible |
|--------|--------|
| LCP | < 2.5s |
| INP | < 200ms |
| CLS | < 0.1 |
| FCP | < 1.5s |
| TBT | < 200ms |

## Budget de bundle

| Type de page | Budget JS (gzippé) | Budget CSS |
|-----------|---------------------|------------|
| Page d'atterrissage | < 150kb | < 30kb |
| Page d'application | < 300kb | < 50kb |
| Microsite | < 80kb | < 15kb |

## Stratégie de chargement

1. Intégrer en ligne le CSS critique au-dessus de la ligne de flottaison quand c'est justifié
2. Précharger uniquement l'image hero et la police principale
3. Différer le CSS ou JS non critique
4. Importer dynamiquement les bibliothèques lourdes

```js
const gsapModule = await import('gsap');
const { ScrollTrigger } = await import('gsap/ScrollTrigger');
```

## Optimisation des images

- `width` et `height` explicites
- `loading="eager"` plus `fetchpriority="high"` uniquement pour le média hero
- `loading="lazy"` pour les assets sous la ligne de flottaison
- Préférer AVIF ou WebP avec repli (fallback)
- Ne jamais livrer d'images source largement au-delà de la taille rendue

## Chargement des polices

- Maximum deux familles de polices sauf exception claire
- `font-display: swap`
- Sous-ensembler (subset) quand possible
- Précharger uniquement le poids/style véritablement critique

## Performance des animations

- N'animer que des propriétés adaptées au compositeur
- Utiliser `will-change` avec parcimonie et le retirer une fois terminé
- Préférer CSS pour les transitions simples
- Utiliser `requestAnimationFrame` ou des bibliothèques d'animation établies pour le mouvement JS
- Éviter le remue-ménage des handlers de scroll ; utiliser IntersectionObserver ou des bibliothèques bien élevées

## Checklist de performance

- [ ] Toutes les images ont des dimensions explicites
- [ ] Aucune ressource bloquant le rendu par accident
- [ ] Aucun décalage de mise en page dû au contenu dynamique
- [ ] Le mouvement reste sur des propriétés adaptées au compositeur
- [ ] Les scripts tiers se chargent en async/defer et uniquement quand nécessaire
