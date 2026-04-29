# RED-GREEN-REFACTOR — конкретные примеры

> Реалистичные walk-through циклы для разных типов задач.

---

## Пример 1 — Новая бизнес-формула (УСН-Доходы)

### RED

```typescript
// usn-income.test.ts
import { describe, test, expect } from 'vitest';
import { calculateUsnIncome } from './usn-income';

describe('calculateUsnIncome', () => {
  test('canon §26.1: квартал доход 1М × 6% → 60k', () => {
    const result = calculateUsnIncome({
      period: { kind: 'quarter', taxYear: 2026, periodStart: '2026-01-01', periodEnd: '2026-03-31' },
      sourceData: {
        recognizedIncomeKopecks: 100_000_000,
        recognizedExpensesKopecks: 0,
        prepaymentsKopecks: 0,
        ndsOutputKopecks: 0,
        ndsInputKopecks: 0,
        region: null,
      },
      taxProfile: { taxRegime: 'usn_income', subjectType: 'organization', /* ... */ } as any,
      taxRates: [{
        id: 1, rateKind: 'usn_income', taxYear: 2026, region: null,
        rateValue: '6.0', validFrom: '2026-01-01', validTo: null, isActive: true,
        regime: null, thresholdAmount: null, stepIndex: null, notes: null, createdAt: new Date(),
      }],
    });
    
    expect(result.obligations).toHaveLength(1);
    expect(result.obligations[0].kind).toBe('usn_income_advance');
    expect(result.obligations[0].amountKopecks).toBe(6_000_000);
    expect(result.obligations[0].appliedRate).toBe('6%');
  });
});
```

Run: `pnpm test usn-income`. Result: **FAIL** — функция `calculateUsnIncome` не существует. ✓ RED.

### GREEN

```typescript
// usn-income.ts (минимум кода)
import type { CalculationInput, CalculationResult, CalculatedObligation } from './types';

export function calculateUsnIncome(input: CalculationInput): CalculationResult {
  const rateRow = input.taxRates.find(r => r.rateKind === 'usn_income');
  if (!rateRow) {
    return { obligations: [], warnings: [], errors: ['Rate not found'] };
  }
  
  const rate = Number(rateRow.rateValue);
  const taxBase = input.sourceData.recognizedIncomeKopecks;
  const grossTax = Math.round(taxBase * rate / 100);
  
  return {
    obligations: [{
      kind: 'usn_income_advance',
      amountKopecks: grossTax,
      taxBaseKopecks: taxBase,
      appliedRate: `${rate}%`,
      rateRegistryId: rateRow.id,
      dueDate: '2026-04-28',
      calculationMethod: 'usn_income_quarter',
      calculationDetails: { rate, taxBase, grossTax },
      description: 'Авансовый платёж УСН Доходы',
      initialStatus: 'calculated',
    }],
    warnings: [],
    errors: [],
  };
}
```

Run: `pnpm test usn-income`. Result: **PASS**. ✓ GREEN.

### REFACTOR

Code stub минимален. Refactor:

1. Вынести rate-resolution в отдельный helper (потому что в `usn-income-minus.ts` будет та же логика):

```typescript
// rates.ts
export function pickRate(taxRates: TaxRate[], kind: string, year: number, region: string | null) {
  return taxRates.find(r => 
    r.rateKind === kind 
    && r.taxYear === year
    && r.isActive
    && (r.region === region || r.region === null)
  );
}
```

2. Вынести due-date computation:

```typescript
// periods/due-date-calculator.ts
export function computeUsnDueDate(period, subjectType): string {
  // ...
}
```

3. Импортировать в `usn-income.ts`:

```typescript
import { pickRate } from './rates';
import { computeUsnDueDate } from '../periods/due-date-calculator';

export function calculateUsnIncome(input: CalculationInput): CalculationResult {
  const rateRow = pickRate(input.taxRates, 'usn_income', input.period.taxYear, input.sourceData.region);
  // ...
  const dueDate = computeUsnDueDate(input.period, input.taxProfile.subjectType);
  // ...
}
```

Re-run test: **PASS** still. ✓ REFACTOR successful.

### Continue cycle

Теперь добавь следующий test (next behavior):

```typescript
test('canon §26.1 with regional override: 5% вместо 6%', () => {
  const result = calculateUsnIncome({
    // ...
    taxRates: [
      { rateKind: 'usn_income', region: null, rateValue: '6.0', /* ... */ },
      { rateKind: 'usn_income', region: '77', rateValue: '5.0', /* ... */ },  // Москва override
    ],
    sourceData: { ..., region: '77' },
  });
  expect(result.obligations[0].appliedRate).toBe('5%');
  expect(result.obligations[0].amountKopecks).toBe(5_000_000);  // 1M × 5% = 50k
});
```

Run. Если pass — наш `pickRate` уже умеет regional override, продолжаем. Если fail — добавить логику в pickRate или в calculator.

---

## Пример 2 — Bug-fix (RED-first для production-bug)

### Bug report

> Production: при approve tax_payment без `financeAccountId` — operation создаётся со статусом 'posted' но ОДДС не пересчитывается.

### RED — воспроизведение

```typescript
test('regression: post tax_payment without financeAccountId должен ошибиться, не silent succeed', async () => {
  await expect(postTaxPayment({
    paymentId: 1,
    userId: 'u1',
    // missing: financeAccountId
  })).rejects.toThrow(/finance account/i);
});
```

Run: **FAIL** — bug в том что functor не проверяет, и operation создаётся silently.

### GREEN — fix

```typescript
// post-payment.ts
export async function postTaxPayment(input: PostInput): Promise<PostResult> {
  if (!input.financeAccountId) {
    throw new BusinessError('MISSING_FINANCE_ACCOUNT', 'Cannot post payment без financeAccountId');
  }
  // ... existing logic
}
```

Run test: **PASS**. ✓

### REFACTOR

Проверь — есть ли подобные missing checks в смежных функциях? `confirmPayment`, `cancelPayment`? Audit, добавить tests + checks.

### Commit

```
fix(taxes): require financeAccountId at post-payment

Production bug: payment posted без счёта списания → 
operation created с status='posted' но ОДДС не отразил отток.

Added check at postTaxPayment entry. Test reproduces bug
and verifies fix.

Refs: incident #{X}
```

---

## Пример 3 — Lifecycle test

### RED

```typescript
describe('Tax obligation lifecycle', () => {
  test('approved → cancelled allowed (с reason)', () => {
    const result = canTransition('approved', 'cancelled', { reason: 'Period reopened' });
    expect(result.allowed).toBe(true);
  });
  
  test('approved → cancelled blocked если reason missing', () => {
    const result = canTransition('approved', 'cancelled', {});
    expect(result.allowed).toBe(false);
    expect(result.reason).toMatch(/reason required/);
  });
  
  test('paid → draft никогда не разрешено', () => {
    const result = canTransition('paid', 'draft', { reason: 'whatever' });
    expect(result.allowed).toBe(false);
  });
});
```

### GREEN

```typescript
const TRANSITIONS: Record<Status, Status[]> = {
  draft: ['calculated', 'cancelled'],
  calculated: ['approved', 'cancelled'],
  approved: ['partially_paid', 'paid', 'cancelled'],
  partially_paid: ['paid', 'cancelled'],
  paid: ['cancelled'],  // только cancellation
  cancelled: [],
  // ...
};

const REQUIRES_REASON = new Set(['cancelled']);

export function canTransition(from: Status, to: Status, opts: { reason?: string } = {}) {
  if (!TRANSITIONS[from].includes(to)) {
    return { allowed: false, reason: `${from} → ${to} not allowed` };
  }
  if (REQUIRES_REASON.has(to) && !opts.reason) {
    return { allowed: false, reason: 'reason required for cancellation' };
  }
  return { allowed: true };
}
```

All 3 tests **PASS**. ✓

### REFACTOR

Можно вынести `REQUIRES_REASON` в data-driven config (если grow'ёт). Сейчас минимум — оставляем.

---

## Что сэкономлено

В каждом примере **без TDD** было бы:
- Пример 1: написал бы calculator целиком (~80 строк), потом manual-протестил один-два случая, нашёл бы баг через 2 недели в production
- Пример 2: написал бы fix без test, через 3 месяца кто-то рефакторит и regress'ит тот же bug
- Пример 3: реализовал бы state machine, потестил happy-path, забыл про edge case (reason required), нашёл бы в QA через 2 недели

С TDD — каждое поведение явно зафиксировано, regression-protected, ошибка ловится сразу.
