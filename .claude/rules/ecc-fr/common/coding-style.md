# Style de code

## Immutabilité (CRITIQUE)

TOUJOURS créer de nouveaux objets, NE JAMAIS muter les objets existants :

```
// Pseudocode
FAUX :    modify(original, field, value) → modifie l'original sur place
CORRECT : update(original, field, value) → renvoie une nouvelle copie avec le changement
```

Justification : les données immuables évitent les effets de bord cachés, facilitent le débogage et permettent une concurrence sûre.

## Principes fondamentaux

### KISS (Keep It Simple)

- Préférer la solution la plus simple qui fonctionne réellement
- Éviter l'optimisation prématurée
- Optimiser pour la clarté plutôt que pour l'ingéniosité

### DRY (Don't Repeat Yourself)

- Extraire la logique répétée dans des fonctions ou utilitaires partagés
- Éviter la dérive due au copier-coller
- Introduire des abstractions quand la répétition est réelle, pas spéculative

### YAGNI (You Aren't Gonna Need It)

- Ne pas construire de fonctionnalités ou d'abstractions avant qu'elles ne soient nécessaires
- Éviter la généralité spéculative
- Commencer simple, puis refactoriser quand le besoin devient réel

## Organisation des fichiers

BEAUCOUP DE PETITS FICHIERS > PEU DE GROS FICHIERS :
- Forte cohésion, faible couplage
- 200-400 lignes en général, avec 800 lignes comme plafond souple de maintenabilité pour les fichiers source
- Les fichiers de test, générés et vendorisés peuvent dépasser ce plafond lorsque leur taille est justifiée par leur rôle
- Extraire les utilitaires des gros modules
- Organiser par fonctionnalité/domaine, pas par type

## Gestion des erreurs

TOUJOURS gérer les erreurs de manière exhaustive :
- Gérer les erreurs explicitement à chaque niveau
- Fournir des messages d'erreur conviviaux dans le code orienté UI
- Journaliser le contexte détaillé de l'erreur côté serveur
- Ne jamais avaler silencieusement les erreurs

## Validation des entrées

TOUJOURS valider aux frontières du système :
- Valider toutes les entrées utilisateur avant traitement
- Utiliser une validation basée sur un schéma quand disponible
- Échouer rapidement avec des messages d'erreur clairs
- Ne jamais faire confiance aux données externes (réponses d'API, entrées utilisateur, contenu de fichier)

## Conventions de nommage

> **Note de langage** : Cette règle peut être remplacée par des règles spécifiques au langage pour
> les langages où un pattern n'est pas idiomatique. La casse et les préfixes propres à un framework
> relèvent de la règle de langage ou de package applicable.

Indépendant du langage :

- Noms descriptifs : le nom dit ce que la chose contient ou fait, sans commentaire.
- Les noms de booléens se lisent clairement comme des affirmations selon la convention du
  langage ou du package applicable.
- Là où le langage fait la distinction, les constantes et les types se distinguent visuellement
  des valeurs ordinaires selon la forme définie par sa règle de langage ou de package.

## Odeurs de code à éviter

### Imbrication profonde

Préférer les retours anticipés aux conditions imbriquées dès que la logique commence à s'empiler.

### Nombres magiques

Utiliser des constantes nommées pour les seuils, délais et limites significatifs.

### Fonctions longues

Diviser les grandes fonctions en morceaux ciblés avec des responsabilités claires.

## Checklist de qualité de code

Avant de considérer le travail terminé :
- [ ] Le code est lisible et bien nommé
- [ ] Les fonctions sont petites (<50 lignes)
- [ ] Les fichiers sont ciblés (<800 lignes)
- [ ] Pas d'imbrication profonde (>4 niveaux)
- [ ] Gestion des erreurs appropriée
- [ ] Pas de valeurs en dur (utiliser des constantes ou de la config)
- [ ] Pas de mutation (patterns immuables utilisés)
