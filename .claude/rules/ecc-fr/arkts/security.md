---
paths:
  - "**/*.ets"
  - "**/*.ts"
  - "**/module.json5"
---
# Sécurité HarmonyOS / ArkTS

> Ce fichier étend [common/security.md](../common/security.md) avec des pratiques de sécurité spécifiques à HarmonyOS.

## Gestion des permissions

### Déclarer les permissions dans module.json5

Tous les appels d'API système nécessitant des permissions doivent être déclarés :

```json5
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.INTERNET",
        "reason": "$string:internet_permission_reason",
        "usedScene": {
          "abilities": ["EntryAbility"],
          "when": "always"
        }
      }
    ]
  }
}
```

### Checklist des permissions

Avant d'appeler des API système, vérifie :

- [ ] Permission déclarée dans `module.json5`
- [ ] Chaîne de justification de la permission définie dans les ressources (pour les permissions visibles par l'utilisateur)
- [ ] Demande de permission à l'exécution implémentée pour les permissions sensibles (caméra, localisation, etc.)
- [ ] Vérification de la permission avant l'appel d'API avec repli gracieux en cas de refus

### Demande de permission à l'exécution

```typescript
import { abilityAccessCtrl, bundleManager, Permissions } from '@kit.AbilityKit';

async function checkAndRequestPermission(permission: Permissions): Promise<boolean> {
  const atManager = abilityAccessCtrl.createAtManager();
  const bundleInfo = await bundleManager.getBundleInfoForSelf(
    bundleManager.BundleFlag.GET_BUNDLE_INFO_WITH_APPLICATION
  );
  const tokenId = bundleInfo.appInfo.accessTokenId;
  const grantStatus = await atManager.checkAccessToken(tokenId, permission);

  if (grantStatus === abilityAccessCtrl.GrantStatus.PERMISSION_GRANTED) {
    return true;
  }

  const result = await atManager.requestPermissionsFromUser(getContext(), [permission]);
  return result.authResults[0] === abilityAccessCtrl.GrantStatus.PERMISSION_GRANTED;
}
```

## Gestion des secrets

- **JAMAIS** figer en dur des clés d'API, jetons ou mots de passe dans les fichiers source `.ets`/`.ts`
- Utilise l'API HarmonyOS Preferences pour la configuration non sensible
- Utilise le HarmonyOS Keystore pour les identifiants sensibles
- Les configurations spécifiques à l'environnement doivent être gérées via les profils de build

```typescript
// MAUVAIS : secret figé en dur
const API_KEY: string = 'sk-xxxxxxxxxxxx';

// BON : depuis la configuration du profil de build (non sensible)
import { BuildProfile } from 'BuildProfile';
const endpoint = BuildProfile.API_ENDPOINT;

// BON : utiliser HUKS pour chiffrer/déchiffrer des données sans exposer le matériel de clé
import { huks } from '@kit.UniversalKeystoreKit';
async function decryptWithKeystore(alias: string, nonce: Uint8Array, aad: Uint8Array, cipherData: Uint8Array): Promise<Uint8Array> {
  const options: huks.HuksOptions = {
    properties: [
      { tag: huks.HuksTag.HUKS_TAG_ALGORITHM, value: huks.HuksKeyAlg.HUKS_ALG_AES },
      { tag: huks.HuksTag.HUKS_TAG_PURPOSE, value: huks.HuksKeyPurpose.HUKS_KEY_PURPOSE_DECRYPT },
      { tag: huks.HuksTag.HUKS_TAG_BLOCK_MODE, value: huks.HuksCipherMode.HUKS_MODE_GCM },
      { tag: huks.HuksTag.HUKS_TAG_PADDING, value: huks.HuksKeyPadding.HUKS_PADDING_NONE },
      { tag: huks.HuksTag.HUKS_TAG_NONCE, value: nonce },
      { tag: huks.HuksTag.HUKS_TAG_ASSOCIATED_DATA, value: aad }
    ],
    inData: cipherData
  };
  const handle = await huks.initSession(alias, options);
  const result = await huks.finishSession(handle.handle, options);
  return result.outData;
}
```

## Validation des entrées

- Valide toutes les entrées utilisateur avant traitement
- Sanitise les données avant affichage dans l'UI pour prévenir les injections
- Valide les paramètres de deep link avant la navigation

```typescript
// Valider avant navigation
function handleDeepLink(uri: string): void {
  const allowedPaths: string[] = ['detail', 'settings', 'profile'];
  const parsed = new URL(uri);
  const path = parsed.pathname.replace('/', '');

  if (!allowedPaths.includes(path)) {
    hilog.warn(0x0000, 'DeepLink', 'Invalid deep link path: %{public}s', path);
    return;
  }

  navPathStack.pushPath({ name: path });
}
```

## Sécurité réseau

- Utilise toujours HTTPS pour les requêtes réseau
- Valide les certificats serveur
- Implémente des politiques de timeout et de réessai pour les requêtes
- Ne journalise jamais de données sensibles (jetons, identifiants utilisateur) dans les logs de requête/réponse réseau

## Sécurité du stockage de données

- Utilise des préférences chiffrées pour les données locales sensibles
- Efface les données sensibles de la mémoire lorsqu'elles ne sont plus nécessaires
- Implémente une gestion appropriée du cycle de vie des données
- Prends en compte la classification des données (public, interne, confidentiel) lors du choix du mécanisme de stockage

## Sécurité des dépendances

- Utilise uniquement des dépendances provenant de sources fiables (registre ohpm officiel)
- Vérifie les versions des dépendances dans `oh-package.json5`
- Vérifie régulièrement les vulnérabilités connues dans les bibliothèques tierces
- Fige les versions de dépendances pour éviter les mises à jour inattendues
