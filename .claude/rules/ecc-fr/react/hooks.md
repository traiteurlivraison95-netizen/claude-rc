---
paths:
  - "**/*.tsx"
  - "**/*.jsx"
  - "**/hooks/**/*.ts"
  - "**/hooks/**/*.js"
  - "**/use-*.ts"
  - "**/use-*.tsx"
---
# Hooks React

> Ce fichier couvre les **hooks React** (`useState`, `useEffect`, `useMemo`, `useCallback`, hooks personnalisés) — PAS le système d'exécution `hooks/` de Claude Code. Le nommage suit la convention par langage `rules/<lang>/hooks.md` utilisée dans tout ce dépôt.
>
> Étend [typescript/patterns.md](../typescript/patterns.md) et [common/patterns.md](../common/patterns.md).

## Règles des hooks

Appliquer `eslint-plugin-react-hooks` avec `react-hooks/rules-of-hooks` réglé sur error.

1. Les hooks uniquement au niveau supérieur d'un composant fonctionnel ou d'un autre hook
2. Jamais dans des boucles, des conditions, des fonctions imbriquées, ou après des retours anticipés
3. Toujours appelés dans le même ordre à chaque rendu
4. Uniquement à l'intérieur de composants fonctionnels React ou de hooks personnalisés (fonctions commençant par `use`)

```tsx
// INCORRECT : hook conditionnel
function Foo({ enabled }: { enabled: boolean }) {
  if (enabled) {
    const [x, setX] = useState(0); // violation de la règle
  }
}

// CORRECT : hook inconditionnel, condition à l'intérieur
function Foo({ enabled }: { enabled: boolean }) {
  const [x, setX] = useState(0);
  if (!enabled) return null;
  return <span>{x}</span>;
}
```

## `useEffect` — Quand ne PAS l'utiliser

`useEffect` sert à synchroniser avec des systèmes externes (abonnements, API navigateur, bibliothèques tierces). Ce n'est **pas** le bon outil pour :

- État dérivé — le calculer pendant le rendu
- Transformer des données pour le rendu — les calculer pendant le rendu
- Réinitialiser l'état lorsqu'une prop change — utiliser une `key` sur le parent ou dériver des props
- Notifier les parents des changements d'état — appeler le callback dans le gestionnaire d'événement
- Initialiser des singletons au niveau de l'application — appeler la fonction côté module ou dans `main.tsx`

```tsx
// INCORRECT : effet pour un état dérivé
const [fullName, setFullName] = useState("");
useEffect(() => {
  setFullName(`${first} ${last}`);
}, [first, last]);

// CORRECT : dériver pendant le rendu
const fullName = `${first} ${last}`;
```

## Tableaux de dépendances

- Toujours inclure chaque valeur réactive référencée à l'intérieur de l'effet/callback
- Activer la règle de lint `react-hooks/exhaustive-deps` — ne jamais la désactiver sans un commentaire expliquant pourquoi
- Si le tableau de dépendances devient ingérable, l'effet fait trop de choses — le diviser
- Identité stable pour les fonctions passées en dépendances : envelopper dans `useCallback` uniquement lorsque la fonction est elle-même une dépendance d'un autre hook ou passée à un enfant mémoïsé

## Nettoyage

Chaque abonnement, intervalle, écouteur, ou requête en cours doit être nettoyé.

```tsx
useEffect(() => {
  const controller = new AbortController();
  fetch(url, { signal: controller.signal }).then(handleResponse);
  return () => controller.abort();
}, [url]);
```

```tsx
useEffect(() => {
  const id = setInterval(tick, 1000);
  return () => clearInterval(id);
}, []);
```

L'absence de nettoyage entraîne des conditions de course lorsque les dépendances changent, et des fuites mémoire au démontage.

## `useMemo` et `useCallback` — Quand cela vaut la peine

Position par défaut : **ne pas mémoïser**. Ajouter `useMemo` / `useCallback` uniquement lorsque :

1. La valeur est passée en prop à un enfant enveloppé par `React.memo`, et l'identité compte
2. La valeur est une dépendance d'un autre `useEffect` / `useMemo` / `useCallback`
3. Le calcul est mesurablement coûteux (profiler avant de supposer)

La mémoïsation prématurée ajoute du bruit, masque des bugs, et peut être plus lente que le recalcul qu'elle remplace.

## Hooks personnalisés

Extraire un hook personnalisé lorsque :

- La même séquence de hooks (état + effet + calcul) apparaît dans 2 composants ou plus
- La logique a un objectif clair et nommable (`useDebounce`, `useOnClickOutside`, `useLocalStorage`)
- Vous voulez tester la logique indépendamment de tout composant

Ne PAS extraire lorsque :

- Il n'aurait qu'un seul appelant — le mettre en ligne
- Le « hook » n'est que `useState` sous un autre nom — ajoute de l'indirection, aucune valeur

```tsx
export function useDebounce<T>(value: T, delay: number): T {
  const [debounced, setDebounced] = useState(value);
  useEffect(() => {
    const id = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(id);
  }, [value, delay]);
  return debounced;
}
```

## Patterns `useState`

- État initial issu d'une prop uniquement au montage : passer une fonction `useState(() => computeInitial(prop))` lorsque le calcul est coûteux
- Mise à jour fonctionnelle lorsque le nouvel état dépend de l'ancien : `setCount(c => c + 1)` — jamais `setCount(count + 1)` dans des contextes asynchrones ou groupés (batched)
- Regrouper un état lié dans un seul objet uniquement lorsqu'il change toujours ensemble ; sinon, le diviser en plusieurs appels `useState`
- Utiliser `useReducer` dès que les transitions d'état dépendent de l'état précédent ou qu'il y a 3 valeurs liées ou plus

## Patterns `useRef`

- Refs DOM pour les API impératives (focus, scroll, bibliothèques tierces)
- Conteneur mutable qui ne déclenche pas de re-rendu (ids de timer, valeurs précédentes, indicateurs « is mounted »)
- Ne jamais lire ou écrire `ref.current` pendant le rendu — uniquement à l'intérieur des effets ou des gestionnaires d'événements
- `useImperativeHandle` uniquement pour exposer une API enfant à une ref parente — échappatoire de dernier recours

## `useSyncExternalStore`

Utiliser ce hook pour s'abonner à n'importe quel store externe (API navigateur, bibliothèque d'état tierce, émetteur d'événements personnalisé). C'est la façon prise en charge de rendre un état externe sûr avec le rendu concurrent.

```tsx
const isOnline = useSyncExternalStore(
  (cb) => {
    window.addEventListener("online", cb);
    window.addEventListener("offline", cb);
    return () => {
      window.removeEventListener("online", cb);
      window.removeEventListener("offline", cb);
    };
  },
  () => navigator.onLine,
  () => true,
);
```

## Nouveautés de React 19

- `use()` — déballe les promesses et les contextes en ligne ; utilisable conditionnellement (seul hook avec cette propriété)
- `useFormStatus()` / `useFormState()` (ou `useActionState`) — état de soumission de formulaire sans prop drilling
- `useOptimistic()` — mises à jour d'UI optimistes pendant qu'une action serveur est en attente
- `useTransition()` — marquer les mises à jour d'état non urgentes afin que les urgentes restent réactives

Lorsque le projet cible React 19+, préférer ceux-ci à des équivalents faits main.

## Piège de la closure obsolète (stale closure)

Les gestionnaires asynchrones et les intervalles capturent les valeurs du rendu où ils ont été créés. Corriger en :

1. Utilisant la forme de mise à jour fonctionnelle de `setState`
2. Plaçant la valeur changeante dans le tableau de dépendances de `useEffect` et en reconstruisant le gestionnaire
3. Lisant depuis une ref maintenue synchronisée

## Configuration du lint

Règles requises :

```json
{
  "rules": {
    "react-hooks/rules-of-hooks": "error",
    "react-hooks/exhaustive-deps": "warn"
  }
}
```

Traiter les avertissements `exhaustive-deps` comme des erreurs en CI pour le nouveau code.
