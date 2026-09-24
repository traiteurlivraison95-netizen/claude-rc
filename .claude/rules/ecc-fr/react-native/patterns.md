---
paths:
  - "**/*.ts"
  - "**/*.tsx"
---
# Patterns React Native / Expo

> Ce fichier étend [common/patterns.md](../common/patterns.md) avec des patterns spécifiques à React Native / Expo.
> Remarque : NE PAS installer le jeu de règles `web/` dans un projet React Native — ces patterns supposent le DOM (ex. l'URL comme état) et ne s'appliquent pas ici.

## Navigation (Expo Router)

Expo Router est le routeur intégré d'Expo, basé sur les fichiers (répertoire `app/`) ; React Navigation en est l'alternative établie. Les exemples ci-dessous utilisent Expo Router ; les principes s'appliquent dans les deux cas.

- Garder les fichiers de route (`app/**`) minces — ils relient les params + hooks à un composant d'écran qui vit dans `components/` ou `features/`.
- Typer les params de route ; valider les params non fiables (ex. issus de deep links) avec Zod avant utilisation.
- Utiliser des helpers de navigation typés (`useLocalSearchParams`, `Link`, `router.push`).
- Centraliser la configuration de liaison (linking) ; ne jamais faire confiance aux params de deep link sans validation.

```tsx
// app/user/[id].tsx
import { useLocalSearchParams, router } from 'expo-router'
import { z } from 'zod'

const Params = z.object({ id: z.string().uuid() })

export default function UserScreen() {
  // Utiliser safeParse, pas parse : un deep link malformé lèverait sinon une exception
  // pendant le rendu et ferait planter l'écran. Rediriger plutôt que de lever une exception.
  const parsed = Params.safeParse(useLocalSearchParams())
  if (!parsed.success) {
    router.replace('/not-found')
    return null
  }
  return <UserProfile userId={parsed.data.id} />
}
```

## Gestion de l'état

La règle est de garder ces préoccupations séparées et de ne pas dupliquer les données serveur dans des stores client. Les outils listés sont des choix courants, pas des exigences — choisir ce qui convient à votre projet.

| Préoccupation | Choix courants |
|---------|---------|
| État serveur | une bibliothèque de cache serveur (TanStack Query, SWR) |
| État client/UI | un store léger (Zustand, Jotai) ou Context |
| État de navigation/route | params Expo Router (PAS un store global) |
| État de formulaire | une bibliothèque de formulaire (ex. React Hook Form) avec validation par schéma |
| Persistance sécurisée | `expo-secure-store` |
| Persistance non sécurisée | `AsyncStorage` / MMKV |

- Dériver les valeurs plutôt que de stocker un état calculé redondant.
- Garder l'état client global minimal ; préférer `useState` local jusqu'à ce que le partage soit réellement nécessaire.

## Récupération de données

Utiliser une bibliothèque de cache serveur (TanStack Query, SWR) plutôt qu'un fetch ad hoc dans `useEffect`. Les exemples utilisent TanStack Query.

- Faire transiter les lectures serveur par le cache (ex. `useQuery`) et les mutations également (ex. `useMutation`) avec invalidation du cache.
- Valider les réponses d'API avec Zod à la frontière ; inférer les types depuis le schéma. (Zod est déjà la valeur par défaut de validation dans les règles `typescript/` d'ECC.)
- Gérer explicitement les trois états dans l'UI : chargement, erreur, vide.
- Utiliser des mises à jour optimistes pour des interactions rapides : instantané, application, retour en arrière en cas d'échec avec un retour visible.
- Récupérer les données indépendantes en parallèle ; éviter les cascades de requêtes entre parent et enfant.

```tsx
function useUser(id: string) {
  return useQuery({
    queryKey: ['user', id],
    queryFn: async () => userSchema.parse(await api.getUser(id)),
  })
}
```

## Listes

- Utiliser `FlatList`/`SectionList` (ou `FlashList` pour les listes volumineuses/lourdes) — ne jamais faire `.map()` sur un grand tableau à l'intérieur d'un `ScrollView`.
- Fournir un `keyExtractor` stable ; mémoïser `renderItem`.
- Paginer ou virtualiser les longs jeux de données.

## Hooks personnalisés

- Extraire la logique réutilisable (données, permissions, API de l'appareil) dans des hooks `use*`.
- Garder les effets de bord (appels au SDK Expo, abonnements) à l'intérieur des hooks, pas dans le JSX.

## Async et effets

- Nettoyer les abonnements, timers, et écouteurs dans la fonction de retour de l'effet.
- Annuler ou ignorer les résultats asynchrones obsolètes au démontage pour éviter un setState-après-démontage.
