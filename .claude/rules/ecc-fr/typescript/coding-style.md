---
paths:
  - "**/*.ts"
  - "**/*.tsx"
  - "**/*.js"
  - "**/*.jsx"
---
# Style de code TypeScript/JavaScript

> Ce fichier étend [common/coding-style.md](../common/coding-style.md) avec du contenu spécifique à TypeScript/JavaScript.

## Types et interfaces

Utiliser des types pour rendre les API publiques, les modèles partagés, et les props de composants explicites, lisibles, et réutilisables.

### API publiques

- Ajouter des types de paramètre et de retour aux fonctions exportées, aux utilitaires partagés, et aux méthodes de classe publiques
- Laisser TypeScript inférer les types de variables locales évidents
- Extraire les formes d'objets inline répétées en types ou interfaces nommés

```typescript
// FAUX : fonction exportée sans types explicites
export function formatUser(user) {
  return `${user.firstName} ${user.lastName}`
}

// CORRECT : types explicites sur les API publiques
interface User {
  firstName: string
  lastName: string
}

export function formatUser(user: User): string {
  return `${user.firstName} ${user.lastName}`
}
```

### Interfaces vs. alias de type

- Utiliser `interface` pour les formes d'objets susceptibles d'être étendues ou implémentées
- Utiliser `type` pour les unions, intersections, tuples, mapped types, et types utilitaires
- Préférer les unions de littéraux de chaîne à `enum` sauf si un `enum` est requis pour l'interopérabilité

```typescript
interface User {
  id: string
  email: string
}

type UserRole = 'admin' | 'member'
type UserWithRole = User & {
  role: UserRole
}
```

### Éviter `any`

- Éviter `any` dans le code d'application
- Utiliser `unknown` pour les entrées externes ou non fiables, puis les restreindre de manière sûre
- Utiliser des génériques quand le type d'une valeur dépend de l'appelant

```typescript
// FAUX : any supprime la sécurité de type
function getErrorMessage(error: any) {
  return error.message
}

// CORRECT : unknown force une restriction sûre
function getErrorMessage(error: unknown): string {
  if (error instanceof Error) {
    return error.message
  }

  return 'Unexpected error'
}
```

### Props React

- Définir les props de composant avec une `interface` ou un `type` nommé
- Typer explicitement les props de callback
- Ne pas utiliser `React.FC` sauf raison spécifique

```typescript
interface User {
  id: string
  email: string
}

interface UserCardProps {
  user: User
  onSelect: (id: string) => void
}

function UserCard({ user, onSelect }: UserCardProps) {
  return <button onClick={() => onSelect(user.id)}>{user.email}</button>
}
```

### Fichiers JavaScript

- Dans les fichiers `.js` et `.jsx`, utiliser JSDoc quand les types améliorent la clarté et qu'une migration TypeScript n'est pas pratique
- Garder le JSDoc aligné avec le comportement à l'exécution

```javascript
/**
 * @param {{ firstName: string, lastName: string }} user
 * @returns {string}
 */
export function formatUser(user) {
  return `${user.firstName} ${user.lastName}`
}
```

## Immutabilité

Utiliser l'opérateur spread pour les mises à jour immuables :

```typescript
interface User {
  id: string
  name: string
}

// FAUX : mutation
function updateUser(user: User, name: string): User {
  user.name = name // MUTATION !
  return user
}

// CORRECT : immutabilité
function updateUser(user: Readonly<User>, name: string): User {
  return {
    ...user,
    name
  }
}
```

## Gestion des erreurs

Utiliser async/await avec try-catch et restreindre les erreurs unknown de manière sûre :

```typescript
interface User {
  id: string
  email: string
}

declare function riskyOperation(userId: string): Promise<User>

function getErrorMessage(error: unknown): string {
  if (error instanceof Error) {
    return error.message
  }

  return 'Unexpected error'
}

const logger = {
  error: (message: string, error: unknown) => {
    // Remplacer par votre logger de production (par exemple, pino ou winston).
  }
}

async function loadUser(userId: string): Promise<User> {
  try {
    const result = await riskyOperation(userId)
    return result
  } catch (error: unknown) {
    logger.error('Operation failed', error)
    throw new Error(getErrorMessage(error))
  }
}
```

## Validation des entrées

Utiliser Zod pour la validation basée sur des schémas et inférer les types depuis le schéma :

```typescript
import { z } from 'zod'

const userSchema = z.object({
  email: z.string().email(),
  age: z.number().int().min(0).max(150)
})

type UserInput = z.infer<typeof userSchema>

const validated: UserInput = userSchema.parse(input)
```

## Console.log

- Aucune instruction `console.log` dans le code de production
- Utiliser des bibliothèques de journalisation appropriées à la place
- Voir les hooks pour la détection automatique
