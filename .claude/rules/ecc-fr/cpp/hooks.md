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
# Hooks C++

> Ce fichier étend [common/hooks.md](../common/hooks.md) avec du contenu spécifique à C++.

## Hooks de build

Exécute ces vérifications avant de commiter des changements C++ :

```bash
# Vérification du formatage
clang-format --dry-run --Werror src/*.cpp src/*.hpp

# Analyse statique
clang-tidy src/*.cpp -- -std=c++17

# Build
cmake --build build

# Tests
ctest --test-dir build --output-on-failure
```

## Pipeline CI recommandé

1. **clang-format** — vérification du formatage
2. **clang-tidy** — analyse statique
3. **cppcheck** — analyse supplémentaire
4. **cmake build** — compilation
5. **ctest** — exécution des tests avec sanitizers
