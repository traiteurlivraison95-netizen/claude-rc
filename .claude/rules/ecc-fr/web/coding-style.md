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
> Ce fichier étend [common/coding-style.md](../common/coding-style.md) avec du contenu frontend spécifique au web.

# Style de code Web

## Organisation des fichiers

Organiser par fonctionnalité ou surface, pas par type de fichier :

```text
src/
├── components/
│   ├── hero/
│   │   ├── Hero.tsx
│   │   ├── HeroVisual.tsx
│   │   └── hero.css
│   ├── scrolly-section/
│   │   ├── ScrollySection.tsx
│   │   ├── StickyVisual.tsx
│   │   └── scrolly.css
│   └── ui/
│       ├── Button.tsx
│       ├── SurfaceCard.tsx
│       └── AnimatedText.tsx
├── hooks/
│   ├── useReducedMotion.ts
│   └── useScrollProgress.ts
├── lib/
│   ├── animation.ts
│   └── color.ts
└── styles/
    ├── tokens.css
    ├── typography.css
    └── global.css
```

## Propriétés personnalisées CSS

Définir les tokens de design comme des variables. Ne pas coder en dur la palette, la typographie, ou l'espacement de manière répétée :

```css
:root {
  --color-surface: oklch(98% 0 0);
  --color-text: oklch(18% 0 0);
  --color-accent: oklch(68% 0.21 250);

  --text-base: clamp(1rem, 0.92rem + 0.4vw, 1.125rem);
  --text-hero: clamp(3rem, 1rem + 7vw, 8rem);

  --space-section: clamp(4rem, 3rem + 5vw, 10rem);

  --duration-fast: 150ms;
  --duration-normal: 300ms;
  --ease-out-expo: cubic-bezier(0.16, 1, 0.3, 1);
}
```

## Propriétés adaptées à l'animation

Préférer le mouvement adapté au compositeur :
- `transform`
- `opacity`
- `clip-path`
- `filter` (avec parcimonie)

Éviter d'animer des propriétés liées à la mise en page :
- `width`
- `height`
- `top`
- `left`
- `margin`
- `padding`
- `border`
- `font-size`

## HTML sémantique d'abord

```html
<header>
  <nav aria-label="Main navigation">...</nav>
</header>
<main>
  <section aria-labelledby="hero-heading">
    <h1 id="hero-heading">...</h1>
  </section>
</main>
<footer>...</footer>
```

Ne pas recourir à des piles de `div` génériques quand un élément sémantique existe.

## Nommage

- Composants : PascalCase (`ScrollySection`, `SurfaceCard`)
- Hooks : préfixe `use` (`useReducedMotion`)
- Classes CSS : kebab-case ou classes utilitaires
- Timelines d'animation : camelCase avec intention (`heroRevealTl`)
