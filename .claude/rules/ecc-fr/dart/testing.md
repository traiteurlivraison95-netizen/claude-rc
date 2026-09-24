---
paths:
  - "**/*.dart"
  - "**/pubspec.yaml"
  - "**/analysis_options.yaml"
---
# Tests Dart/Flutter

> Ce fichier étend [common/testing.md](../common/testing.md) avec du contenu spécifique à Dart et Flutter.

## Framework de test

- **flutter_test** / **dart:test** — lanceur de test intégré
- **mockito** (avec `@GenerateMocks`) ou **mocktail** (sans génération de code) pour le mocking
- **bloc_test** pour les tests unitaires BLoC/Cubit
- **fake_async** pour contrôler le temps dans les tests unitaires
- **integration_test** pour les tests de bout en bout sur appareil

## Types de tests

| Type | Outil | Emplacement | Quand écrire |
|------|------|----------|---------------|
| Unitaire | `dart:test` | `test/unit/` | Toute la logique du domaine, les gestionnaires d'état, les repositories |
| Widget | `flutter_test` | `test/widget/` | Tous les widgets ayant un comportement significatif |
| Golden | `flutter_test` | `test/golden/` | Composants d'UI critiques pour le design |
| Intégration | `integration_test` | `integration_test/` | Parcours utilisateur critiques sur appareil réel/émulateur |

## Tests unitaires : gestionnaires d'état

### BLoC avec `bloc_test`

```dart
group('CartBloc', () {
  late CartBloc bloc;
  late MockCartRepository repository;

  setUp(() {
    repository = MockCartRepository();
    bloc = CartBloc(repository);
  });

  tearDown(() => bloc.close());

  blocTest<CartBloc, CartState>(
    'emits updated items when CartItemAdded',
    build: () => bloc,
    act: (b) => b.add(CartItemAdded(testItem)),
    expect: () => [CartState(items: [testItem])],
  );

  blocTest<CartBloc, CartState>(
    'emits empty cart when CartCleared',
    seed: () => CartState(items: [testItem]),
    build: () => bloc,
    act: (b) => b.add(CartCleared()),
    expect: () => [const CartState()],
  );
});
```

### Riverpod avec `ProviderContainer`

```dart
test('usersProvider loads users from repository', () async {
  final container = ProviderContainer(
    overrides: [userRepositoryProvider.overrideWithValue(FakeUserRepository())],
  );
  addTearDown(container.dispose);

  final result = await container.read(usersProvider.future);
  expect(result, isNotEmpty);
});
```

## Tests de widget

```dart
testWidgets('CartPage shows item count badge', (tester) async {
  await tester.pumpWidget(
    ProviderScope(
      overrides: [
        cartNotifierProvider.overrideWith(() => FakeCartNotifier([testItem])),
      ],
      child: const MaterialApp(home: CartPage()),
    ),
  );

  await tester.pump();
  expect(find.text('1'), findsOneWidget);
  expect(find.byType(CartItemTile), findsOneWidget);
});

testWidgets('shows empty state when cart is empty', (tester) async {
  await tester.pumpWidget(
    ProviderScope(
      overrides: [cartNotifierProvider.overrideWith(() => FakeCartNotifier([]))],
      child: const MaterialApp(home: CartPage()),
    ),
  );

  await tester.pump();
  expect(find.text('Your cart is empty'), findsOneWidget);
});
```

## Fakes plutôt que mocks

Privilégie les fakes écrits à la main pour les dépendances complexes :

```dart
class FakeUserRepository implements UserRepository {
  final _users = <String, User>{};
  Object? fetchError;

  @override
  Future<User?> getById(String id) async {
    if (fetchError != null) throw fetchError!;
    return _users[id];
  }

  @override
  Future<List<User>> getAll() async {
    if (fetchError != null) throw fetchError!;
    return _users.values.toList();
  }

  @override
  Stream<List<User>> watchAll() => Stream.value(_users.values.toList());

  @override
  Future<void> save(User user) async {
    _users[user.id] = user;
  }

  @override
  Future<void> delete(String id) async {
    _users.remove(id);
  }

  void addUser(User user) => _users[user.id] = user;
}
```

## Tests asynchrones

```dart
// Utiliser fake_async pour contrôler les timers et les Futures
test('debounce triggers after 300ms', () {
  fakeAsync((async) {
    final debouncer = Debouncer(delay: const Duration(milliseconds: 300));
    var callCount = 0;
    debouncer.run(() => callCount++);
    expect(callCount, 0);
    async.elapse(const Duration(milliseconds: 200));
    expect(callCount, 0);
    async.elapse(const Duration(milliseconds: 200));
    expect(callCount, 1);
  });
});
```

## Tests golden

```dart
testWidgets('UserCard golden test', (tester) async {
  await tester.pumpWidget(
    MaterialApp(home: UserCard(user: testUser)),
  );

  await expectLater(
    find.byType(UserCard),
    matchesGoldenFile('goldens/user_card.png'),
  );
});
```

Exécute `flutter test --update-goldens` lorsque des changements visuels intentionnels sont apportés.

## Nommage des tests

Utilise des noms descriptifs, centrés sur le comportement :

```dart
test('returns null when user does not exist', () { ... });
test('throws NotFoundException when id is empty string', () { ... });
testWidgets('disables submit button while form is invalid', (tester) async { ... });
```

## Organisation des tests

```
test/
├── unit/
│   ├── domain/
│   │   └── usecases/
│   └── data/
│       └── repositories/
├── widget/
│   └── presentation/
│       └── pages/
└── golden/
    └── widgets/

integration_test/
└── flows/
    ├── login_flow_test.dart
    └── checkout_flow_test.dart
```

## Couverture

- Vise 80%+ de couverture de lignes pour la logique métier (domaine + gestionnaires d'état)
- Toutes les transitions d'état doivent avoir des tests : loading → success, loading → error, retry
- Exécute `flutter test --coverage` et inspecte `lcov.info` avec un outil de rapport de couverture
- Les échecs de couverture doivent bloquer la CI en dessous du seuil
