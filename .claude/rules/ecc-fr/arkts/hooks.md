---
paths:
  - "**/*.ets"
  - "**/*.ts"
  - "**/module.json5"
  - "**/oh-package.json5"
---
# Hooks HarmonyOS / ArkTS

> Ce fichier étend [common/hooks.md](../common/hooks.md) avec des hooks de build et de validation spécifiques à HarmonyOS.

## Commandes de build

### Build du paquet HAP

```bash
# Construire le paquet HAP (environnement hvigor global)
hvigorw assembleHap -p product=default

# Construire avec un module spécifique
hvigorw assembleHap -p module=entry -p product=default

# Build propre
hvigorw clean
```

### CLI DevEco Studio

```bash
# Vérifier la structure du projet
hvigorw --version

# Installer les dépendances
ohpm install

# Mettre à jour les dépendances
ohpm update
```

## Hooks PostToolUse recommandés

### Après édition des fichiers .ets/.ts

Exécute le build hvigor pour détecter les erreurs de compilation ArkTS :

```json
{
  "type": "PostToolUse",
  "matcher": {
    "tool": ["Edit", "Write"],
    "filePath": ["**/*.ets", "**/*.ts"]
  },
  "hooks": [
    {
      "command": "hvigorw assembleHap -p product=default 2>&1 | tail -20",
      "async": true,
      "timeout": 60000
    }
  ]
}
```

### Après édition de module.json5

Valide les déclarations de permissions et d'abilities :

```json
{
  "type": "PostToolUse",
  "matcher": {
    "tool": "Edit",
    "filePath": "**/module.json5"
  },
  "hooks": [
    {
      "command": "echo '[HarmonyOS] module.json5 modified - verify permissions and abilities'",
      "async": false
    }
  ]
}
```

### Après édition de oh-package.json5

Réinstalle les dépendances :

```json
{
  "type": "PostToolUse",
  "matcher": {
    "tool": "Edit",
    "filePath": "**/oh-package.json5"
  },
  "hooks": [
    {
      "command": "ohpm install 2>&1 | tail -10",
      "async": true,
      "timeout": 30000
    }
  ]
}
```

## Hooks PreToolUse

### Garde-fou des décorateurs V1

Avertit quand le code contient des décorateurs de gestion d'état V1 :

```json
{
  "type": "PreToolUse",
  "matcher": {
    "tool": ["Write", "Edit"],
    "filePath": "**/*.ets"
  },
  "hooks": [
    {
      "command": "echo '[HarmonyOS] Reminder: Use @ComponentV2 / @Local / @Param - V1 decorators (@State, @Prop, @Link) are prohibited'"
    }
  ]
}
```

## Checklist de validation

Après chaque cycle d'implémentation, vérifie :

- [ ] `hvigorw assembleHap` se termine sans erreur
- [ ] Aucun décorateur V1 dans les fichiers `.ets` nouveaux ou modifiés
- [ ] Aucun import `@ohos.router` dans les fichiers nouveaux ou modifiés
- [ ] Toutes les permissions d'API déclarées dans `module.json5`
- [ ] Toutes les dépendances listées dans `oh-package.json5`
- [ ] Chaînes de ressources ajoutées dans tous les répertoires i18n
- [ ] Couleurs de thème sombre fournies pour les nouvelles ressources de couleur
