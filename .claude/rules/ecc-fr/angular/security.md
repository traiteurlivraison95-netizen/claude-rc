---
paths:
  - "**/*.component.ts"
  - "**/*.component.html"
  - "**/*.service.ts"
  - "**/*.interceptor.ts"
---
# Sécurité Angular

> Ce fichier étend [common/security.md](../common/security.md) avec du contenu spécifique à Angular.

## Prévention du XSS

Angular sanitise automatiquement les valeurs liées. Ne contourne jamais le sanitizer sur une entrée contrôlée par l'utilisateur.

```typescript
// INCORRECT : contourne la sanitisation — risque XSS
this.safeHtml = this.sanitizer.bypassSecurityTrustHtml(userInput);

// CORRECT : sanitise explicitement avant de faire confiance
this.safeHtml = this.sanitizer.sanitize(SecurityContext.HTML, userInput);
```

- N'utilise jamais les méthodes `bypassSecurityTrust*` sans raison documentée et revue
- Évite `[innerHTML]` avec du contenu non fiable — utilise `innerText` ou un pipe de sanitisation
- Ne lie jamais `[href]` à une entrée utilisateur — Angular ne bloque pas les URLs `javascript:` dans tous les contextes
- Ne construis jamais de chaînes de template à partir de données utilisateur

## Sécurité HTTP

Utilise exclusivement `HttpClient` — jamais `fetch()` brut ou `XHR` sauf si aucune alternative n'existe.

```typescript
// INCORRECT : contourne les intercepteurs (en-têtes d'auth, gestion d'erreurs, journalisation)
const res = await fetch('/api/users');

// CORRECT
users$ = this.http.get<User[]>('/api/users');
```

- Attache les jetons d'authentification via des intercepteurs — ne les fige jamais en dur dans des appels de service individuels
- Type et valide les réponses d'API — traite les données externes comme `unknown` à la frontière
- Ne journalise jamais de réponses HTTP pouvant contenir des jetons, des PII ou des identifiants

## Gestion des secrets

```typescript
// INCORRECT : secret figé en dur dans le code source
const apiKey = 'sk-live-xxxx';

// CORRECT : injecté via l'environnement
import { environment } from '../environments/environment';
const apiKey = environment.apiKey;
```

- Traite `environment.ts` comme une forme de configuration — ne stocke jamais de vrais secrets dans des fichiers d'environnement versionnés
- Injecte les secrets de production via le CI/CD (variables d'environnement, gestionnaires de secrets)

## Guards de route

Chaque route authentifiée ou restreinte par rôle doit avoir un guard. Ne te repose jamais uniquement sur le masquage d'éléments d'UI.

```typescript
{
  path: 'admin',
  canMatch: [authGuard, roleGuard('admin')],
  loadChildren: () => import('./admin/admin.routes'),
}
```

Utilise `canMatch` pour les routes sensibles — cela empêche le module de route de se charger pour de bon pour les utilisateurs non autorisés.

## Sécurité SSR

Lors de l'utilisation du SSR Angular :

- N'expose jamais de variables d'environnement côté serveur au client via `TransferState` sauf si elles sont intentionnellement publiques
- Sanitise toutes les entrées avant le rendu côté serveur — le XSS basé sur le DOM peut aussi se produire côté serveur
- Évite `window`, `document`, `localStorage` sur le serveur — protège avec `isPlatformBrowser` ou injecte via le token `DOCUMENT`

## Content Security Policy

Configure les en-têtes CSP côté serveur. Évite `unsafe-inline` dans `script-src`. Lors de l'utilisation du SSR avec des scripts inline, utilise des nonces via le support CSP d'Angular.

## Support agent

- Utilise le skill **security-reviewer** pour des audits de sécurité complets
