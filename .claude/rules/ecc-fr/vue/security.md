---
paths:
  - "**/*.vue"
---

# Sécurité Vue

> Ce fichier étend [common/security.md](../common/security.md) avec du contenu spécifique à Vue.

## Ce que Vue échappe automatiquement

- L'interpolation de texte `{{ }}` et les bindings d'attributs dynamiques (`:title`) sont auto-échappés. Les vecteurs ci-dessous ne sont PAS protégés.

## Règle n°1 : templates uniquement depuis des sources fiables

- Ne jamais utiliser du contenu non fiable comme template de composant. Pas de compilation de template à l'exécution depuis une entrée utilisateur.
- Pas de `:is` contrôlé par l'utilisateur qui résout un composant depuis une chaîne arbitraire.

## v-html et fonctions de rendu

- `v-html` contourne l'échappement et est un vecteur XSS direct. À éviter sur du contenu utilisateur.
- Si inévitable, assainir avec DOMPurify (config allowlist) avant le binding, ou rendre dans un iframe sandboxé. Vue lui-même recommande d'assainir côté backend avant la persistance.
- La sortie des fonctions de rendu et des scoped slots comporte le même risque. Passer du HTML utilisateur via `h()` avec `innerHTML` équivaut à `v-html` sous un autre nom. Assainir d'abord.

## Injection URL, style, et événement

- `:href` et `:src` ne sont pas échappés. Les URL `javascript:` s'exécutent. Valider le schéma, n'autoriser que `http` / `https` / `mailto`. Les docs Vue référencent `@braintree/sanitize-url`, mais assainir côté backend avant la persistance.
- `:style` avec une entrée utilisateur est dangereux (exfiltration CSS). Utiliser la syntaxe objet avec des propriétés en liste blanche, jamais une chaîne utilisateur brute.
- Ne jamais binder une entrée utilisateur à `onclick`, `onfocus`, ou tout attribut d'événement.

## Secrets du bundle client

- Tout ce qui se trouve dans `import.meta.env.VITE_*` est expédié au navigateur. Garder les clés API et tokens côté serveur.
- Utiliser des cookies httpOnly pour les tokens de session. Ne jamais empaqueter de credentials dans le client.

```vue
<!-- dangereux -->
<div v-html="userBio" />
<!-- sûr -->
<div v-html="sanitize(userBio)" />
```

## Référence

- Skills ECC : `frontend-patterns`, `vite-patterns`.
- Docs : <https://vuejs.org/guide/best-practices/security.html> · <https://github.com/cure53/DOMPurify> · <https://github.com/braintree/sanitize-url>
