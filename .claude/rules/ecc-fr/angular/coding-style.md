---
paths:
  - "**/*.component.ts"
  - "**/*.component.html"
  - "**/*.service.ts"
  - "**/*.directive.ts"
  - "**/*.pipe.ts"
  - "**/*.guard.ts"
  - "**/*.resolver.ts"
  - "**/*.module.ts"
---
# Style de code Angular

> Ce fichier étend [common/coding-style.md](../common/coding-style.md) avec du contenu spécifique à Angular.

## Connaissance de la version

Vérifie toujours la version Angular du projet avant d'écrire du code — les fonctionnalités diffèrent significativement d'une version à l'autre. Exécute `ng version` ou inspecte `package.json`. Lors de la création d'un nouveau projet, ne fige pas de version sauf si l'utilisateur en spécifie une.

Après avoir généré ou modifié du code Angular, exécute toujours `ng build` pour détecter les erreurs avant de terminer.

## Nommage des fichiers

Suis les conventions du CLI Angular — un artefact par fichier :

- `user-profile.component.ts` + `user-profile.component.html` + `user-profile.component.spec.ts`
- `user.service.ts`, `auth.guard.ts`, `date-format.pipe.ts`
- Dossiers de fonctionnalités : `features/users/`, `features/auth/`
- Génère avec le CLI : `ng generate component features/users/user-card`

## Composants

Privilégie les composants standalone (par défaut depuis v17+). Utilise la détection de changements `OnPush` sur tous les nouveaux composants.

```typescript
@Component({
  selector: 'app-user-card',
  standalone: true,
  imports: [RouterModule],
  templateUrl: './user-card.component.html',
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class UserCardComponent {
  user = input.required<User>();
  select = output<string>();
}
```

## Injection de dépendances

Utilise `inject()` plutôt que l'injection par constructeur. Garde les constructeurs vides ou supprime-les complètement.

```typescript
// CORRECT
@Injectable({ providedIn: 'root' })
export class UserService {
  private http = inject(HttpClient);
  private router = inject(Router);
}

// INCORRECT : l'injection par constructeur est verbeuse et moins facile à tree-shaker
constructor(private http: HttpClient, private router: Router) {}
```

Utilise `InjectionToken` pour les dépendances non-classes :

```typescript
const API_URL = new InjectionToken<string>('API_URL');

// Fournir :
{ provide: API_URL, useValue: 'https://api.example.com' }

// Consommer :
private apiUrl = inject(API_URL);
```

## Signals

### Primitives fondamentales

```typescript
count = signal(0);
doubled = computed(() => this.count() * 2);

increment() {
  this.count.update(n => n + 1);
}
```

### `linkedSignal` — état dérivé accessible en écriture

Utilise `linkedSignal` lorsqu'un signal doit se réinitialiser ou s'adapter quand une source change, tout en restant modifiable indépendamment :

```typescript
selectedOption = linkedSignal(() => this.options()[0]);
// Se réinitialise à la première option quand options change, mais l'utilisateur peut la remplacer
```

### `resource` — données asynchrones vers des signals

Utilise `resource()` pour récupérer des données asynchrones de façon réactive sans abonnements manuels :

```typescript
userResource = resource({
  request: () => ({ id: this.userId() }),
  loader: ({ request }) => fetch(`/api/users/${request.id}`).then(r => r.json()),
});

// Accès : userResource.value(), userResource.isLoading(), userResource.error()
```

### Utilisation d'`effect`

Utilise `effect()` uniquement pour les effets de bord qui doivent réagir aux changements de signal (journalisation, manipulation DOM tierce). N'utilise jamais les effects pour synchroniser des signals — utilise `computed` ou `linkedSignal` à la place. Pour le travail DOM après le rendu, utilise `afterRenderEffect`.

```typescript
// CORRECT : effet de bord
effect(() => console.log('User changed:', this.user()));

// INCORRECT : utiliser computed à la place
effect(() => { this.fullName.set(`${this.first()} ${this.last()}`); });
```

## Templates

Utilise la syntaxe de blocs v17+. Fournis toujours `track` dans `@for` :

```html
@for (item of items(); track item.id) {
  <app-item [item]="item" />
}

@if (isLoading()) {
  <app-spinner />
} @else if (error()) {
  <app-error [message]="error()" />
} @else {
  <app-content [data]="data()" />
}
```

Aucune logique dans les templates au-delà de conditions simples — déplace-la vers les méthodes du composant ou vers des pipes.

## Formulaires

Choisis la stratégie de formulaire qui correspond à l'approche déjà en place dans le projet :

- **Signal Forms** (v21+) : à privilégier pour les nouveaux projets en v21+. État de formulaire basé sur les signals.
- **Reactive Forms** : `FormBuilder` + `FormGroup` + `FormControl`. Idéal pour les formulaires complexes avec validation dynamique.
- **Template-Driven Forms** : `ngModel`. Adapté uniquement aux formulaires simples.

```typescript
// Reactive Forms — approche standard pour la plupart des applications
export class LoginComponent {
  private fb = inject(FormBuilder);

  form = this.fb.group({
    email: ['', [Validators.required, Validators.email]],
    password: ['', [Validators.required, Validators.minLength(8)]],
  });

  submit() {
    if (this.form.valid) {
      // utiliser this.form.value
    }
  }
}
```

## Styles de composants

Utilise des styles au niveau du composant avec `ViewEncapsulation.Emulated` (par défaut). Évite `ViewEncapsulation.None` sauf lors de la construction d'un design system qui doit intentionnellement laisser fuir ses styles.

- Cantonne les styles au composant — n'utilise pas de noms de classes globaux dans les feuilles de style des composants
- Utilise `:host` pour styler l'élément hôte
- Privilégie les propriétés CSS personnalisées pour les valeurs thémables

## Détection de changements

- Utilise `ChangeDetectionStrategy.OnPush` par défaut sur tous les nouveaux composants
- Les signals et le pipe `async` gèrent la détection automatiquement — évite `markForCheck()` et `detectChanges()`
- Ne mute jamais les objets `@Input()` en place quand OnPush est utilisé
