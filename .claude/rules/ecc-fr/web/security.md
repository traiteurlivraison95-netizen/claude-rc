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
> Ce fichier étend [common/security.md](../common/security.md) avec du contenu de sécurité spécifique au web.

# Règles de sécurité Web

## Content Security Policy

Toujours configurer une CSP de production.

### CSP basée sur nonce

Utiliser un nonce par requête pour les scripts plutôt que `'unsafe-inline'`.

```text
Content-Security-Policy:
  default-src 'self';
  script-src 'self' 'nonce-{RANDOM}' https://cdn.jsdelivr.net;
  style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
  img-src 'self' data: https:;
  font-src 'self' https://fonts.gstatic.com;
  connect-src 'self' https://*.example.com;
  frame-src 'none';
  object-src 'none';
  base-uri 'self';
```

Ajuster les origines au projet. Ne pas copier-coller ce bloc sans le modifier.

## Prévention XSS

- Ne jamais injecter de HTML non assaini
- Éviter `innerHTML` / `dangerouslySetInnerHTML` sauf assainissement préalable
- Échapper les valeurs de template dynamiques
- Assainir le HTML utilisateur avec un sanitizer local vérifié quand c'est absolument nécessaire

## Scripts tiers

- Charger de manière asynchrone
- Utiliser SRI lors du service depuis un CDN
- Auditer trimestriellement
- Préférer l'auto-hébergement pour les dépendances critiques quand c'est pratique

## HTTPS et en-têtes

```text
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
```

## Formulaires

- Protection CSRF sur les formulaires qui changent d'état
- Limitation de débit sur les endpoints de soumission
- Valider côté client et côté serveur
- Préférer les honeypots ou des contrôles anti-abus légers aux CAPTCHA par défaut trop lourds
