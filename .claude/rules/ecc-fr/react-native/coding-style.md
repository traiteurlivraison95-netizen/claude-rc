---
paths:
  - "**/*.ts"
  - "**/*.tsx"
---
# Style de code React Native / Expo

> Ce fichier étend [common/coding-style.md](../common/coding-style.md) avec du contenu spécifique à React Native / Expo.

## Composants

- Définir les props avec une `interface` ou un `type` nommé ; ne pas utiliser `React.FC`.
- Garder les écrans minces : un écran compose des hooks + des composants de présentation, il ne contient pas de logique lourde.
- Un composant par fichier pour tout ce qui est réutilisable ; co-localiser les petits sous-composants privés.
- Préférer les composants fonctionnels et les hooks. Pas de composants de classe.

```tsx
interface AvatarProps {
  uri: string
  size?: number
  onPress?: () => void
}

export function Avatar({ uri, size = 40, onPress }: AvatarProps) {
  return (
    <Pressable onPress={onPress}>
      <Image source={{ uri }} style={{ width: size, height: size, borderRadius: size / 2 }} />
    </Pressable>
  )
}
```

## Style (styling)

Choisir UN SEUL système de style par projet et rester cohérent. `StyleSheet.create()` est l'option native du framework ; les bibliothèques à classes utilitaires (ex. NativeWind) sont une alternative courante. Cette règle est indépendante de la bibliothèque — ce qui compte, c'est la cohérence et l'évitement des allocations en ligne.

- StyleSheet : définir les styles avec `StyleSheet.create()` au niveau du module — ne jamais construire d'objets de style en ligne dans `render`/JSX sur les chemins critiques (cela alloue à chaque rendu).
- Approche à classes utilitaires : extraire les chaînes de classes répétées dans des constantes partagées ou un helper de variantes.
- Ne jamais coder en dur des couleurs brutes, des espacements, ou des tailles de police dispersées dans les fichiers. Centraliser les tokens de design (fichier de thème ou configuration).

```tsx
// INCORRECT : objet de style en ligne recréé à chaque rendu
<View style={{ padding: 16, backgroundColor: '#fff' }} />

// CORRECT (StyleSheet)
const styles = StyleSheet.create({ card: { padding: 16, backgroundColor: '#fff' } })
<View style={styles.card} />

// CORRECT (NativeWind)
<View className="p-4 bg-white" />
```

## Différences de plateforme

- Utiliser des fichiers spécifiques à la plateforme (`Component.ios.tsx`, `Component.android.tsx`) pour des divergences importantes.
- Utiliser `Platform.select()` / `Platform.OS` uniquement pour de petites différences.
- Tenir compte des zones sûres (safe areas) avec `react-native-safe-area-context` ; ne pas coder en dur les décalages de barre de statut / encoche.

## Imports et organisation du projet

- Utiliser l'alias de chemin Expo/TS (ex. `@/components/...`) plutôt que de longues chaînes relatives.
- Organiser par fonctionnalité/domaine, pas par type. Garder les fichiers ciblés (200-400 lignes typiquement, 800 maximum).

## Journalisation

- Pas de `console.log` dans le code livré. Utiliser un logger et retirer les logs dans les builds de production.
- Exposer les erreurs destinées à l'utilisateur via l'état de l'UI, pas la console.

## TypeScript

Toutes les règles TypeScript de `rules/typescript/` s'appliquent (types explicites sur les API publiques, éviter `any`, Zod pour la validation, mises à jour immuables). Ce fichier ne fait qu'ajouter des directives spécifiques à RN en plus.
