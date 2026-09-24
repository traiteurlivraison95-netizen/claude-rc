---
paths:
  - "**/*.tsx"
  - "**/*.jsx"
  - "**/components/**/*.ts"
  - "**/components/**/*.js"
  - "**/app/**/*.tsx"
  - "**/pages/**/*.tsx"
---
# Patterns React

> Ce fichier étend [typescript/patterns.md](../typescript/patterns.md) et [common/patterns.md](../common/patterns.md) avec du contenu spécifique à React. Pour les règles spécifiques aux hooks, voir [hooks.md](./hooks.md).

## Séparation Container / Présentation

Les composants Container possèdent la récupération de données, l'état et les effets de bord. Les composants de présentation reçoivent des props et effectuent le rendu — pas d'appels de service, pas de hooks au-delà de l'état UI local.

```tsx
// Container — possède les données
export function UserPage({ userId }: { userId: string }) {
  const { data: user, isLoading } = useUser(userId);
  if (isLoading) return <Spinner />;
  if (!user) return <NotFound />;
  return <UserCard user={user} onSelect={handleSelect} />;
}

// Présentation — pur
export function UserCard({ user, onSelect }: { user: User; onSelect: (id: string) => void }) {
  return <button onClick={() => onSelect(user.id)}>{user.name}</button>;
}
```

## Arbre de décision pour la localisation de l'état

1. Utilisé par un seul composant → `useState` à l'intérieur
2. Utilisé par le parent + quelques enfants → remonter vers l'ancêtre commun le plus proche, passer via des props
3. Utilisé à travers des branches distantes → React Context **uniquement pour des lectures à faible fréquence** (thème, authentification, locale)
4. Mises à jour à haute fréquence partagées dans l'arbre → store externe (Zustand, Jotai, Redux Toolkit)
5. Données dérivées du serveur → bibliothèque d'état serveur (TanStack Query, SWR, fetch RSC) — pas de l'état applicatif

Un Context mal utilisé pour des valeurs changeant fréquemment provoque le re-rendu de chaque consommateur à chaque mise à jour.

## Frontière Server / Client Component (RSC, Next.js App Router)

- Les Server Components sont le comportement par défaut — ils s'exécutent sur le serveur, ne sont pas envoyés au client, et peuvent faire `await` directement
- Les Client Components s'activent avec `"use client"` en haut du fichier
- Les données descendent : un Server Component peut rendre un Client Component et lui passer des props sérialisables
- Un Client Component ne peut pas importer un Server Component, mais peut en recevoir un via `children` ou des slots nommés

```tsx
// Serveur (par défaut)
export default async function Page() {
  const user = await fetchUser();
  return <UserClient user={user} />;
}

// Client
"use client";
export function UserClient({ user }: { user: User }) {
  const [tab, setTab] = useState("profile");
  return <Tabs value={tab} onChange={setTab}>{user.name}</Tabs>;
}
```

- Ne jamais importer des packages `"server-only"` (clients BD, secrets) depuis un fichier Client Component — les envelopper dans un Server Component ou une Server Action
- Marquer les modules sensibles avec `import "server-only"` afin que le bundler génère une erreur si un fichier client les importe

## Suspense + Error Boundaries

Chaque frontière Suspense a besoin d'un Error Boundary au-dessus. Le duo gère les deux états.

```tsx
<ErrorBoundary fallback={<ErrorView />}>
  <Suspense fallback={<Skeleton />}>
    <UserDetails id={id} />
  </Suspense>
</ErrorBoundary>
```

- Placer les frontières Suspense près de l'endroit où les données sont nécessaires, pas à la racine de la route
- Plusieurs frontières plus étroites révèlent le contenu chargé progressivement
- L'Error Boundary doit être un composant de classe (React 19 n'a pas encore d'équivalent fonctionnel) OU utiliser un wrapper de bibliothèque tel que `react-error-boundary`

## Formulaires

### Non contrôlés (React 19 + actions de formulaire)

Préférer les champs non contrôlés avec des actions de formulaire lorsque le formulaire a une étape de soumission claire. Le navigateur possède la valeur ; React la lit via `FormData` à la soumission.

```tsx
async function action(formData: FormData) {
  "use server";
  await saveUser({ name: String(formData.get("name")) });
}

export function UserForm() {
  return (
    <form action={action}>
      <input name="name" required />
      <button type="submit">Save</button>
    </form>
  );
}
```

### Contrôlés

Utiliser des champs contrôlés lorsque la valeur pilote d'autres éléments d'UI, nécessite une validation en temps réel, ou du formatage.

```tsx
const [email, setEmail] = useState("");
return <input value={email} onChange={(e) => setEmail(e.target.value)} />;
```

### Bibliothèques de formulaires

Pour les formulaires complexes (multi-étapes, tableaux de champs dynamiques, validation croisée entre champs), utiliser une bibliothèque :

- React Hook Form — re-rendus minimaux, priorité au non contrôlé
- TanStack Form — typé, indépendant du framework
- Final Form — lorsque les re-rendus basés sur abonnement comptent

## Récupération de données

| Stratégie | Quand |
|---|---|
| Fetch RSC (`await` dans un Server Component) | Données par requête dans Next.js App Router, aucun cache côté client nécessaire |
| TanStack Query | Cache côté client, mutations, mises à jour optimistes, polling |
| SWR | Cache léger + revalidation, plus simple que TanStack Query |
| `fetch` dans `useEffect` | À éviter — conditions de course, pas de cache, pas de nouvelle tentative. Acceptable uniquement pour un envoi ponctuel sans suivi (fire-and-forget) |

Ne jamais faire de fetch dans un `useEffect` lorsqu'une véritable bibliothèque de cache est disponible — elles gèrent la déduplication, l'invalidation du cache, la nouvelle tentative en cas d'erreur, et l'intégration avec Suspense.

## Listes et clés

- La `key` doit être stable entre les rendus — jamais `index` pour une liste qui peut être réordonnée, avoir des insertions ou des suppressions
- La `key` doit être unique parmi les frères, pas globalement
- Une liste réordonnée avec des clés d'index provoque l'attachement de l'état des composants enfants à la mauvaise ligne

## Composition plutôt qu'héritage

- Passer `children` pour une composition de type slot
- Passer des fonctions render-prop pour un rendu paramétré
- Passer des types de composants pour des points d'extension : `renderItem={UserRow}`
- Ne jamais étendre une classe de composant pour spécialiser un comportement

## Compound Components

Pour des contrôles liés (Tabs, Accordion, Menu), utiliser des compound components partageant l'état via Context :

```tsx
<Tabs defaultValue="profile">
  <Tabs.List>
    <Tabs.Trigger value="profile">Profile</Tabs.Trigger>
    <Tabs.Trigger value="settings">Settings</Tabs.Trigger>
  </Tabs.List>
  <Tabs.Panel value="profile"><ProfileForm /></Tabs.Panel>
  <Tabs.Panel value="settings"><SettingsForm /></Tabs.Panel>
</Tabs>
```

## Portails

Utiliser `createPortal` pour les modales, tooltips, conteneurs de toasts — tout ce qui doit échapper au `overflow: hidden` du parent ou à son contexte d'empilement `z-index`. Effectuer le rendu vers un nœud DOM stable monté dans `index.html`.

## Refs et transfert (React 19+)

React 19 permet aux composants fonctionnels d'accepter `ref` comme une prop ordinaire — `forwardRef` n'est plus nécessaire.

```tsx
export function Input({ ref, ...rest }: { ref?: React.Ref<HTMLInputElement> } & InputProps) {
  return <input ref={ref} {...rest} />;
}
```

Les bases de code plus anciennes sur React 18 ont toujours besoin de `forwardRef`.

## Hors périmètre (sections de renvoi)

### Next.js (App Router)

- Server Actions, Route Handlers, Middleware, routes parallèles/interceptées, Metadata en streaming
- Traité comme une préoccupation de framework distincte — lors de l'ajout de patterns spécifiques à Next en profondeur, proposer une piste dédiée `rules/nextjs/`
- Pour l'instant, suivre la documentation officielle de Next.js pour les spécificités de l'App Router

### React Native

- Imports spécifiques à la plateforme (`Platform.OS`, `.ios.tsx` / `.android.tsx`), `StyleSheet`, bibliothèques de navigation (React Navigation, Expo Router)
- Traité comme une piste distincte — `rules/react-native/` n'est pas encore présent
- Les hooks/patterns React de base de ce fichier s'appliquent toujours

## Référence des compétences

Pour des approfondissements spécifiques à React, voir `skills/react-patterns/SKILL.md`. Pour les préoccupations frontend transversales aux frameworks, voir `skills/frontend-patterns/SKILL.md`. Pour l'accessibilité, voir `skills/accessibility/SKILL.md`.
