# Test anti-patterns

> Конкретные паттерны которые **выглядят** как тесты но реально вредят. Используй как чек-лист при review.

---

## AP1 — Snapshot tests для UI

```typescript
// ❌ Плохо
test('Header renders correctly', () => {
  expect(render(<Header />).asFragment()).toMatchSnapshot();
});
```

**Почему плохо:**
- Любая стилевая правка → snapshot fails. Команда привыкает «обновлять snapshot не глядя».
- Не ловит реальные bugs (snapshot fix совпадает с бага fix?).
- Создаёт шум, делает PR-diff'ы непрочитываемыми.

**Что делать вместо:**
- UI-correctness — через design-canon HTML preview (visual review)
- Функциональность UI — через E2E (Playwright) для critical-flows
- Логика внутри компонента — через unit-test pure-функций отдельно

---

## AP2 — Mock-everything tests

```typescript
// ❌ Плохо
test('processPayment работает', async () => {
  const mockDb = mock<Database>();
  const mockAuth = mock<AuthService>();
  const mockEmail = mock<EmailService>();
  const mockLogger = mock<Logger>();
  const mockMetrics = mock<MetricsService>();
  
  mockDb.select.mockReturnValue([{ id: 1 }]);
  mockAuth.getUser.mockReturnValue({ id: 'u1' });
  // ... 50 lines of mock setup
  
  const result = await processPayment(/* ... */);
  expect(result).toBeTruthy();  // 🤷
});
```

**Почему плохо:**
- Test setup длиннее реальной production-логики
- Test проходит при mocks. Real code — может не работать.
- Mocks становятся outdated тихо (real DB schema меняется, mock остаётся прежним).
- При возрасте — все игнорируют этот test.

**Что делать вместо:**
- Refactor: извлечь pure logic из `processPayment`. Pure logic — без dependencies, тестируется без mocks.
- Integration tests с real test-DB для DB-dependent логики.
- E2E через Playwright для full-flow.

---

## AP3 — Test all the trivials

```typescript
// ❌ Плохо
test('User has firstName property', () => {
  const u = new User({ firstName: 'Alice' });
  expect(u.firstName).toBe('Alice');
});

test('User can have email', () => {
  const u = new User({ email: 'a@b.c' });
  expect(u.email).toBe('a@b.c');
});

// ... 50 such tests
```

**Почему плохо:**
- Тестирует что property setter работает (TypeScript уже это знает)
- Не ловит bugs
- Скрывает важные tests в шуме

**Что делать вместо:**
- Тестировать только behavior — что происходит когда user invokes method, when validation fails, when state transitions
- Доверять TypeScript для type-correctness

---

## AP4 — Coverage as goal

```typescript
// ❌ Плохо setup
{
  "test": "vitest --coverage --coverage.minimum 90"
}
```

**Почему плохо:**
- Команда пишет low-quality tests чтобы «закрыть coverage»
- 90% coverage с плохими tests хуже чем 60% с хорошими
- Tests добавляются для метрики, не для regression-protection

**Что делать вместо:**
- Coverage — diagnostic, не target
- Watch для **uncovered critical paths** (например, `pickRate` без тестов = подозрительно)
- Игнорируй coverage на framework-boilerplate (Next.js handlers без логики)

---

## AP5 — Multiple assertions без структуры

```typescript
// ❌ Плохо
test('payment processing works', async () => {
  const p = await createPayment(...);
  expect(p.id).toBeDefined();
  expect(p.status).toBe('draft');
  
  const p2 = await transitionPayment(p.id, 'review');
  expect(p2.status).toBe('review');
  
  const p3 = await transitionPayment(p2.id, 'approved');
  expect(p3.status).toBe('approved');
  
  // ... 30 lines, 15 assertions
});
```

**Почему плохо:**
- Если упал — не сразу понятно какое assertion
- Не «one test = one behavior»
- При первом fail — следующие assertions не проверяются

**Что делать вместо:**
- Один test = одно поведение
- Использовать `describe` для группировки связанных
- Setup в `beforeEach` если нужно общее состояние

```typescript
describe('payment lifecycle', () => {
  let payment;
  beforeEach(async () => {
    payment = await createPayment(...);
  });
  
  test('create → status draft', () => {
    expect(payment.status).toBe('draft');
  });
  
  test('draft → review allowed', async () => {
    const updated = await transitionPayment(payment.id, 'review');
    expect(updated.status).toBe('review');
  });
  
  // ...
});
```

---

## AP6 — Test внутренней реализации

```typescript
// ❌ Плохо
test('calculateTax вызывает pickRate', () => {
  const spy = vi.spyOn(rateModule, 'pickRate');
  calculateTax(...);
  expect(spy).toHaveBeenCalled();
});
```

**Почему плохо:**
- Test ломается при refactor (если pickRate inlined — bug или нет?)
- Тестирует "как" вместо "что"
- Hard-couples test к internal structure

**Что делать вместо:**
- Тестировать observable output: что возвращает функция, какие side-effects (DB-records, audit-log entries)
- Internal helpers — не fault-line, могут refactor'иться

---

## AP7 — Async без await assertions

```typescript
// ❌ Плохо
test('async error handling', () => {
  expect(async () => {
    await failingOperation();
  }).toThrow();  // ← это не работает для async!
});
```

**Почему плохо:**
- Test passes даже когда не должен (toThrow не дожидается async resolve)
- False positive — пропускает bugs

**Что делать вместо:**
```typescript
test('async error handling', async () => {
  await expect(failingOperation()).rejects.toThrow(/expected message/);
});
```

---

## AP8 — Date.now() / Math.random() в тестируемом коде без injection

```typescript
// ❌ Плохо
function generateInvoiceNumber() {
  return `INV-${Date.now()}`;
}

// Test:
test('generates invoice number', () => {
  expect(generateInvoiceNumber()).toBe('INV-???');  // что писать?
});
```

**Почему плохо:**
- Test зависит от текущего времени → flaky
- Не reproducible

**Что делать вместо:**
```typescript
function generateInvoiceNumber(now: Date = new Date()): string {
  return `INV-${now.getTime()}`;
}

test('generates invoice number', () => {
  const now = new Date('2026-01-01T00:00:00Z');
  expect(generateInvoiceNumber(now)).toBe('INV-1735689600000');
});
```

Inject dependencies (Date, random, env) — параметром или DI-style.

---

## AP9 — Test описывает что **не** должно делать (negative-only)

```typescript
// ❌ Плохо
test('does not crash', () => {
  expect(() => processPayment(...)).not.toThrow();
});
```

**Почему плохо:**
- "Не падает" — ничего не говорит о correctness
- Может вернуть garbage, test всё равно passes

**Что делать вместо:**
- Указать что **должно** произойти: специфические values, side-effects

---

## AP10 — Слепое копирование real-data в test fixtures

```typescript
// ❌ Плохо
const fixtureUser = JSON.parse(fs.readFileSync('production-dump.json'));
test('handle user', () => {
  process(fixtureUser);  // 200 полей, неизвестно что test'тируется
});
```

**Почему плохо:**
- Production data = шум
- Test становится опе fragile к любым data-changes
- PII leak risk если fixture commit'ишь

**Что делать вместо:**
- Build minimal fixtures **specifically for the test**
- Только relevant поля, остальные — реалистичные defaults

---

## Проверка test'а перед commit'ом

Чек-лист:

- [ ] One test = one behavior?
- [ ] Test name описывает behavior, не implementation?
- [ ] Assertions конкретные (specific values, не just "truthy")?
- [ ] Не mock'ает > 3 dependencies?
- [ ] Async с правильным await?
- [ ] Не зависит от Date.now() / random?
- [ ] Fail message информативный?
- [ ] Test ломается при правильном bug-инъекции (mutation testing mindset)?

Если всё ✓ — test готов.
