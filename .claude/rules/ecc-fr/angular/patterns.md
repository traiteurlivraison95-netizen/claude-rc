---
paths:
  - "**/*.component.ts"
  - "**/*.component.html"
  - "**/*.service.ts"
  - "**/*.store.ts"
  - "**/*.routes.ts"
---
# Patterns Angular

> Ce fichier étend [common/patterns.md](../common/patterns.md) avec du contenu spécifique à Angular.

## Séparation composants intelligents / composants bêtes

Les composants intelligents (conteneurs) gèrent la récupération des données et l'état. Les composants bêtes (de présentation) reçoivent uniquement des inputs et émettent des outputs — aucune injection de service.

```typescript
// Intelligent — possède les données
@Component({ standalone: true, changeDetection: ChangeDetectionStrategy.OnPush })
export class UserPageComponent {
  private userService = inject(UserService);
  user = toSignal(this.userService.getUser(this.userId));
}
```

```html
<!-- Bête — présentation pure -->
<app-user-card [user]="user()" (select)="onSelect($event)" />
```

## Couche de services

Les services possèdent tout l'accès aux données et toute la logique métier. Les composants délèguent — pas de `HttpClient` dans les composants.

```typescript
@Injectable({ providedIn: 'root' })
export class UserService {
  private http = inject(HttpClient);

  getUsers(): Observable<User[]> {
    return this.http.get<User[]>('/api/users');
  }
}
```

## Données asynchrones avec `resource`

Utilise `resource()` pour la récupération asynchrone réactive. À privilégier par rapport aux pipelines RxJS manuels pour le chargement de données simple :

```typescript
export class UserDetailComponent {
  userId = input.required<string>();

  userResource = resource({
    request: () => ({ id: this.userId() }),
    loader: ({ request }) =>
      firstValueFrom(inject(UserService).getUser(request.id)),
  });
}
```

Accès à l'état : `userResource.value()`, `userResource.isLoading()`, `userResource.error()`, `userResource.reload()`.

## Patterns d'état avec les Signals

```typescript
// État local mutable
count = signal(0);

// Dérivé (jamais dupliqué)
doubled = computed(() => this.count() * 2);

// État dérivé accessible en écriture, qui se réinitialise avec la source
selectedItem = linkedSignal(() => this.items()[0]);

// Pont entre Observable et signal
users = toSignal(this.userService.getUsers(), { initialValue: [] });
```

Ne stocke jamais de valeurs dérivées dans des signals séparés — utilise `computed`. N'utilise jamais `effect` pour synchroniser des signals — utilise `computed` ou `linkedSignal`.

## Nettoyage des abonnements

Utilise `takeUntilDestroyed()` pour tous les abonnements manuels. N'utilise jamais `ngOnDestroy` manuel + `Subject` + `takeUntil` dans du nouveau code.

```typescript
export class UserComponent {
  private destroyRef = inject(DestroyRef);

  ngOnInit() {
    this.userService.updates$
      .pipe(takeUntilDestroyed(this.destroyRef))
      .subscribe(update => this.handleUpdate(update));
  }
}
```

## Routage

### Définition des routes

```typescript
// app.routes.ts
export const routes: Routes = [
  { path: '', component: HomeComponent },
  {
    path: 'admin',
    canMatch: [authGuard],           // CanMatch empêche le chargement du chunk pour de bon
    loadChildren: () => import('./admin/admin.routes').then(m => m.ADMIN_ROUTES),
  },
  {
    path: 'users/:id',
    resolve: { user: userResolver },
    component: UserDetailComponent,
  },
];
```

- Utilise `canMatch` plutôt que `canActivate` lorsque le module de route ne doit pas se charger pour les utilisateurs non autorisés
- Charge tous les modules de fonctionnalités en lazy-loading avec `loadChildren`
- Pré-récupère les données avec `resolve` pour éviter les états de chargement dans les composants

### Guards fonctionnels

```typescript
export const authGuard: CanActivateFn = () => {
  const auth = inject(AuthService);
  return auth.isAuthenticated()
    ? true
    : inject(Router).createUrlTree(['/login']);
};
```

### Résolveurs de données

```typescript
export const userResolver: ResolveFn<User> = (route) => {
  return inject(UserService).getUser(route.paramMap.get('id')!);
};
```

### Transitions de vue

Active des transitions de route fluides avec la View Transitions API :

```typescript
// app.config.ts
provideRouter(routes, withViewTransitions())
```

## Patterns d'injection de dépendances

### Providers cantonnés

Fournis les services au niveau du composant ou de la route lorsqu'ils ne doivent pas être des singletons :

```typescript
@Component({
  providers: [UserEditService],   // cantonné à cette sous-arborescence de composant
})
export class UserEditComponent {}
```

### `InjectionToken`

```typescript
export const CONFIG = new InjectionToken<AppConfig>('APP_CONFIG');

// Dans providers :
{ provide: CONFIG, useValue: appConfig }
{ provide: CONFIG, useFactory: () => loadConfig(), deps: [] }

// Consommer :
private config = inject(CONFIG);
```

### `viewProviders` vs `providers`

- `providers` : disponible pour le composant et tous ses enfants de contenu
- `viewProviders` : disponible uniquement pour la propre vue du composant (pas le contenu projeté)

## Intercepteurs HTTP

Utilise les intercepteurs fonctionnels (v15+) pour l'authentification, la gestion des erreurs et les tentatives de réessai :

```typescript
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const token = inject(AuthService).token();
  if (!token) return next(req);
  return next(req.clone({ setHeaders: { Authorization: `Bearer ${token}` } }));
};
```

Enregistre dans `app.config.ts` :

```typescript
provideHttpClient(withInterceptors([authInterceptor, errorInterceptor]))
```

## Opérateurs RxJS

- `switchMap` — recherche, navigation (annule la précédente)
- `mergeMap` — requêtes parallèles indépendantes
- `exhaustMap` — soumissions de formulaire (ignore jusqu'à ce que la précédente se termine)
- Gère toujours les erreurs avec `catchError` — ne laisse jamais un flux mourir en silence

```typescript
search$ = this.query$.pipe(
  debounceTime(300),
  distinctUntilChanged(),
  switchMap(q => this.service.search(q).pipe(catchError(() => of([])))),
);
```

## Formulaires

Aligne-toi sur la stratégie de formulaire déjà en place dans le projet. Pour les nouvelles applications v21+, privilégie les signal forms.

```typescript
// Reactive Forms — standard pour les formulaires complexes
export class UserFormComponent {
  private fb = inject(FormBuilder);

  form = this.fb.group({
    name: ['', Validators.required],
    email: ['', [Validators.required, Validators.email]],
  });
}
```

## Stratégies de rendu

- **CSR** (par défaut) : SPA standard
- **SSR + Hydration** : `ng add @angular/ssr` — améliore le FCP et le SEO
- **SSG (Prerendering)** : pages statiques générées au build pour les routes riches en contenu

Lors de l'utilisation du SSR, évite `window`, `document`, `localStorage` directement — utilise `isPlatformBrowser` ou le token `DOCUMENT`.

## Accessibilité

Utilise Angular CDK pour des composants headless et accessibles (Accordion, Listbox, Combobox, Menu, Tabs, Toolbar, Tree, Grid). Style les attributs ARIA plutôt que de les gérer manuellement :

```css
[aria-selected="true"] { background: var(--color-selected); }
```

## Référence de skill

Voir le skill : `angular-developer` pour des conseils approfondis sur les signals, les formulaires, le routage, l'injection de dépendances, le SSR et les patterns d'accessibilité.
