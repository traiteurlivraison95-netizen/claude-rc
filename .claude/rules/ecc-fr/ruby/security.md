---
paths:
  - "**/*.rb"
  - "**/*.rake"
  - "**/Gemfile"
  - "**/Gemfile.lock"
  - "**/config/routes.rb"
  - "**/config/credentials*.yml.enc"
---
# Sécurité Ruby

> Ce fichier étend [common/security.md](../common/security.md) avec du contenu spécifique à Ruby et Rails.

## Valeurs par défaut Rails

- Garder la protection CSRF activée pour les requêtes navigateur qui changent d'état.
- Utiliser les strong parameters ou des objets frontière typés avant tout mass assignment.
- Stocker les secrets dans les credentials Rails, des variables d'environnement, ou un gestionnaire de secrets. Ne jamais commiter de clés en clair, de tokens, de credentials privés, ou de valeurs `.env` copiées.

## SQL et Active Record

- Préférer les API de requête Active Record et le SQL paramétré.
- Ne jamais interpoler des valeurs de requête, cookie, en-tête, job, ou webhook dans des chaînes SQL.
- Cadrer soigneusement les callbacks de modèle ; les effets de bord sensibles pour la sécurité doivent être explicites et couverts par des tests.

## Authentification et sessions

- Utiliser le générateur d'authentification Rails 8 pour une authentification par session simple, ou Devise quand OAuth, MFA, confirmable, lockable, l'authentification multi-modèle, ou des conventions Devise existantes sont requises.
- Faire tourner les sessions après la connexion et les changements de privilèges.
- Protéger les flux de récupération de compte avec expiration, tokens à usage unique, limitation de débit, et journalisation d'audit.

## Dépendances

- Exécuter des vérifications de dépendances quand le lockfile change :

```bash
bundle exec bundle-audit check --update
bundle exec brakeman --no-progress
```

- Examiner les nouvelles gems pour l'activité du mainteneur, le risque d'extension native, les dépendances transitives, et si le même comportement peut être implémenté avec le cœur de Rails.

## Sécurité web

- Échapper la sortie des templates par défaut. Traiter `html_safe`, `raw`, et les sanitizers personnalisés comme du code sensible pour la sécurité.
- Valider les téléversements de fichiers par type de contenu, extension, taille, et destination de stockage.
- Traiter les jobs en arrière-plan, les webhooks, les messages Action Cable, et les entrées Turbo Stream comme des frontières non fiables.

## Référence

Voir la skill : `security-review` pour des patterns de revue sécurisés par défaut.
