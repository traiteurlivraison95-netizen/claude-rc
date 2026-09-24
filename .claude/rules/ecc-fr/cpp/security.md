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
# Sécurité C++

> Ce fichier étend [common/security.md](../common/security.md) avec du contenu spécifique à C++.

## Sécurité mémoire

- N'utilise jamais `new`/`delete` bruts — utilise des smart pointers
- N'utilise jamais de tableaux de style C — utilise `std::array` ou `std::vector`
- N'utilise jamais `malloc`/`free` — utilise l'allocation C++
- Évite `reinterpret_cast` sauf absolue nécessité

## Débordements de tampon

- Utilise `std::string` plutôt que `char*`
- Utilise `.at()` pour un accès avec vérification des limites quand la sécurité importe
- N'utilise jamais `strcpy`, `strcat`, `sprintf` — utilise `std::string` ou `fmt::format`

## Comportement indéfini

- Initialise toujours les variables
- Évite les débordements d'entiers signés
- Ne déréférence jamais de pointeur null ou pendant (dangling)
- Utilise les sanitizers en CI :
  ```bash
  cmake -DCMAKE_CXX_FLAGS="-fsanitize=address,undefined" ..
  ```

## Analyse statique

- Utilise **clang-tidy** pour des vérifications automatisées :
  ```bash
  clang-tidy --checks='*' src/*.cpp
  ```
- Utilise **cppcheck** pour une analyse supplémentaire :
  ```bash
  cppcheck --enable=all src/
  ```

## Référence

Voir le skill : `cpp-coding-standards` pour des directives de sécurité détaillées.
