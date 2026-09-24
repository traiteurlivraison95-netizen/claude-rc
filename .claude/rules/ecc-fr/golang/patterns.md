---
paths:
  - "**/*.go"
  - "**/go.mod"
  - "**/go.sum"
---
# Patterns Go

> Ce fichier étend [common/patterns.md](../common/patterns.md) avec du contenu spécifique à Go.

## Options fonctionnelles

```go
type Option func(*Server)

func WithPort(port int) Option {
    return func(s *Server) { s.port = port }
}

func NewServer(opts ...Option) *Server {
    s := &Server{port: 8080}
    for _, opt := range opts {
        opt(s)
    }
    return s
}
```

## Interfaces petites

Définir les interfaces là où elles sont utilisées, pas là où elles sont implémentées.

## Injection de dépendances

Utiliser des fonctions constructeur pour injecter les dépendances :

```go
func NewUserService(repo UserRepository, logger Logger) *UserService {
    return &UserService{repo: repo, logger: logger}
}
```

## Référence

Voir le skill : `golang-patterns` pour des patterns Go complets incluant la concurrence, la gestion des erreurs et l'organisation des paquets.
