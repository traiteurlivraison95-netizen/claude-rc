# Règles

> Version française de `.claude/rules/ecc/`. Coexiste avec la version anglaise d'origine ; ne la remplace pas.

## Structure

Les règles sont organisées en une couche **commune** (« common ») plus des répertoires **spécifiques à chaque langage** :

```
rules/
├── common/          # Principes indépendants du langage (à toujours installer)
│   ├── coding-style.md
│   ├── git-workflow.md
│   ├── testing.md
│   ├── performance.md
│   ├── patterns.md
│   ├── hooks.md
│   ├── agents.md
│   └── security.md
├── typescript/      # Spécifique à TypeScript/JavaScript
├── angular/         # Spécifique à Angular
├── vue/             # Spécifique à Vue 3
├── nuxt/            # Spécifique à Nuxt 4
├── python/          # Spécifique à Python
├── golang/          # Spécifique à Go
├── web/             # Spécifique au Web et au frontend
├── react-native/    # Spécifique à React Native / Expo
├── swift/           # Spécifique à Swift
├── php/             # Spécifique à PHP
├── ruby/            # Spécifique à Ruby / Rails
└── arkts/           # Spécifique à HarmonyOS / ArkTS
```

- **common/** contient les principes universels — pas d'exemples de code spécifiques à un langage.
- Les **répertoires de langage** étendent les règles communes avec des patterns, outils et exemples de code propres au framework. Chaque fichier référence son équivalent commun.

## Installation

### Option 1 : Script d'installation (recommandé)

```bash
# Installer common + un ou plusieurs jeux de règles spécifiques à un langage
./install.sh typescript
./install.sh angular
./install.sh vue
./install.sh nuxt
./install.sh python
./install.sh golang
./install.sh web
./install.sh react-native
./install.sh swift
./install.sh php
./install.sh ruby
./install.sh arkts

# Installer plusieurs langages à la fois
./install.sh typescript python
```

### Option 2 : Installation manuelle

> **Important :** copiez des répertoires entiers — ne les aplatissez PAS avec `/*`.
> Les répertoires common et spécifiques à un langage contiennent des fichiers
> portant les mêmes noms. Les aplatir dans un seul répertoire fait que les fichiers
> spécifiques à un langage écrasent les règles communes, et casse les références
> relatives `../common/` utilisées par les fichiers spécifiques à un langage.
>
> Utilisez l'espace de noms propre à l'ECC ci-dessous pour les installations Claude
> au niveau utilisateur. Les destinations à plat, au niveau du paquet, peuvent entrer
> en collision avec des jeux de règles non-ECC et ne correspondent pas aux
> recommandations du README principal.

```bash
# Créer l'espace de noms de règles ECC une seule fois.
mkdir -p ~/.claude/rules/ecc

# Installer les règles communes (requises pour tous les projets)
cp -r rules/common ~/.claude/rules/ecc/

# Installer les règles spécifiques au langage selon la stack de votre projet
cp -r rules/typescript ~/.claude/rules/ecc/
cp -r rules/angular ~/.claude/rules/ecc/
cp -r rules/vue ~/.claude/rules/ecc/
cp -r rules/nuxt ~/.claude/rules/ecc/
cp -r rules/python ~/.claude/rules/ecc/
cp -r rules/golang ~/.claude/rules/ecc/
cp -r rules/web ~/.claude/rules/ecc/
cp -r rules/react-native ~/.claude/rules/ecc/
cp -r rules/swift ~/.claude/rules/ecc/
cp -r rules/php ~/.claude/rules/ecc/
cp -r rules/ruby ~/.claude/rules/ecc/
cp -r rules/arkts ~/.claude/rules/ecc/

# Attention ! ! ! Adaptez selon les besoins réels de votre projet ; la configuration ci-dessus n'est donnée qu'à titre indicatif.
```

Pour des règles locales au projet, utilisez le même espace de noms à la racine du projet :

```bash
mkdir -p .claude/rules/ecc
cp -r rules/common .claude/rules/ecc/
cp -r rules/typescript .claude/rules/ecc/
```

## Règles vs Skills

- Les **règles** (« rules ») définissent des standards, conventions et checklists qui s'appliquent largement (par ex. « 80 % de couverture de tests », « pas de secrets en dur »).
- Les **skills** (répertoire `skills/`) fournissent une documentation de référence approfondie et actionnable pour des tâches spécifiques (par ex. `python-patterns`, `golang-testing`).

Les fichiers de règles spécifiques à un langage référencent les skills pertinentes le cas échéant. Les règles disent _quoi_ faire ; les skills disent _comment_ le faire.

## Ajouter un nouveau langage

Pour ajouter le support d'un nouveau langage (par ex. `rust/`) :

1. Créer un répertoire `rules/rust/`
2. Ajouter des fichiers qui étendent les règles communes :
   - `coding-style.md` — outils de formatage, idiomes, patterns de gestion d'erreurs
   - `testing.md` — framework de test, outils de couverture, organisation des tests
   - `patterns.md` — patterns de conception spécifiques au langage
   - `hooks.md` — hooks PostToolUse pour formateurs, linters, vérificateurs de types
   - `security.md` — gestion des secrets, outils d'analyse de sécurité
3. Chaque fichier doit commencer par :
   ```
   > This file extends [common/xxx.md](../common/xxx.md) with <Language> specific content.
   ```
4. Référencer les skills existantes si disponibles, ou en créer de nouvelles sous `skills/`.

Pour les domaines non liés à un langage comme `web/`, suivez le même schéma en couches dès lors qu'il existe suffisamment de conseils réutilisables et spécifiques au domaine pour justifier un jeu de règles autonome.

## Priorité des règles

Quand les règles spécifiques à un langage et les règles communes entrent en conflit, **les règles spécifiques au langage priment** (le spécifique l'emporte sur le général). Cela suit le schéma standard de configuration en couches (similaire à la spécificité CSS ou à la précédence de `.gitignore`).

- `rules/common/` définit les valeurs par défaut universelles applicables à tous les projets.
- `rules/golang/`, `rules/python/`, `rules/swift/`, `rules/php/`, `rules/typescript/`, `rules/react-native/`, etc. remplacent ces valeurs par défaut là où les idiomes du langage diffèrent.

### Exemple

`common/coding-style.md` recommande l'immutabilité comme principe par défaut. Un fichier spécifique au langage `golang/coding-style.md` peut le remplacer :

> Le Go idiomatique utilise des receveurs par pointeur pour la mutation de struct — voir [common/coding-style.md](../common/coding-style.md) pour le principe général, mais la mutation idiomatique en Go est préférée ici.

### Règles communes avec notes de dérogation

Les règles de `rules/common/` qui peuvent être remplacées par des fichiers spécifiques à un langage sont marquées par :

> **Note de langage** : Cette règle peut être remplacée par des règles spécifiques au langage pour les langages où ce pattern n'est pas idiomatique.
