---
name: canon-tdd
description: Use when implementing any feature or bugfix that has testable logic — particularly business calculations, lifecycle transitions, validators, money handling, antidubli enforcement. Enforces RED-GREEN-REFACTOR discipline — write a failing test first, write minimum code to pass, refactor while keeping tests green. Defines what to test (high-leverage paths) and what NOT to test (UI snapshots, framework code, trivial getters). Cross-cuts canon-execute — invoked alongside it for testable phases.
---

# canon-tdd

🚀 Дисциплина test-driven-development в canon-dev pipeline

> Этот скилл — **дисциплина**, не процесс с фазами. Применяется внутри `canon-execute` когда фаза включает testable логику. Параллелен с реализацией, не отдельная фаза.

---

## Когда использовать

- Реализация бизнес-формул (расчёты, агрегаты, dengi)
- Реализация lifecycle state machine
- Реализация validators
- Реализация antidubli механизмов
- Реализация cross-module integration logic
- Bug fixes — ВСЕГДА начинать с failing test который воспроизводит баг

## Когда НЕ использовать

- Pure UI styling (тестируется визуально через design-canon HTML preview)
- Framework boilerplate (Next.js routes setup, generic CRUD без бизнес-логики)
- Глупые getters/setters
- Logging
- Ad-hoc throwaway scripts (одноразовый migration, etc.)

---

## Философия

**RED-GREEN-REFACTOR** — три фазы каждого изменения:

```
1. RED:      Напиши test который описывает желаемое поведение.
             Test должен FAIL — потому что код для него ещё не написан.
             Если test проходит без кода — test слабый, переписать.

2. GREEN:    Напиши МИНИМУМ кода чтобы test passed.
             Не optimize, не generalize, не «делать красиво».
             Просто: test green.

3. REFACTOR: Test green — теперь рефактори код к чистому состоянию.
             Удали duplication, переименуй, выдели helpers.
             Test продолжает быть green после каждой правки.
```

Этот цикл — **минут**, не часов. Один test → код → refactor — за 5-15 минут. Если фаза занимает дольше — test слишком большой, разбей.

### Почему это работает

1. **Test-first защищает от over-engineering.** Без test'а соблазн написать general-purpose решение которое не нужно. Test ограничивает scope.

2. **Test-first защищает от bugs.** Каждое поведение проверено в момент написания, не «потом протестируем».

3. **Refactor под зелёным test'ом — безопасен.** Можно агрессивно менять структуру, test ловит регрессии.

4. **Coverage возникает естественно.** Без отдельной «тестовой фазы» — каждая строка production-кода написана под существующий test.

---

## Что тестировать (high-leverage)

### Money calculations

**Каждая** формула которая считает деньги — должна иметь test с конкретными входами и ожидаемыми числами:

```typescript
// Из канона §21.2: УСН-Доходы 1М × 6% = 60k
test('УСН-Доходы quarterly: 1М доход → 60k к уплате', () => {
  const result = calculateUsnIncome({
    period: { kind: 'quarter', taxYear: 2026 },
    sourceData: { recognizedIncomeKopecks: 100_000_000, prepaymentsKopecks: 0 },
    taxRates: [{ rateKind: 'usn_income', rateValue: 6, region: null, taxYear: 2026, isActive: true }],
  });
  expect(result.obligations[0].amountKopecks).toBe(6_000_000);
});
```

Канон обычно даёт «учебные примеры» — каждый из них = test case.

### Lifecycle transitions

Каждое разрешённое и запрещённое состояние — test:

```typescript
test('Lifecycle: draft → calculated разрешено', () => {
  expect(canTransition('draft', 'calculated')).toBe(true);
});

test('Lifecycle: draft → posted напрямую запрещено', () => {
  expect(canTransition('draft', 'posted')).toBe(false);
});
```

### Antidubli механизмы

Idempotency — тест на двойной вызов:

```typescript
test('Создание payment с тем же idempotencyKey возвращает existing', async () => {
  const key = 'tax-payment-123-pay';
  const first = await createTaxPayment({ ..., idempotencyKey: key });
  const second = await createTaxPayment({ ..., idempotencyKey: key });
  expect(second.id).toBe(first.id);
});
```

### Edge cases формул

Каждый явный edge case — separate test:

```typescript
test('УСН-Доходы-Расходы годовой: убыток → minimum-tax', () => {
  // Доход 10M, расходы 12M → база = 0
  // Min-tax = 10M × 1% = 100k
  // Должно использовать min-tax вместо обычного расчёта (canon §21.3)
  const result = calculateUsnIncomeMinus({
    period: { kind: 'year', ... },
    sourceData: { recognizedIncomeKopecks: 1_000_000_000, recognizedExpensesKopecks: 1_200_000_000 },
    ...
  });
  expect(result.obligations[0].amountKopecks).toBe(10_000_000);  // 100k в копейках
  expect(result.obligations[0].calculationMethod).toBe('usn_income_minus_min_tax');
});
```

### Validators

Каждое правило валидации — test для valid и invalid:

```typescript
test('Tax profile: НПД с сотрудниками — invalid', () => {
  expect(() => validateTaxProfileCombination({
    taxRegime: 'npd',
    hasEmployees: true,
    ...
  })).toThrow(/НПД запрещает наличие сотрудников/);
});
```

---

## Чего НЕ тестировать

### UI снапшоты

```typescript
// ❌ Плохо — фиксирует HTML, ломается при любом стилевом изменении
expect(component.toMatchSnapshot()).toBe(true);
```

UI правильность проверяется визуально (через design-canon HTML preview + manual T-сценарии в canon-execute), не snapshot-тестами.

### Framework boilerplate

```typescript
// ❌ Плохо — тестирует Next.js, не нашу логику
test('GET /api/foo возвращает 200', () => {...});
```

Если endpoint просто возвращает `db.select()` — тестирование добавляет шум, не ловит bugs. Тестируй логику внутри handler'а отдельно.

### Mock-everything tests

```typescript
// ❌ Плохо — mocks так много, что test не отражает реальность
const mockDb = mock(...);
const mockAuth = mock(...);
const mockLogger = mock(...);
// Test проходит. Real code не работает. Test бесполезен.
```

Если требуется mock'ать > 3 dependencies — это сигнал что функция делает слишком много. Refactor и тестируй pure logic отдельно.

### Trivial getters/setters

```typescript
// ❌ Плохо — тестирует что property setter работает
test('user.name setter', () => {
  user.name = 'Alice';
  expect(user.name).toBe('Alice');
});
```

Java-style verbose tests. TypeScript типы это уже проверяют.

---

## Bug-fix workflow (RED-FIRST)

Когда находишь баг в production — **никогда** не пиши fix первым.

```
1. Воспроизведи баг локально через test (RED — test fails because of the bug)
2. Fix code → test становится GREEN
3. Refactor если нужно
4. Commit с reference на bug
```

**Почему это критично:**
- Без test'а — нет гарантии что fix реально fixит. Может «как будто работает» но edge case остался.
- Test становится regression-protection: если баг повторится через 6 месяцев — test поймает.
- Forces understanding: чтобы написать test который воспроизводит баг — нужно понять root cause.

См. `canon-debug` для systematic-debugging workflow которое предшествует write-test-first для bug-fix'ов.

---

## Test placement

### Co-location

Tests рядом с кодом который тестируют:

```
src/lib/taxes/calculation/
├── usn-income.ts
├── usn-income.test.ts          ← здесь
├── usn-income-minus.ts
├── usn-income-minus.test.ts    ← здесь
└── ...
```

Не в отдельной `tests/`-tree вдалеке. Co-location делает связь test ↔ code очевидной.

### Naming

```
{module}.test.ts        для unit tests
{module}.integration.test.ts   для tests требующих DB / external
```

### Structure

Один test = один behavior. Не «test всех vodorodov ratio in one go».

```typescript
describe('calculateUsnIncome', () => {
  describe('quarter period', () => {
    test('1M доход → 60k tax_due', () => { ... });
    test('0 доход → 0 tax_due, no error', () => { ... });
    test('region override → региональная ставка', () => { ... });
  });
  
  describe('year period', () => {
    test('применяет prepayments', () => { ... });
    test('prepayments=0 → warning issued', () => { ... });
  });
  
  describe('error paths', () => {
    test('rate not found in registry → error', () => { ... });
    test('non-supported period kind → empty obligations + warning', () => { ... });
  });
});
```

---

## Tools

### Vitest (recommended)

Быстрый, ESM-нативный, отличная DX:

```bash
pnpm add -D vitest
```

`vitest.config.ts`:
```typescript
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,
    environment: 'node',
  },
});
```

`package.json`:
```json
{
  "scripts": {
    "test": "vitest",
    "test:run": "vitest run",
    "test:coverage": "vitest run --coverage"
  }
}
```

### Когда добавлять test runner

Если в проекте ещё **нет** test runner — добавлять не самостоятельно. Координатор включает его как часть подходящей фазы (обычно Phase 0 — инфраструктура, или first phase где появляется testable логика).

Если test runner есть — используй существующий.

### Pre-merge integration

Добавь в `canon-execute` pre-merge gates:

```bash
pnpm test:run          # 0 failing tests
```

Если упал — fix или revert before PR.

---

## Anti-patterns

❌ **«Test'ы пишем потом, после стабилизации фичи»** — нет. Test-first или код без тестов. «Test потом» = test никогда.

❌ **«Сначала весь код, потом все test'ы»** — нет. Цикл RED-GREEN-REFACTOR — короткий (5-15 минут). Не «час кода → час тестов».

❌ **Test один большой, покрывает всё** — нет. Один test = один behavior. Если ломается — сразу понятно что именно.

❌ **Mock'ать everything** — нет. Если нужно > 3 mocks — refactor. Тестируй pure logic отдельно от dependencies.

❌ **Snapshot tests для UI** — нет. UI визуально через design-canon preview.

❌ **Coverage % как цель** — нет. Coverage — побочный эффект, не цель. 100% coverage с плохими тестами хуже чем 60% с хорошими.

❌ **Test для каждого if-else в private function** — нет. Тестируй public API. Private логика покрывается через tests на public.

---

## Critical principles

1. **RED first.** Test FAIL до того как код написан.
2. **GREEN minimum.** Минимум кода чтобы test passed.
3. **REFACTOR с green tests.** Изменяй структуру, tests ловят регрессии.
4. **Bug-fix начинается с failing test.** Без него — нет гарантии fix'а.
5. **Test high-leverage paths.** Calculations, lifecycle, validators, antidubli.
6. **Skip low-leverage tests.** UI snapshots, trivial getters, framework boilerplate.
7. **Co-location.** Tests рядом с кодом.
8. **One test = one behavior.**

---

## Templates и references

- `references/test-anti-patterns.md` — расширенный anti-pattern guide
- `references/red-green-refactor-examples.md` — конкретные примеры цикла

---

## Финальная заметка

TDD не делает разработку медленнее. Делает её **предсказуемее**: каждое изменение — green tests или явный red. Не «работает на моей машине → ломается у пользователя».

Цена дисциплины — 20-30% времени. Окупается на первом же regression-bug который не появился в production благодаря test'у написанному месяц назад.
