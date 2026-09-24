---
paths:
  - "**/*.tsx"
  - "**/*.jsx"
  - "**/components/**/*.ts"
  - "**/app/**/*.ts"
  - "**/pages/**/*.ts"
---
# Sécurité React

> Ce fichier étend [typescript/security.md](../typescript/security.md) et [common/security.md](../common/security.md) avec du contenu spécifique à React.

## XSS via `dangerouslySetInnerHTML`

CRITIQUE. Le nom de la prop est délibérément effrayant — traiter chaque utilisation comme un point d'arrêt en revue de code.

```tsx
// CRITIQUE : entrée utilisateur non nettoyée
<div dangerouslySetInnerHTML={{ __html: userBio }} />

// Options CORRECTES :
// 1. Rendre comme du texte
<div>{userBio}</div>

// 2. Rendre du markdown analysé via une bibliothèque qui nettoie
<ReactMarkdown>{userBio}</ReactMarkdown>

// 3. Si du HTML brut est nécessaire, le nettoyer d'abord avec DOMPurify
import DOMPurify from "isomorphic-dompurify";
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userBio) }} />
```

Liste de vérification à auditer pour chaque appel à `dangerouslySetInnerHTML` :

- L'entrée est-elle toujours sous notre contrôle ? Documenter la source.
- Si dérivée de l'utilisateur : est-elle nettoyée au **même endroit d'appel** ? (Le nettoyage à la frontière de l'API n'est acceptable que si chaque consommateur est vérifié.)
- La configuration du nettoyeur utilise-t-elle une liste blanche de balises, et non une liste noire ?

## Schémas d'URL non sûrs

Les URL `javascript:` et `data:` dans `href`, `src`, et `xlink:href` exécutent du code arbitraire.

```tsx
// CRITIQUE : injection d'URL javascript:
<a href={user.website}>Visit</a>   // si user.website = "javascript:alert(1)"

// CORRECT : valider le schéma
function safeUrl(url: string): string | undefined {
  try {
    const parsed = new URL(url);
    if (["http:", "https:", "mailto:"].includes(parsed.protocol)) return url;
  } catch {
    return undefined;
  }
  return undefined;
}
<a href={safeUrl(user.website)}>Visit</a>
```

React avertit à propos des URL `javascript:` dans `href` en mode développement, mais ne les bloque pas à l'exécution. Les URL `data:` et autres schémas passent également au travers. Toujours valider.

## `target="_blank"` sans `rel`

`<a target="_blank">` sans `rel="noopener noreferrer"` permet à la page cible d'accéder à `window.opener` et d'exécuter des détournements de navigation.

```tsx
// INCORRECT
<a href={externalUrl} target="_blank">External</a>

// CORRECT
<a href={externalUrl} target="_blank" rel="noopener noreferrer">External</a>
```

Les navigateurs modernes appliquent `noopener` par défaut lorsque `target="_blank"`, mais ne pas se fier aux comportements par défaut du navigateur — être explicite.

## Validation des entrées des Server Actions

Les Server Actions (`"use server"`) s'exécutent avec le même niveau de confiance qu'un endpoint d'API public. Valider chaque entrée.

```tsx
"use server";
import { z } from "zod";

const Input = z.object({
  email: z.string().email(),
  age: z.number().int().min(0).max(120),
});

export async function updateUser(_state: unknown, formData: FormData) {
  const parsed = Input.safeParse({
    email: formData.get("email"),
    age: Number(formData.get("age")),
  });
  if (!parsed.success) return { error: parsed.error.flatten() };
  // ...
}
```

- Authentifier à l'intérieur de l'action — ne pas faire confiance au verrou de route côté client
- Autoriser : confirmer que l'utilisateur courant a la permission pour l'enregistrement spécifique qu'il modifie
- Limiter le débit des actions sensibles

## Exposition de secrets via les variables d'environnement

Les variables d'environnement préfixées sont incluses dans le bundle client. Les traiter comme publiques.

| Framework | Préfixe public | Privé |
|---|---|---|
| Next.js | `NEXT_PUBLIC_*` | Toutes les autres |
| Vite | `VITE_*` | `.env` côté serveur uniquement |
| Create React App | `REACT_APP_*`, plus `NODE_ENV` et `PUBLIC_URL` | Toutes les autres (tout ce qui n'a pas le préfixe `REACT_APP_` est réservé au serveur) |
| Remix | Accès à `process.env` uniquement dans `loader`/`action` | Identique |

```ts
// CRITIQUE : secret fuité dans le bundle client
const apiKey = process.env.NEXT_PUBLIC_STRIPE_SECRET_KEY;
```

Auditer chaque PR qui touche aux variables d'environnement : cette chaîne dans le bundle public poserait-elle problème ?

## Authentification / Autorisation

- Ne jamais stocker les sessions dans `localStorage` — accessible à n'importe quel XSS. Utiliser des cookies sécurisés httpOnly.
- Ne jamais faire confiance à un état défini côté client pour verrouiller une UI sensible. Le verrouillage du rendu en JSX empêche l'affichage, pas l'accès — l'API doit l'imposer.
- CSRF : l'authentification par cookie nécessite des jetons CSRF ou des cookies `SameSite=Strict`/`Lax`
- Utiliser des cookies à double soumission ou une vérification d'origine pour les actions de formulaire lorsque les comportements par défaut du framework ne sont pas utilisés

## Content Security Policy (CSP)

À configurer côté serveur. La CSP minimale acceptable pour une application React :

```
default-src 'self';
script-src 'self' 'nonce-{REQUEST_NONCE}';
style-src 'self' 'unsafe-inline';
img-src 'self' data: https:;
connect-src 'self' https://api.example.com;
frame-ancestors 'none';
```

- Éviter `unsafe-inline` et `unsafe-eval` dans `script-src`
- Pour le SSR avec des scripts en ligne (streaming Next.js, données d'hydratation), utiliser des nonces par requête — Next.js et Remix prennent tous deux en charge l'injection de nonce
- `style-src 'unsafe-inline'` est souvent inévitable pour les bibliothèques CSS-in-JS — documenter ce compromis

## Pollution de prototype via l'étalement d'objet (object spread)

```tsx
// INCORRECT : JSON non fiable étalé directement dans l'état
const update = await req.json();
setState({ ...state, ...update });    // l'attaquant contrôle __proto__

// CORRECT : analyser avec un schéma, ou protéger les clés
const Allowed = z.object({ name: z.string(), email: z.string().email() });
const parsed = Allowed.parse(await req.json());
setState({ ...state, ...parsed });
```

## Injection de template SSR

Lors de l'utilisation de `renderToString` ou `renderToPipeableStream` :

- Toutes les valeurs rendues à l'intérieur du JSX sont échappées par React — sûr
- Les valeurs passées à `dangerouslySetInnerHTML` ne sont PAS échappées — mêmes règles que côté client
- Les enveloppes HTML construites manuellement autour de la sortie React doivent être échappées ou nettoyées — ne jamais concaténer une entrée utilisateur dans le template HTML environnant

## Composants tiers

- Auditer avec `npm audit` avant d'ajouter toute bibliothèque d'UI
- Vérifier que la bibliothèque n'utilise pas en interne `dangerouslySetInnerHTML` sur son entrée (ex. éditeurs de texte enrichi)
- Épingler les versions, examiner les changelogs avant les montées de version majeures
- Se méfier des composants qui acceptent des chaînes HTML en props

## Exposition des source maps en production

Les builds de production doivent être livrés sans source maps, ou avec des sourcemaps téléversées vers un traqueur d'erreurs (Sentry) puis retirées du bundle public. Des source maps publiques divulguent la logique interne et la structure des fichiers.

## Support par agent

- Utiliser l'agent `security-reviewer` pour des audits de sécurité complets sur l'ensemble de la base de code
- Utiliser l'agent `react-reviewer` pour les patterns spécifiques à React et les règles ci-dessus lors de la revue de code active
