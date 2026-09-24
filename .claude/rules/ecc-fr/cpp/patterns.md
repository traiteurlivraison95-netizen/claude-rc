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
# Patterns C++

> Ce fichier étend [common/patterns.md](../common/patterns.md) avec du contenu spécifique à C++.

## RAII (Resource Acquisition Is Initialization)

Lie la durée de vie d'une ressource à celle d'un objet :

```cpp
class FileHandle {
public:
    explicit FileHandle(const std::string& path) : file_(std::fopen(path.c_str(), "r")) {}
    ~FileHandle() { if (file_) std::fclose(file_); }
    FileHandle(const FileHandle&) = delete;
    FileHandle& operator=(const FileHandle&) = delete;
private:
    std::FILE* file_;
};
```

## Règle des cinq/zéro

- **Règle de zéro** : privilégie les classes n'ayant besoin d'aucun destructeur, constructeur/affectation de copie ou de déplacement personnalisé
- **Règle des cinq** : si tu définis l'un des éléments destructeur/constructeur de copie/affectation de copie/constructeur de déplacement/affectation de déplacement, définis les cinq

## Sémantique de valeur

- Passe les types petits/triviaux par valeur
- Passe les types volumineux par `const&`
- Retourne par valeur (repose-toi sur RVO/NRVO)
- Utilise la sémantique de déplacement pour les paramètres « puits »

## Gestion des erreurs

- Utilise les exceptions pour les conditions exceptionnelles
- Utilise `std::optional` pour les valeurs pouvant ne pas exister
- Utilise `std::expected` (C++23) ou des types de résultat pour les échecs attendus

## Référence

Voir le skill : `cpp-coding-standards` pour des patterns et anti-patterns C++ complets.
