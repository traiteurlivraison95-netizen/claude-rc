---
paths:
  - "**/*.tsx"
  - "**/*.jsx"
  - "**/components/**/*.ts"
  - "**/components/**/*.js"
  - "**/hooks/**/*.ts"
  - "**/hooks/**/*.js"
---
# Style de code React

> Ce fichier étend [typescript/coding-style.md](../typescript/coding-style.md) et [common/coding-style.md](../common/coding-style.md) avec du contenu spécifique à React.

## Extensions de fichiers

- `.tsx` pour tout fichier contenant du JSX, même de simples extraits d'une ligne
- `.ts` pour la logique pure, les hooks personnalisés sans JSX, les définitions de types, les utilitaires
- `.test.tsx` / `.test.ts` reflétant le fichier source
- Utiliser `.jsx` uniquement lorsque le projet évite intentionnellement TypeScript — signaler tout nouveau fichier React non typé en revue

## Nommage

- Composants : `PascalCase` à la fois pour le symbole et le fichier (`UserCard.tsx`, export par défaut `UserCard`)
- Hooks personnalisés : `useCamelCase` pour le symbole, kebab-case pour le fichier lorsque la convention du projet est kebab-case (`use-debounce.ts` exporte `useDebounce`)
- Contexte : symbole `<Domain>Context`, composant fournisseur `<Domain>Provider`, hook consommateur `use<Domain>`
- Gestionnaires d'événements : `handleClick`, `handleSubmit` à l'intérieur du composant ; la prop qui le reçoit est `onClick`, `onSubmit`
- Props booléennes : `isLoading`, `hasError`, `canSubmit` — jamais `loading` ou `error` seul pour des booléens

## Forme des composants

```tsx
type Props = {
  user: User;
  onSelect: (id: string) => void;
};

export function UserCard({ user, onSelect }: Props) {
  return (
    <button type="button" onClick={() => onSelect(user.id)}>
      {user.name}
    </button>
  );
}
```

- Préférer `type Props = {}` pour des formes de props de composant fermées
- Utiliser `interface` uniquement lorsque le type de prop est étendu via la fusion de déclarations ou exporté comme point d'extension d'API publique
- Toujours déstructurer les props dans la liste de paramètres — aucun accès `props.user` à l'intérieur du corps
- Typer le retour implicitement via JSX (`function Foo(): JSX.Element` uniquement lorsque la fonction retourne conditionnellement et que l'union brouille l'inférence)

## JSX

- Fermer automatiquement les balises sans enfants : `<img />`, `<UserCard user={u} />`
- Utiliser les fragments `<>...</>` plutôt qu'un `<div>` englobant quand aucun élément DOM n'est nécessaire
- Rendu conditionnel : `{condition && <Foo />}` pour les booléens, ternaire pour un choix entre deux options, retour anticipé pour les clauses de garde
- Ne jamais mettre de logique en ligne dans le JSX lorsqu'elle s'étend sur plusieurs lignes — l'extraire dans une const au-dessus du return ou dans une fonction

```tsx
// Préférer
const greeting = user.isAdmin ? "Welcome, admin" : `Hello ${user.name}`;
return <h1>{greeting}</h1>;

// Plutôt que
return <h1>{user.isAdmin ? "Welcome, admin" : `Hello ${user.name}`}</h1>;
```

## Frontière serveur / client (Next.js App Router, RSC)

- Par défaut, un nouveau fichier est un Server Component — n'ajouter `"use client"` que lorsque le fichier utilise de l'état, des effets, des refs, des API navigateur ou des gestionnaires d'événements
- Placer la directive `"use client"` en ligne 1, avant tout import
- Ne jamais importer un fichier Client Component depuis un fichier d'action `"use server"`
- Ne jamais ré-exporter du code réservé au serveur via un module client — le bundler l'inclura silencieusement

## Imports

- Les imports React en premier : `import { useState } from "react"`
- Puis les bibliothèques tierces, puis les imports absolus du projet, puis les relatifs
- Imports de type uniquement : `import type { ReactNode } from "react"` — ne jamais mélanger les imports d'exécution et de type dans une même instruction lorsque la règle ESLint `consistent-type-imports` est configurée

## Discipline des hooks

Voir [hooks.md](./hooks.md) pour l'ensemble complet des règles. Points de style à retenir :

- Les hooks personnalisés doivent commencer par `use` — imposé par `eslint-plugin-react-hooks`
- Regrouper tous les appels de hooks en haut du composant, avant toute logique conditionnelle
- Éviter de créer des hooks ad hoc pour de simples wrappers d'une ligne — mettre l'appel en ligne à la place

## État

- Local en priorité (`useState`), remonter uniquement en cas de partage
- Le Context pour l'état transversal lu par de nombreux composants (thème, authentification, i18n) — pas pour les mises à jour à haute fréquence
- Un store externe (Zustand, Jotai, Redux Toolkit) lorsque l'état doit persister entre les changements de route, se synchroniser entre onglets, ou être déboggé via des devtools
- Ne jamais dupliquer un état qui peut être dérivé — le calculer pendant le rendu

## Composants de classe

Interdits dans le nouveau code. Convertir les anciens composants de classe en composants fonctionnels lorsqu'on les modifie pour des changements non triviaux.

## Organisation des fichiers par composant

```
components/UserCard/
  UserCard.tsx
  UserCard.module.css   # ou styled-components, ou classes Tailwind en ligne
  UserCard.test.tsx
  index.ts              # ré-export uniquement
```

Les composants à fichier unique en ligne conviennent pour des éléments de présentation triviaux.
