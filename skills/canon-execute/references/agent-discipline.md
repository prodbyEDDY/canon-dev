# Agent discipline — расширенные правила и примеры

> Конкретные паттерны и анти-паттерны для агента-реализатора. Используй как справочник во время работы.

---

## Discipline 1 — Чтение перед написанием

### Right

```
1. Получил промпт. Прочитал §0-2 (контекст + scope + brand rules).
2. Открыл CLAUDE.md. Прочитал.
3. Открыл src/db/schema/foo.ts (упомянутый в промпте). Прочитал.
4. Открыл src/lib/foo/bar.ts (которое модифицирую). Прочитал.
5. Только теперь — приступил к коду.
```

### Wrong

```
1. Получил промпт. Прочитал заголовок и first 50 строк.
2. Сразу: "ага, понял, нужно создать таблицу X и endpoint Y".
3. Начал писать.
4. Через 30 минут осознал что таблица X уже existsет с другой структурой.
5. Откатил, переделал. Потерял час.
```

---

## Discipline 2 — Spec-gap reporting

### Right

```
Промпт §3.2 говорит:
> "Используй существующий хелпер `formatMoney(kopecks)` из @/lib/utils"

Я открыл @/lib/utils — функция называется `formatCurrency(kopecks)`, не `formatMoney`.

Это spec-gap. Останавливаюсь, сообщаю:
> "В §3.2 промпт ссылается на formatMoney, актуальное имя — formatCurrency. Адаптирую — подтверди?"

Жду ответ. Использую правильное имя.
```

### Wrong

```
Заметил расхождение, подумал "наверное переименовали недавно".
Использовал formatMoney (как в промпте).
TS-error при компиляции.
Полез искать почему.
Через 20 минут — а, на самом деле formatCurrency.
Refactor.
```

---

## Discipline 3 — Atomic transactions

### Right

```typescript
// Канон требует: создание operation + UPDATE связанного payment — atomic.
await db.transaction(async (tx) => {
  const operation = await tx.insert(financialOperation).values(...).returning();
  await tx.update(taxPayment)
    .set({ financialOperationId: operation[0].id, status: "posted" })
    .where(eq(taxPayment.id, paymentId));
  await tx.insert(auditLog).values(...);
});
// Если любая операция fails — все откатываются. Consistency сохранена.
```

### Wrong

```typescript
// Не в transaction — частичные состояния возможны.
const operation = await db.insert(financialOperation).values(...).returning();
// Если процесс crash'ed здесь — operation создана но payment не обновлён.
await db.update(taxPayment)
  .set({ financialOperationId: operation[0].id, status: "posted" })
  .where(eq(taxPayment.id, paymentId));
// audit log может вообще не записаться при крэше
```

---

## Discipline 4 — Idempotency keys

### Right

```typescript
// User clicks "Pay tax" в UI. UI генерирует key один раз на mount.
const idempotencyKey = `tax-payment-${paymentId}-pay`;

// Backend проверяет:
const existing = await db.select().from(taxPayment)
  .where(eq(taxPayment.idempotencyKey, idempotencyKey))
  .limit(1);

if (existing.length > 0) {
  return existing[0];  // already processed, return same result
}

// Otherwise process.
```

### Wrong

```typescript
// Нет idempotency key.
// User кликает "Pay tax" → запрос отправлен. Network задержка.
// User думает "не работает", кликает ещё раз.
// Два запроса на сервере. Два payment созданы. Двойной отток денег.
```

---

## Discipline 5 — Money as integer

### Right

```typescript
// Schema:
amountKopecks: bigint("amount_kopecks", { mode: "number" }).notNull(),

// CHECK:
check("amount_non_negative", sql`amount_kopecks >= 0`),

// Compute:
const grossKopecks = Math.round(taxBaseKopecks * rateNum / 100);

// Display:
formatMoney(grossKopecks);  // "60 000,00 ₽"

// Input parsing:
const kopecks = rublesToKopecks(userInput);  // "1 234,56" → 123456
```

### Wrong

```typescript
// Float arithmetic — precision lost.
amount: numeric("amount", { precision: 10, scale: 2 }),

// Compute:
const tax = baseAmount * 0.06;  // 0.1 + 0.2 = 0.30000000000004 effects

// Display:
`${tax} ₽`;  // "60.00 ₽" not "60,00 ₽" (locale-wrong)
```

---

## Discipline 6 — Brand compliance

### Right

```tsx
<button className="
  bg-[var(--brand-700)] 
  text-white 
  px-6 py-3 
  rounded-[10px] 
  text-[15px] 
  font-medium  /* 500 — допустимый максимум */
  hover:bg-[var(--brand-800)]
  transition-colors duration-150 ease-out
">
  Создать
</button>
```

### Wrong

```tsx
<button className="
  bg-purple-600   /* purple — запрещён */
  text-black      /* запрещён, использовать --fg-1 */
  font-bold       /* запрещён */
  rounded-3xl
  shadow-lg shadow-purple-500/50  /* gradient-like glow */
">
  🚀 Создать сейчас!
  {/* эмодзи + восклицание — запрещены */}
</button>
```

---

## Discipline 7 — Out-of-scope respect

### Right

```
Промпт §0.4 говорит "не трогать legacy `payments` таблицу".

В ходе работы заметил что было бы хорошо рефакторить эту таблицу.
Resist'ил соблазн.
В PR description упомянул:
> "Замечание для будущей фазы: legacy payments table содержит deprecated 
>  колонки X и Y. Рекомендую отдельный cleanup PR."
```

### Wrong

```
Промпт §0.4 говорит "не трогать legacy payments".

Подумал: "Ну она же рядом, заодно зачищу".
Удалил deprecated колонки.
Нарушил scope. PR теперь содержит unrelated changes.
Code-review задерживается на 2 дня — ревьюер должен проверять 2 разных logical changes.
```

---

## Discipline 8 — Error handling

### Right

```typescript
try {
  const result = await postOperation(operationId, userId);
  return Response.json({ data: result }, { status: 200 });
} catch (err) {
  // Известные business-errors → 422 with понятным message
  if (err instanceof BusinessError) {
    return Response.json(
      { error: { code: err.code, message: err.message } },
      { status: 422 }
    );
  }
  
  // PG-error с CHECK violation → 422 с extracted constraint info
  if (isPgError(err) && err.code === '23514') {
    return Response.json(
      { error: { code: 'CONSTRAINT_VIOLATION', message: humanize(err) } },
      { status: 422 }
    );
  }
  
  // Unexpected → log + 500
  console.error('[POST /api/foo]', formatPgError(err));
  return Response.json({ error: 'Internal' }, { status: 500 });
}
```

### Wrong

```typescript
try {
  // ...
} catch (err) {
  console.warn(err.message);  // PG-error detail обрезан, hard to debug
  return Response.json({ error: 'Failed' }, { status: 500 });
  // User видит "Failed" — никакой информации
}
```

---

## Discipline 9 — Comments referencing canon

### Right

```typescript
/**
 * Канон §16 требует НЕ дублировать ФОТ-расход в ОПиУ при проведении 
 * налогов с source='payroll' (НДФЛ-агент, страховые взносы).
 * 
 * Это работает только если ФОТ-зарплата УЖЕ в financial_operation как accrual.
 * Если ФОТ всё ещё на legacy payments — см. acknowledged debt Ф{N} в ROADMAP §10.4.
 */
export function shouldParticipateInOpiu(obligation: TaxObligation): boolean {
  if (obligation.source === "payroll") return false;
  return true;
}
```

### Wrong

```typescript
// returns true if should be in opiu
export function shouldParticipateInOpiu(obligation: TaxObligation): boolean {
  if (obligation.source === "payroll") return false;  // payroll case
  return true;
}
// Через 6 месяцев другой разработчик: "почему именно payroll false?"
// Никаких ссылок. Канон не упомянут. Логика теряется.
```

---

## Discipline 10 — Pre-merge gates

### Right

```
1. Готов реализовать всю фазу. Время gates.
2. pnpm exec tsc --noEmit → 0 errors. ✓
3. pnpm lint → 1 warning ("unused variable").
   - Fix: remove the variable. Re-run.
   - 0 warnings. ✓
4. pnpm db:generate → создаёт migration. Закомичу с code. ✓
5. pnpm build → success. ✓
6. Brand grep — clean. ✓
7. T-сценарии 1-12 — все pass. ✓
8. Готов открывать PR.
```

### Wrong

```
1. Реализовал.
2. pnpm exec tsc → 3 errors про unused imports.
3. "А, не критично, build пройдёт".
4. Открыл PR.
5. CI упал. Coordinator запросил fixы.
6. Fix через несколько часов. Цикл удлинился.
```

---

## Discipline 11 — Branch hygiene

### Right

```bash
# Старт новой фазы:
git fetch --all --prune
git checkout main
git pull --ff-only
git status --short  # clean
git checkout -b feat/taxes-4d3-payments

# Работа...

# Перед PR:
git status  # all changes intentional
git log --oneline -10  # last commits — мои
git push -u origin feat/taxes-4d3-payments
```

### Wrong

```bash
# Уже на какой-то старой ветке.
git status
# > 47 modified files, 12 untracked.
# "Я не помню что это, но точно нужно".

# Создал ветку поверх:
git checkout -b feat/new-stuff
# Закоммитил всё. PR теперь содержит 47+12 unrelated changes.
# Ревьюер: "что это?"
```

---

## Discipline 12 — Session boundaries

Один agent-сессия = одна фаза. Не пытайся делать несколько фаз в одной сессии.

### Right

```
Сессия А (this one): получил Phase 4D.1 промпт. Реализовал. Открыл PR. Сессия закрыта.

Сессия Б (новая, после merge 4D.1): получил Phase 4D.2 промпт. Прочитал предыдущий 
state из current-state.md. Branched. Реализовал. PR. Сессия закрыта.
```

### Wrong

```
Сессия А: реализовал Phase 4D.1. PR открыт.
"Раз уж я в контексте — давай 4D.2 тоже сделаю".
Контекст переполняется. Решения становятся ситуативными.
Когда координатор делает review 4D.1 — нужны корректировки.
Корректировки несовместимы с уже-сделанным 4D.2.
Откат. Месиво.
```

---

## Финальный совет

Дисциплина не убивает скорость — она убивает *неправильную* скорость. Без дисциплины ты двигаешься быстро в первый час и теряешь день на rework. С дисциплиной — двигаешься размеренно весь день и закрываешь PR.

Каждое из этих 12 правил оплачивается одним пропущенным днём rework'а. Окупается быстро.
