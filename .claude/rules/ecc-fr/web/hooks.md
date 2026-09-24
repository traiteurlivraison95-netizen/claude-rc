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
> Ce fichier étend [common/hooks.md](../common/hooks.md) avec des recommandations de hooks spécifiques au web.

# Hooks Web

## Hooks PostToolUse recommandés

Préférer l'outillage local au projet. Ne pas câbler les hooks vers une exécution ponctuelle de package distant.

### Formatage à la sauvegarde

Utiliser le point d'entrée du formateur existant du projet après modification :

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "command": "pnpm prettier --write \"$FILE_PATH\"",
        "description": "Formater les fichiers frontend modifiés"
      }
    ]
  }
}
```

Les commandes locales équivalentes via `yarn prettier` ou `npm exec prettier --` conviennent tant qu'elles utilisent des dépendances possédées par le dépôt.

### Vérification de lint

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "command": "pnpm eslint --fix \"$FILE_PATH\"",
        "description": "Exécuter ESLint sur les fichiers frontend modifiés"
      }
    ]
  }
}
```

### Vérification de types

Utiliser `--incremental` pour que les ré-exécutions réutilisent le `.tsbuildinfo` précédent (1-3s sur du code inchangé au lieu de 30-60s à chaque fois). Envelopper dans `timeout` pour qu'un tsc bloqué soit récupéré par l'OS au lieu de s'accumuler entre les modifications — cela empêche l'accumulation multi-processus qui se produit quand les modifications arrivent plus vite que tsc ne termine.

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "command": "timeout 60 pnpm tsc --noEmit --pretty false --incremental --tsBuildInfoFile node_modules/.cache/tsc-hook.tsbuildinfo",
        "description": "Vérifier les types après les modifications frontend (incrémental + plafonné par timeout)"
      }
    ]
  }
}
```

**Pourquoi les deux options comptent :**
- Sans `--incremental`, chaque modification re-vérifie tout le programme depuis zéro. Sur un vrai projet Next.js, cela s'accumule vite : modifications toutes les 5-10s + exécutions tsc de 30-60s = N processus tsc concurrents.
- Sans `timeout`, un tsc qui se bloque (changement de dépendance transitive, vérificateur de types coincé sur un type récursif) ne se termine jamais et devient orphelin quand le shell parent se termine.
- `--tsBuildInfoFile` est requis car `--noEmit` supprime normalement l'écriture du buildinfo ; spécifier le chemin explicitement garde l'incrémental fonctionnel.

Si vous êtes sur Windows sans coreutils GNU, remplacer `timeout 60` par un wrapper PowerShell ou vous reposer sur un hook Stop/SessionEnd pour balayer les processus tsc obsolètes.

### Lint CSS

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "command": "pnpm stylelint --fix \"$FILE_PATH\"",
        "description": "Linter les feuilles de style modifiées"
      }
    ]
  }
}
```

## Hooks PreToolUse

### Garde de taille de fichier

Bloquer les écritures surdimensionnées depuis le contenu d'entrée de l'outil, pas depuis un fichier qui peut ne pas encore exister :

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write",
        "command": "node -e \"let d='';process.stdin.on('data',c=>d+=c);process.stdin.on('end',()=>{const i=JSON.parse(d);const c=i.tool_input?.content||'';const lines=c.split('\\n').length;if(lines>800){console.error('[Hook] BLOCKED: File exceeds 800 lines ('+lines+' lines)');console.error('[Hook] Split into smaller modules');process.exit(2)}console.log(d)})\"",
        "description": "Bloquer les écritures dépassant 800 lignes"
      }
    ]
  }
}
```

## Hooks Stop

### Vérification finale du build

```json
{
  "hooks": {
    "Stop": [
      {
        "command": "pnpm build",
        "description": "Vérifier le build de production en fin de session"
      }
    ]
  }
}
```

## Ordonnancement

Ordre recommandé :
1. formatage
2. lint
3. vérification de types
4. vérification du build
