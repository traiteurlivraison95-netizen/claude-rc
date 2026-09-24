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
> Ce fichier étend [common/patterns.md](../common/patterns.md) avec des patterns spécifiques au web.

# Patterns Web

## Composition de composants

### Composants composés (Compound Components)

Utiliser des composants composés quand une UI liée partage l'état et la sémantique d'interaction :

```tsx
<Tabs defaultValue="overview">
  <Tabs.List>
    <Tabs.Trigger value="overview">Overview</Tabs.Trigger>
    <Tabs.Trigger value="settings">Settings</Tabs.Trigger>
  </Tabs.List>
  <Tabs.Content value="overview">...</Tabs.Content>
  <Tabs.Content value="settings">...</Tabs.Content>
</Tabs>
```

- Le parent possède l'état
- Les enfants consomment via le contexte
- Préférer ceci au prop drilling pour les widgets complexes

### Render Props / Slots

- Utiliser les render props ou les patterns de slot quand le comportement est partagé mais que le markup doit varier
- Garder la gestion du clavier, ARIA, et la logique de focus dans la couche headless

### Séparation Container / Présentationnel

- Les composants container possèdent le chargement de données et les effets de bord
- Les composants présentationnels reçoivent des props et rendent l'UI
- Les composants présentationnels doivent rester purs

## Gestion d'état

Traiter ces éléments séparément :

| Préoccupation | Outillage |
|---------|---------|
| État serveur | TanStack Query, SWR, tRPC |
| État client | Zustand, Jotai, signals |
| État URL | search params, segments de route |
| État de formulaire | React Hook Form ou équivalent |

- Ne pas dupliquer l'état serveur dans des stores client
- Dériver les valeurs plutôt que stocker un état calculé redondant

## L'URL comme état

Persister l'état partageable dans l'URL :
- filtres
- ordre de tri
- pagination
- onglet actif
- requête de recherche

## Récupération de données

### Stale-While-Revalidate

- Retourner les données en cache immédiatement
- Revalider en arrière-plan
- Préférer les bibliothèques existantes à une implémentation maison

### Mises à jour optimistes

- Prendre un instantané de l'état actuel
- Appliquer la mise à jour optimiste
- Revenir en arrière en cas d'échec
- Émettre un retour d'erreur visible lors du rollback

### Chargement parallèle

- Récupérer les données indépendantes en parallèle
- Éviter les cascades de requêtes parent-enfant
- Précharger les routes ou états suivants probables quand c'est justifié
