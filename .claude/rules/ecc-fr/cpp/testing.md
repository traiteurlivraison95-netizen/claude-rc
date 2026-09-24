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
# Tests C++

> Ce fichier étend [common/testing.md](../common/testing.md) avec du contenu spécifique à C++.

## Framework

Utilise **GoogleTest** (gtest/gmock) avec **CMake/CTest**.

## Exécution des tests

```bash
cmake --build build && ctest --test-dir build --output-on-failure
```

## Couverture

```bash
cmake -DCMAKE_CXX_FLAGS="--coverage" -DCMAKE_EXE_LINKER_FLAGS="--coverage" ..
cmake --build .
ctest --output-on-failure
lcov --capture --directory . --output-file coverage.info
```

## Sanitizers

Exécute toujours les tests avec les sanitizers en CI :

```bash
cmake -DCMAKE_CXX_FLAGS="-fsanitize=address,undefined" ..
```

## Référence

Voir le skill : `cpp-testing` pour des patterns de test C++ détaillés, le workflow TDD et l'usage de GoogleTest/GMock.
