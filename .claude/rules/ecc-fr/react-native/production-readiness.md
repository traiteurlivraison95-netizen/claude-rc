---
paths:
  - "**/*.ts"
  - "**/*.tsx"
---
# Préparation à la production React Native / Expo

> Étend la philosophie ECC aux préoccupations de niveau production que les règles de style/pattern ne peuvent pas encoder à elles seules.
> Une base de code propre est nécessaire mais pas suffisante pour la production — ces éléments sont obligatoires avant la mise en production.

## Architecture

- Livrer sur la **New Architecture** (Fabric + TurboModules). C'est la valeur par défaut dans les SDK Expo récents et c'est obligatoire (impossible à désactiver) à partir du SDK 55+. Auditer la compatibilité des dépendances natives.
- Épingler la version du SDK Expo ; mettre à niveau délibérément avec `npx expo install --check` et tester sur les deux plateformes.

## Build et mise en production (EAS)

- Utiliser **EAS Build** pour les binaires de production et **EAS Submit** pour la livraison aux stores. Ne pas se reposer sur des builds locaux ad hoc pour la mise en production.
- Garder des profils de build séparés (`development`, `preview`, `production`) dans `eas.json`.
- Gérer les identifiants de signature via EAS ; ne jamais commiter de keystores ou de provisioning profiles.

## Mises à jour Over-the-Air

- Utiliser **EAS Update** (`expo-updates`) pour les correctifs JS uniquement, avec une politique de version de runtime définie.
- Ne jamais pousser de changements natifs via OTA — ceux-ci nécessitent un nouveau build de store.
- Déployer progressivement et conserver la capacité de revenir en arrière.

## Observabilité

- Intégrer le rapport de crash + d'erreurs (ex. **Sentry** via `@sentry/react-native`) dans les builds de production.
- Ajouter une journalisation structurée et, si utile, de l'analytics — mais retirer les logs verbeux de la mise en production.
- Capturer et exposer les états d'échec réseau/mutation ; ne jamais échouer silencieusement.

## Configuration et gestion des versions

- Incrémenter `version` et `ios.buildNumber` / `android.versionCode` à chaque mise en production.
- Configuration publique via `EXPO_PUBLIC_*` ; les vrais secrets uniquement via EAS secrets.
- Valider la configuration requise au démarrage et échouer rapidement avec un message clair.

## Porte de pré-mise en production

Avant la mise en production, tout doit passer :

- [ ] `tsc --noEmit` sans erreur
- [ ] `npx expo lint` sans erreur
- [ ] Tests au vert, couverture >= 80 % (voir testing.md)
- [ ] `npx expo-doctor` sain
- [ ] E2E des flux critiques (Maestro/Detox) réussis sur un build réel
- [ ] Aucun secret dans le bundle (voir security.md)
- [ ] Rapport de crash actif et vérifié
- [ ] Testé sur des appareils iOS et Android physiques, pas seulement des simulateurs
