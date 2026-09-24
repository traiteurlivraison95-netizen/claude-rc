---
paths:
  - "**/*.ets"
  - "**/*.ts"
  - "**/module.json5"
  - "**/oh-package.json5"
  - "**/build-profile.json5"
---
# Style de code HarmonyOS / ArkTS

> Ce fichier étend [common/coding-style.md](../common/coding-style.md) avec du contenu spécifique à HarmonyOS et ArkTS.

## Contraintes du langage ArkTS

ArkTS est un sous-ensemble strict et statiquement typé de TypeScript. Violer ces contraintes provoque des **échecs de compilation**.

### Système de types

- Pas de types `any` ou `unknown` - utilise toujours des types explicites
- Pas de types d'accès par index - utilise directement les noms de types
- Pas d'alias de type conditionnels ni de mot-clé `infer`
- Pas de types d'intersection - utilise l'héritage
- Pas de types mappés - utilise des classes et des idiomes classiques
- Pas de `typeof` pour les annotations de type - utilise des déclarations de type explicites
- Pas d'assertions `as const` - utilise des annotations de type explicites
- Pas de typage structurel - utilise l'héritage, les interfaces ou les alias de type
- Aucun type utilitaire TypeScript sauf `Partial`, `Required`, `Readonly`, `Record`
- Pour `Record<K, V>`, le type d'expression d'index est `V | undefined`
- Omets les annotations de type dans les clauses `catch` (ArkTS ne supporte pas `any`/`unknown`)

### Fonctions et classes

- Pas d'expressions de fonction - utilise des fonctions fléchées
- Pas de fonctions imbriquées - utilise des lambdas
- Pas de fonctions génératrices - utilise `async`/`await` pour le multitâche
- Pas de `Function.apply`, `Function.call`, `Function.bind` - suis l'OOP traditionnel pour `this`
- Pas d'expressions de type constructeur - utilise des lambdas
- Pas de signatures de constructeur dans les interfaces ou les types d'objet - utilise des méthodes ou des classes
- Pas de déclaration de champs de classe dans les constructeurs - déclare-les dans le corps de la classe
- Pas de `this` dans les fonctions autonomes ou les méthodes statiques - uniquement dans les méthodes d'instance
- Pas de `new.target`
- Pas d'assertions d'affectation définitive (`let v!: T`) - utilise des déclarations initialisées
- Pas de littéraux de classe - introduis des types de classe nommés
- Pas d'utilisation des classes comme des objets (affectation à des variables) - les déclarations de classe introduisent des types, pas des valeurs
- Un seul bloc statique par classe - fusionne toutes les instructions statiques

### Accès aux objets et propriétés

- Pas de déclaration dynamique de champ ni d'accès `obj["field"]` - utilise la syntaxe `obj.field`
- Pas d'opérateur `delete` - utilise un type nullable avec `null` pour marquer l'absence
- Pas d'affectation de prototype - utilise des classes et des interfaces
- Pas d'opérateur `in` - utilise `instanceof`
- Pas de réaffectation des méthodes d'objet - utilise des fonctions wrapper ou l'héritage
- Pas d'API `Symbol()` (sauf `Symbol.iterator`)
- Pas de `globalThis` ni de portée globale - utilise des exports/imports de module explicites
- Pas de namespaces comme objets - utilise des classes ou des modules
- Pas d'instructions à l'intérieur des namespaces - utilise des fonctions

### Déstructuration et spread

- Pas d'affectations ou de déclarations de variables par déstructuration - utilise des objets intermédiaires et un accès champ par champ
- Pas de déclarations de paramètres par déstructuration - passe les paramètres directement, affecte les noms locaux manuellement
- L'opérateur spread uniquement pour développer des tableaux (ou des classes dérivées de tableau) dans des paramètres rest ou des littéraux de tableau

### Modules et imports

- Pas de `require()` - utilise la syntaxe `import` classique
- Pas de `export = ...` - utilise l'export/import normal
- Pas d'assertions d'import - les imports sont résolus au moment de la compilation en ArkTS
- Pas de modules UMD
- Pas de wildcards dans les noms de module
- Toutes les instructions `import` doivent apparaître avant toute autre instruction
- Les bases de code TypeScript ne doivent pas dépendre de bases de code ArkTS via import (l'inverse est supporté)

### Autres restrictions

- Pas de `var` - utilise `let`
- Pas de boucles `for...in` - utilise des boucles `for` classiques pour les tableaux
- Pas d'instructions `with`
- Pas d'expressions JSX
- Pas d'identifiants privés `#` - utilise le mot-clé `private`
- Pas de fusion de déclarations (classes, interfaces, enums) - garde les définitions compactes
- Pas de signatures d'index - utilise des tableaux
- L'opérateur virgule uniquement dans les boucles `for`
- Les opérateurs unaires `+`, `-`, `~` uniquement pour les types numériques (pas de conversion implicite de chaîne)
- Membres d'enum : uniquement des expressions constantes de même type au moment de la compilation pour les initialiseurs explicites
- L'inférence de type de retour des fonctions est limitée - précise les types de retour explicitement lors de l'appel de fonctions dont le type de retour est omis

### Littéraux d'objet

- Supportés uniquement lorsque le compilateur peut inférer la classe ou l'interface correspondante
- NON supportés pour : les types `any`/`Object`/`object`, les classes/interfaces avec méthodes, les classes avec constructeurs paramétrés, les classes avec champs `readonly`

## Conventions de nommage

- Variables / fonctions : `camelCase` (ex. `getUserInfo`, `goodsList`)
- Classes / interfaces : `PascalCase` (ex. `UserViewModel`, `IGoodsModel`)
- Constantes : `UPPER_SNAKE_CASE` (ex. `MAX_PAGE_SIZE`, `COLOR_PRIMARY`)
- Noms de fichiers : `PascalCase` pour les composants (ex. `HomePage.ets`), `camelCase` pour les utilitaires

## Formatage

- Privilégie les guillemets doubles pour les chaînes
- Points-virgules en fin d'instruction
- N'utilise jamais `var` - privilégie `const`, puis `let`
- Toutes les méthodes, paramètres, valeurs de retour doivent avoir des annotations de type complètes

## Organisation des fichiers

- Fichiers de composants (`.ets`) : un seul `@ComponentV2` par fichier
- Fichiers ViewModel : une seule classe ViewModel par fichier
- Fichiers de modèles : les modèles de données apparentés peuvent partager un fichier
- Garde les fichiers sous 400 lignes ; extrais des helpers pour les fichiers approchant les 800 lignes

## Commentaires

- En-tête de fichier : `@file` (objectif du fichier) + `@author` (développeur), si le projet utilise déjà des en-têtes de fichier
- Méthodes publiques : JSDoc avec `@param`, `@returns` ; ajoute `@example` pour les méthodes complexes
- Aligne-toi sur la langue de documentation déjà en place dans le projet ; utilise l'anglais sauf si le dépôt a déjà standardisé sur des commentaires en chinois

## Gestion des erreurs

```typescript
// Utiliser try/catch avec une gestion d'erreur appropriée
try {
  const result = await riskyOperation()
  return result
} catch (error) {
  hilog.error(0x0000, 'TAG', 'Operation failed: %{public}s', error)
  throw new Error('User-friendly error message')
}
```

## Immutabilité

Suis les principes communs d'immutabilité - crée de nouveaux objets plutôt que de muter :

```typescript
// MAUVAIS : mutation
function updateUser(user: UserModel, name: string): UserModel {
  user.name = name  // mutation directe
  return user
}

// BON : immuable - créer une nouvelle instance
function updateUser(user: UserModel, name: string): UserModel {
  const updated = new UserModel()
  updated.id = user.id
  updated.name = name
  updated.email = user.email
  return updated
}
```
