---
paths:
  - "**/*.cpp"
  - "**/*.hpp"
  - "**/*.cc"
  - "**/*.hh"
  - "**/*.cxx"
  - "**/*.h"
  - "**/CMakeLists.txt"
---
# Style de code C++

> Ce fichier étend [common/coding-style.md](../common/coding-style.md) avec du contenu spécifique à C++.

## C++ moderne (C++17/20/23)

- Privilégie les **fonctionnalités C++ modernes** aux constructions de style C
- Utilise `auto` quand le type est évident d'après le contexte
- Utilise `constexpr` pour les constantes évaluées à la compilation
- Utilise les liaisons structurées : `auto [key, value] = map_entry;`

## Gestion des ressources

- **RAII partout** — pas de `new`/`delete` manuels
- Utilise `std::unique_ptr` pour la propriété exclusive
- Utilise `std::shared_ptr` uniquement quand une propriété partagée est réellement nécessaire
- Utilise `std::make_unique` / `std::make_shared` plutôt que `new` brut

## Conventions de nommage

- Types/Classes : `PascalCase`
- Fonctions/Méthodes : `snake_case` ou `camelCase` (suis la convention du projet)
- Constantes : `kPascalCase` ou `UPPER_SNAKE_CASE`
- Namespaces : `lowercase`
- Variables membres : `snake_case_` (underscore final) ou préfixe `m_`

## Formatage

- Utilise **clang-format** — pas de débat de style
- Exécute `clang-format -i <file>` avant de commiter

## Référence

Voir le skill : `cpp-coding-standards` pour des standards et directives C++ complets.
