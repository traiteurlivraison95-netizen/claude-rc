---
paths:
  - "**/*.spec.ts"
  - "**/*.test.ts"
---
# Tests Angular

> Ce fichier étend [common/testing.md](../common/testing.md) avec du contenu spécifique à Angular.

## Lanceur de tests

Utilise le lanceur de tests configuré par le projet. Vérifie `angular.json` et `package.json` ; les projets Angular utilisent couramment Vitest, Jest, ou Jasmine + Karma.

```bash
ng test               # mode watch
ng test --no-watch    # mode CI
```

## Configuration TestBed

Pour les composants standalone, importe le composant directement. Appelle `compileComponents()` pour les composants avec des templates externes.

```typescript
describe('UserCardComponent', () => {
  let fixture: ComponentFixture<UserCardComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [UserCardComponent],
    }).compileComponents();

    fixture = TestBed.createComponent(UserCardComponent);
  });
});
```

## Inputs basés sur des signals

Définis les inputs basés sur des signals via `fixture.componentRef.setInput()` :

```typescript
fixture.componentRef.setInput('user', mockUser);
fixture.detectChanges();
```

## Harnesses de composants

Privilégie les harnesses de composants Angular CDK aux requêtes DOM directes pour l'interaction avec l'UI. Les harnesses sont plus résilients aux changements de balisage.

```typescript
import { HarnessLoader } from '@angular/cdk/testing';
import { TestbedHarnessEnvironment } from '@angular/cdk/testing/testbed';
import { MatButtonHarness } from '@angular/material/button/testing';

let loader: HarnessLoader;

beforeEach(() => {
  loader = TestbedHarnessEnvironment.loader(fixture);
});

it('triggers save on button click', async () => {
  const button = await loader.getHarness(MatButtonHarness.with({ text: 'Save' }));
  await button.click();
  expect(saveSpy).toHaveBeenCalled();
});
```

## Tests de routeur

Utilise `RouterTestingHarness` pour les composants dépendant du routeur :

```typescript
import { RouterTestingHarness } from '@angular/router/testing';

it('renders user on navigation', async () => {
  const harness = await RouterTestingHarness.create();
  const component = await harness.navigateByUrl('/users/1', UserDetailComponent);
  expect(component.userId()).toBe('1');
});
```

## Tests asynchrones

Utilise `fakeAsync` + `tick` pour un contrôle précis de l'asynchrone. Utilise `waitForAsync` pour de l'asynchrone réel avec `fixture.whenStable()`.

```typescript
it('loads user after delay', fakeAsync(() => {
  const service = TestBed.inject(UserService);
  vi.spyOn(service, 'getUser').mockReturnValue(of(mockUser));

  fixture.detectChanges();
  tick();
  fixture.detectChanges();

  expect(fixture.nativeElement.querySelector('.name').textContent).toBe(mockUser.name);
}));
```

## Tests HTTP

```typescript
import { provideHttpClientTesting } from '@angular/common/http/testing';
import { HttpTestingController } from '@angular/common/http/testing';

beforeEach(() => {
  TestBed.configureTestingModule({
    providers: [provideHttpClient(), provideHttpClientTesting()],
  });
  httpMock = TestBed.inject(HttpTestingController);
});

afterEach(() => httpMock.verify());
```

## Tests de services

Injecte les services directement sans fixture de composant :

```typescript
describe('UserService', () => {
  let service: UserService;

  beforeEach(() => {
    TestBed.configureTestingModule({
      providers: [provideHttpClient(), provideHttpClientTesting()],
    });
    service = TestBed.inject(UserService);
  });
});
```

## Quoi tester

- **Services** : toutes les méthodes publiques, chemins d'erreur, interactions HTTP
- **Composants** : liaisons input/output, rendu produit pour les états clés, interactions utilisateur via les harnesses
- **Pipes** : transformation pure — tests unitaires simples, pas besoin de TestBed
- **Guards/Resolvers** : valeurs de retour pour les états autorisés et refusés en utilisant `RouterTestingHarness`

## Tests E2E

Utilise le framework E2E configuré par le projet, tel que Cypress ou Playwright, pour les parcours utilisateur critiques.

```typescript
describe('Login flow', () => {
  it('redirects to dashboard on valid credentials', () => {
    cy.visit('/login');
    cy.get('[data-cy=email]').type('user@example.com');
    cy.get('[data-cy=password]').type('password123');
    cy.get('[data-cy=submit]').click();
    cy.url().should('include', '/dashboard');
  });
});
```

- Ajoute des attributs `data-cy` aux éléments interactifs pour des sélecteurs stables
- Ne te repose pas sur les classes CSS ou le contenu textuel pour les sélecteurs dans les tests E2E

## Couverture

Vise ≥80% pour les services et les pipes. Composants : teste le comportement, pas les détails d'implémentation.

## Référence de skill

Voir le skill : `angular-developer` pour des patterns de test complets, l'usage des harnesses et les bonnes pratiques asynchrones.
