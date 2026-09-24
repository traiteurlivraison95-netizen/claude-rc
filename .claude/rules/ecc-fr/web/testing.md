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
> Ce fichier étend [common/testing.md](../common/testing.md) avec du contenu de test spécifique au web.

# Règles de test Web

## Ordre de priorité

### 1. Régression visuelle

- Capturer les points de rupture clés : 320, 768, 1024, 1440
- Tester les sections hero, les sections scrollytelling, et les états significatifs
- Utiliser les captures d'écran Playwright pour le travail à forte composante visuelle
- Si les deux thèmes existent, tester les deux

### 2. Accessibilité

- Exécuter des vérifications d'accessibilité automatisées
- Tester la navigation au clavier
- Vérifier le comportement de mouvement réduit (reduced-motion)
- Vérifier le contraste des couleurs

### 3. Performance

- Exécuter Lighthouse ou équivalent sur les pages significatives
- Respecter les cibles CWV de [performance.md](performance.md)

### 4. Cross-navigateur

- Minimum : Chrome, Firefox, Safari
- Tester le scroll, le mouvement, et le comportement de repli (fallback)

### 5. Responsive

- Tester 320, 375, 768, 1024, 1440, 1920
- Vérifier l'absence de débordement
- Vérifier les interactions tactiles

## Forme des tests E2E

```ts
import { test, expect } from '@playwright/test';

test('landing hero loads', async ({ page }) => {
  await page.goto('/');
  await expect(page.locator('h1')).toBeVisible();
});
```

- Éviter les assertions basées sur des timeouts instables
- Préférer les attentes déterministes

## Tests unitaires

- Tester les utilitaires, les transformations de données, et les hooks personnalisés
- Pour les composants fortement visuels, la régression visuelle porte souvent plus de signal que des assertions de markup fragiles
- La régression visuelle complète les cibles de couverture ; elle ne les remplace pas
