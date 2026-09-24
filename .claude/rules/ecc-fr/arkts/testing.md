---
paths:
  - "**/*.ets"
  - "**/*.ts"
  - "**/ohosTest/**"
---
# Tests HarmonyOS / ArkTS

> Ce fichier étend [common/testing.md](../common/testing.md) avec des pratiques de test spécifiques à HarmonyOS.

## Framework de test

HarmonyOS utilise le framework de test intégré avec les capacités `@ohos.test` :

- **Tests unitaires** : situés dans `src/ohosTest/ets/test/`
- **Tests d'UI** : utilise `@ohos.UiTest` pour tester les composants
- **Tests instrumentés** : s'exécutent sur appareil/émulateur

## Structure du répertoire de tests

```
module/
  |-- src/
  |   |-- main/ets/          # Code de production
  |   |-- ohosTest/ets/      # Code de test
  |       |-- test/
  |       |   |-- Ability.test.ets
  |       |   |-- List.test.ets
  |       |-- TestAbility.ets
  |       |-- TestRunner.ets
```

## Exécution des tests

```bash
# Exécuter tous les tests d'un module
hvigorw testHap -p product=default

# Exécuter les tests sur un appareil connecté
hdc shell aa test -b com.example.app -m entry_test -s unittest /ets/TestRunner/OpenHarmonyTestRunner
```

## Exemple de test unitaire

```typescript
import { describe, it, expect } from '@ohos/hypium';

export default function UserViewModelTest() {
  describe('UserViewModel', () => {
    it('should_initialize_with_empty_state', 0, () => {
      const vm = new UserViewModel();
      expect(vm.userName).assertEqual('');
      expect(vm.isLoading).assertFalse();
    });

    it('should_update_user_name', 0, () => {
      const vm = new UserViewModel();
      vm.updateUserName('Alice');
      expect(vm.userName).assertEqual('Alice');
    });

    it('should_handle_empty_input', 0, () => {
      const vm = new UserViewModel();
      vm.updateUserName('');
      expect(vm.userName).assertEqual('');
      expect(vm.hasError).assertFalse();
    });
  });
}
```

## Exemple de test d'UI

```typescript
import { describe, it, expect } from '@ohos/hypium';
import { Driver, ON } from '@ohos.UiTest';

export default function HomePageUITest() {
  describe('HomePage_UI', () => {
    it('should_display_title', 0, async () => {
      const driver = Driver.create();
      await driver.delayMs(1000);

      const title = await driver.findComponent(ON.text('Home'));
      expect(title !== null).assertTrue();
    });

    it('should_navigate_to_detail_on_click', 0, async () => {
      const driver = Driver.create();
      const button = await driver.findComponent(ON.id('detailButton'));
      await button.click();
      await driver.delayMs(500);

      const detailTitle = await driver.findComponent(ON.text('Detail'));
      expect(detailTitle !== null).assertTrue();
    });
  });
}
```

## Workflow TDD pour HarmonyOS

Suis le cycle TDD standard adapté à HarmonyOS :

1. **RED** : écrire un test qui échoue dans `ohosTest/ets/test/`
2. **GREEN** : implémenter le code minimal dans `main/ets/` pour le faire passer
3. **REFACTOR** : nettoyer en gardant les tests au vert
4. **BUILD** : exécuter `hvigorw assembleHap` pour vérifier la compilation
5. **VERIFY** : exécuter les tests sur appareil/émulateur

## Exigences de couverture de test

- Minimum 80% de couverture pour tout le code applicatif critique (ViewModels, services, utilitaires)
- **Tests unitaires** : toutes les fonctions utilitaires, la logique des ViewModels, les modèles de données
- **Tests d'intégration** : appels API, opérations de base de données, interactions inter-modules
- **Tests E2E / UI** : parcours utilisateur critiques (connexion, navigation, soumission de données)
- Teste les cas limites : données vides, erreurs réseau, refus de permission

## Bonnes pratiques de test

- Garde les tests indépendants - pas d'état mutable partagé entre les tests
- Simule les appels réseau et les API système dans les tests unitaires
- Utilise des noms de test explicites : `should_[expected_behavior]_when_[condition]`
- Teste la réactivité de la gestion d'état V2 : vérifie que les propriétés `@Trace` déclenchent les mises à jour d'UI
- Teste les flux de Navigation : vérifie les opérations push/pop/replace de `NavPathStack`
- Évite de tester les mécanismes internes du framework - concentre-toi sur la logique métier et le comportement visible par l'utilisateur
