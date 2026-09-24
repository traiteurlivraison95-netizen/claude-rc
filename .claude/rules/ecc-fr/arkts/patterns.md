---
paths:
  - "**/*.ets"
  - "**/*.ts"
---
# Patterns HarmonyOS / ArkTS

> Ce fichier étend [common/patterns.md](../common/patterns.md) avec des patterns spécifiques à HarmonyOS et ArkTS.

## Gestion d'état : V2 uniquement

**Utilisation obligatoire** de la gestion d'état ArkUI V2. Les décorateurs V1 sont dépréciés et ne doivent plus être utilisés.

### Décorateurs V2

| Décorateur | Objet |
|-----------|---------|
| `@ComponentV2` | Marque une struct comme composant V2 |
| `@Local` | État local au sein d'un composant |
| `@Param` | Props reçues du parent (lecture seule) |
| `@Event` | Événements de rappel de l'enfant vers le parent |
| `@Provider` | Fournit l'état aux composants descendants |
| `@Consumer` | Consomme l'état d'un ancêtre `@Provider` |
| `@Monitor` | Surveille les changements d'état (remplace le `@Watch` V1) |
| `@Computed` | Valeurs dérivées/calculées |
| `@ObservedV2` | Rend une classe observable pour la gestion d'état V2 |
| `@Trace` | Marque les propriétés observables dans les classes `@ObservedV2` |

### Décorateurs V1 interdits

N'utilise jamais : `@State`, `@Prop`, `@Link`, `@ObjectLink`, `@Observed`, `@Provide`, `@Consume`, `@Watch`, `@Component` (utilise `@ComponentV2` à la place).

### Exemple de composant V2

```typescript
@ObservedV2
class UserModel {
  @Trace name: string = ''
  @Trace age: number = 0
}

@ComponentV2
struct UserCard {
  @Param user: UserModel = new UserModel()
  @Event onDelete: () => void = () => {}

  build() {
    Column() {
      Text(this.user.name)
        .fontSize($r('app.float.font_size_title'))
      Text(`${this.user.age}`)
        .fontSize($r('app.float.font_size_body'))
      Button($r('app.string.delete'))
        .onClick(() => this.onDelete())
    }
  }
}
```

### Synchronisation d'état

```typescript
@ComponentV2
struct ParentPage {
  @Provider('userState') userModel: UserModel = new UserModel()

  build() {
    Column() {
      ChildComponent()  // reçoit automatiquement @Consumer('userState')
    }
  }
}

@ComponentV2
struct ChildComponent {
  @Consumer('userState') userModel: UserModel = new UserModel()

  build() {
    Text(this.userModel.name)
  }
}
```

## Routage : Navigation uniquement

**Utilisation obligatoire** du composant `Navigation` avec `NavPathStack`. N'utilise jamais `@ohos.router`.

### Configuration de Navigation

```typescript
@ComponentV2
struct MainPage {
  @Local navPathStack: NavPathStack = new NavPathStack()

  build() {
    Navigation(this.navPathStack) {
      // Contenu de l'accueil
    }
    .navDestination(this.routerMap)
  }

  @Builder
  routerMap(name: string, param: ESObject) {
    if (name === 'detail') {
      DetailPage()
    } else if (name === 'settings') {
      SettingsPage()
    }
  }
}
```

### Navigation entre pages

```typescript
// Empiler une nouvelle page
this.navPathStack.pushPath({ name: 'detail', param: { id: '123' } })

// Remplacer la page courante
this.navPathStack.replacePath({ name: 'settings' })

// Revenir en arrière
this.navPathStack.pop()

// Revenir à la racine
this.navPathStack.clear()
```

### Sous-page NavDestination

```typescript
@ComponentV2
struct DetailPage {
  build() {
    NavDestination() {
      Column() {
        Text($r('app.string.detail_title'))
      }
    }
    .title($r('app.string.detail_nav_title'))
  }
}
```

## Pattern d'architecture : MVVM

Architecture recommandée pour les applications HarmonyOS :

```
feature/
  |-- model/           # Modèles de données (classes @ObservedV2)
  |-- viewmodel/       # Logique métier (classes ViewModel)
  |-- view/            # Composants d'UI (structs @ComponentV2)
  |-- service/         # Appels API, accès aux données
```

- **View** : uniquement de la logique de rendu, aucune logique métier dans `build()`
- **ViewModel** : toute la logique métier y est encapsulée
- **Model** : classes de données pures avec `@ObservedV2` et `@Trace`
- **Service** : requêtes réseau, opérations de base de données, E/S de fichiers

## Patterns d'animation ArkUI

### Animation pilotée par l'état

```typescript
@ComponentV2
struct AnimatedCard {
  @Local isExpanded: boolean = false
  @Local cardScale: number = 0.8

  build() {
    Column() {
      // Contenu
    }
    .scale({ x: this.cardScale, y: this.cardScale })
    .animation({ duration: 300, curve: Curve.EaseInOut })
    .onClick(() => {
      this.isExpanded = !this.isExpanded
      this.cardScale = this.isExpanded ? 1.0 : 0.8
    })
  }
}
```

### Règles d'animation

- Privilégie les API d'animation natives HarmonyOS et les templates avancés
- Utilise une UI déclarative avec des animations pilotées par l'état (change les variables d'état pour déclencher les animations)
- Définis `renderGroup(true)` pour les animations de sous-composants complexes afin de réduire les lots de rendu
- **JAMAIS** changer fréquemment `width`, `height`, `padding`, `margin` pendant les animations - impact sévère sur les performances
- Utilise `animateTo` pour un contrôle explicite de l'animation
- Privilégie `transform` (translate, scale, rotate) et `opacity` pour des animations performantes

## Patterns de performance

### LazyForEach pour les grandes listes

```typescript
@ComponentV2
struct LargeList {
  @Local dataSource: MyDataSource = new MyDataSource()

  build() {
    List() {
      LazyForEach(this.dataSource, (item: ItemModel) => {
        ListItem() {
          ItemComponent({ item: item })
        }
      }, (item: ItemModel) => item.id)
    }
  }
}
```

### Réutilisation de composants

- Extrais les composants réutilisables dans des fichiers séparés
- Utilise `@Builder` pour des fragments d'UI légers au sein d'un composant
- Utilise `@Param` pour des composants configurables

## Références de ressources

Définis toujours les constantes d'UI comme des ressources et référence-les via `$r()` :

```typescript
// MAUVAIS : valeurs figées en dur
Text('Hello')
  .fontSize(16)
  .fontColor('#333333')

// BON : références de ressources
Text($r('app.string.greeting'))
  .fontSize($r('app.float.font_size_body'))
  .fontColor($r('app.color.text_primary'))
```
