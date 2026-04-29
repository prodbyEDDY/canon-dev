---
name: canon-debug
description: Use when encountering any bug, test failure, or unexpected behavior, before proposing fixes. Enforces 4-phase systematic root-cause investigation (reproduce → isolate → diagnose → fix → verify) instead of jumping directly to fix-by-guess. Includes defense-in-depth principles (fix the root cause AND add safeguards), condition-based-waiting patterns (instead of timing-based hacks), and integration with canon-tdd (RED-test reproducing the bug must precede fix).
---

# canon-debug

🚀 Дисциплина systematic debugging в canon-dev pipeline

> Этот скилл — когда что-то сломалось. Применяется до любого fix-attempt'а. Параллелен `canon-tdd` для bug-fix workflow.

---

## Когда использовать

- Test fails и не понятно почему
- Production bug — пользователь жалуется что что-то не работает
- Unexpected behavior во время разработки
- CI упал и причина не очевидна
- Performance regression
- Конкретные ошибки которые не получается воспроизвести

## Когда НЕ использовать

- TypeScript compile errors с явным указанием места — fix straightforward
- Linter warnings — fix straightforward
- Известные проблемы с существующим acknowledged-debt в ROADMAP §10.4

---

## Философия

**Не угадывай. Расследуй.**

Когда сломано — соблазн «попробую fix X, может поможет», «закомменчу эту строку и посмотрю». Это shotgun debugging. Сэкономит 10 минут сейчас, потеряет 2 часа потом когда «fix» окажется неправильным или скроет реальную причину.

**Systematic debugging:**
1. **Reproduce** — точно воспроизвести проблему
2. **Isolate** — минимальный test case
3. **Diagnose** — найти root cause
4. **Fix** — точечный fix причины (не симптома)
5. **Verify** — fix действительно работает + не сломал другое
6. **Defense** — добавить safeguard чтобы не повторилось

Каждая фаза — отдельная. Не прыгай через.

---

## Phase 1 — Reproduce

Без воспроизведения — fix наугад. Реально воспроизвести = понять.

### Steps

1. **Опиши проблему конкретно.** Не «не работает» — а «при действии X в условии Y получаю Z вместо ожидаемого W».

2. **Изоляруй среду.** Локально / staging / production? Какая версия? Какой пользователь? Какие данные?

3. **Воспроизведи минимум 2 раза.** Один раз = может быть случайность. Два = pattern.

4. **Запиши steps to reproduce точно.**

### Шаблон

```
**Bug:** {1-line summary}

**Environment:**
- Branch / commit: {SHA}
- Database state: {fresh / has fixture X / production replica}
- User context: {role / org / specific permissions}

**Steps to reproduce:**
1. ...
2. ...
3. ...

**Expected:** ...
**Actual:** ...

**Reproduction rate:** {100% / 50% — flaky / 1% — rare}
```

### Если не получается воспроизвести

- **Logs** — собери всё что есть
- **Сравни с пользовательским отчётом** — может условия отличаются
- **Add temporary logging** в подозрительные места, deploy, дождись повторения
- **Check error tracker** (Sentry / etc.) — есть похожие?

Не двигайся в Phase 2 пока не воспроизведено локально (или хотя бы на staging с конкретными steps).

---

## Phase 2 — Isolate

Теперь сжать reproduction до minimum.

### Steps

1. **Уберай условия по одному.** Помогает понять что нужно для бага.
   - Без user X — баг есть? Если нет — баг user-specific
   - Без org Y — баг есть? Если нет — данные-specific
   - На fresh DB — баг есть? Если нет — state-specific

2. **Удаляй код по одному** (если можно).
   - Закомментировать middleware → баг ушёл? значит он
   - Bypass cache → баг ушёл? значит cache invalidation issue
   - Без race-condition (single user) → баг ушёл? concurrency

3. **Найти minimum reproducer.** Самый маленький набор условий + кода + данных.

### Цель

После Phase 2 ты можешь точно сказать: **«Bug происходит когда {condition} в коде {component}. Без этого — нет бага.»**

---

## Phase 3 — Diagnose

Теперь — **почему** в этих условиях баг происходит.

### Hypothesis-driven

1. **Сформулируй гипотезу** — что именно идёт не так.
   ```
   Hypothesis: при concurrent payment-create запросах оба видят пустой 
   antiDuplicateKey unique index, оба INSERT'ят, результат — два payments.
   ```

2. **Спроектируй experiment** который **подтвердит или опровергнет** hypothesis.
   ```
   Experiment: запустить 10 concurrent createPayment() с одинаковым key.
   Если hypothesis верна — увидим 2+ payments в DB.
   Если опровергнута — все запросы вернут existing.
   ```

3. **Запусти experiment.** Реально воспроизведи.

4. **Если hypothesis опровергнута** — формулируй следующую. Не говори «странно» — формулируй новую гипотезу.

5. **Если подтверждена** — у тебя root cause.

### Common root causes

Проверь стандартные:

#### Race conditions
- Concurrent writes на одно state
- Symptom: intermittent fails, hard to reproduce

#### Missing transaction boundaries
- Partial state if process crashes mid-flow
- Symptom: orphan records, inconsistent state

#### Caching без invalidation
- Stale data shown
- Symptom: «работает в DevTools, не работает в browser»

#### Off-by-one в дате/времени
- Timezone issues, DST, end-of-month
- Symptom: 1 запись пропадает, edge dates

#### Float precision на деньгах
- Symptom: «копейка не сходится», sums don't match

#### Missing CHECK constraint
- Bypass через direct SQL
- Symptom: invalid state в DB которое app не должна была создать

#### Authorization shortcut
- Permissions проверяются на client, не на server
- Symptom: вручную через DevTools fetch — работает то что не должно

#### N+1 queries
- Loop делает query на каждой итерации
- Symptom: запрос медленный с большим dataset, быстрый с маленьким

#### Stale reference / closure
- React useEffect не передаёт обновлённое значение
- Symptom: UI показывает старое значение после mutation

### Тред Sentry / logs

Часто root cause виден в логах если правильно сформулирован. Поищи:
- Stack traces вокруг времени бага
- Database errors (constraint violations, foreign key fails)
- HTTP errors clusters
- Specific user agents / IPs

---

## Phase 4 — Fix

Теперь **точечный** fix root cause. Не симптома.

### Discipline

1. **Fix причину**, не последствие.
   ```
   ❌ Симптом-fix: «когда два payments — удалять старее».
   ✅ Cause-fix: «добавить advisory lock чтобы concurrent createPayment был сериализован».
   ```

2. **Fix one thing at a time.** Если в Phase 3 нашёл несколько issues — fix каждый отдельно (отдельные commits / PRs). Иначе непонятно какой fix реально помог.

3. **Document why.** В commit message или inline comment — почему именно этот fix.

4. **Use TDD.** Перед fix-кодом — failing test который воспроизводит баг (см. canon-tdd «Bug-fix workflow»). Test не должен passing после revert'а fix'а.

### Anti-patterns

❌ **Fix наугад** («попробую этот try/catch»)
❌ **Suppress error** (`try {} catch (e) { /* ignore */ }`)
❌ **Bypass проверки** (commented-out validation)
❌ **Magic-fix** (изменить значение константы без понимания почему помогло)
❌ **Refactor вместе с fix** — отдельный PR

---

## Phase 5 — Verify

Fix готов? Проверка перед PR.

### Steps

1. **Run failing test** (из Phase 1 / TDD workflow). Должен **pass** теперь.

2. **Run full test suite.** Не сломал ли что-то ещё?
   ```bash
   pnpm test:run
   ```

3. **Reproduce manually** один раз. Не «test passes значит fix работает» — фактически пройди через UI или запрос.

4. **Test edge cases** которые fix мог затронуть.
   - Concurrency: запусти multiple parallel
   - Empty state
   - Maximum values
   - Permissions

5. **Test inverse** — то что fix НЕ должен был сломать.

---

## Phase 6 — Defense in depth

Bug-fix не закончен после Phase 5. Phase 6 — **превентивная**.

### Defense layers

После понимания root cause — добавить safeguards чтобы не повторилось:

#### Layer 1 — Fix в коде

(Phase 4)

#### Layer 2 — Test для regression

(canon-tdd: failing test → green after fix → остаётся forever)

#### Layer 3 — DB-level constraint (если применимо)

Если bug — race condition leading to invalid state, добавь CHECK / UNIQUE constraint:

```sql
ALTER TABLE payment 
  ADD CONSTRAINT no_duplicate_key 
  UNIQUE (organization_id, idempotency_key);
```

DB ловит даже если app-level fix забыли.

#### Layer 4 — Monitoring

Добавь metric / alert для подозрительного состояния:

```typescript
// Если detected duplicate-attempt — log warning + send to Sentry
if (existingPayment) {
  logger.warn('Duplicate payment attempt detected', { key, ... });
}
```

#### Layer 5 — Documentation

Если deviation от каноничного pattern — добавь в ROADMAP §10.4 как acknowledged debt с условиями закрытия.

### Why defense matters

Один root-cause fix — point fix. Через 6 месяцев другой разработчик может accidentally re-introduce. Defense-in-depth ловит таких.

Каждый layer — independently catches. Fix может revert'нуться, test может удалиться, но constraint остаётся.

---

## Condition-based waiting (vs timing-based)

Часто bug fixes / tests включают «ждать пока что-то произойдёт». **Никогда** через `setTimeout(2000)` — это flaky.

### ❌ Плохо

```typescript
test('async operation finishes', async () => {
  startOperation();
  await new Promise(resolve => setTimeout(resolve, 2000));  // 2 секунды наугад
  expect(state.completed).toBe(true);
});
```

Проблемы:
- Slow CI → 2 секунды мало → fail
- Fast CI → 2 секунды лишние → wasted time
- Race condition между setTimeout и реальной готовностью

### ✅ Хорошо

```typescript
test('async operation finishes', async () => {
  startOperation();
  await waitFor(() => expect(state.completed).toBe(true), { timeout: 5000 });
});

// helper:
async function waitFor(condition: () => void | Promise<void>, opts: { timeout?: number } = {}) {
  const timeout = opts.timeout ?? 5000;
  const start = Date.now();
  while (Date.now() - start < timeout) {
    try {
      await condition();
      return;  // success
    } catch (err) {
      await new Promise(resolve => setTimeout(resolve, 50));
    }
  }
  // last attempt — let it throw with full error
  await condition();
}
```

Wait until condition met, not arbitrary time.

В production code — same principle: use observable signals (events, callbacks, polling-with-condition), not arbitrary delays.

---

## Anti-patterns

❌ **«Я знаю что это, попробую X»** — без воспроизведения, без понимания. Phase 1 reproduce обязателен.

❌ **Comment out failing code** — это suppress, не fix. Test остаётся broken.

❌ **«Это одноразовое, не нужен test»** — это самообман. Bug return через 6 месяцев.

❌ **Multiple fixes в одном PR** — нельзя isolate какой реально помог. Один fix = один PR.

❌ **Skip Phase 6 defense** — root-cause-fix без layered defense — fragile. Кто-то может revert.

❌ **Пропустить Phase 5 verify** — «должен работать, я уверен». Не уверен — проверь.

---

## Critical principles

1. **Reproduce first.** Без воспроизведения — fix наугад.
2. **Hypothesis-driven diagnose.** Формулируй predictions, тести experiments.
3. **Cause-fix, не symptom-fix.** Симптом могут возвращаться.
4. **TDD для bug-fix.** RED test reproduces bug, GREEN после fix.
5. **One fix per PR.** Изолированные изменения, легче review/revert.
6. **Defense-in-depth.** Code fix + test + constraint + monitoring + docs.
7. **Condition-based waiting.** Никогда timing-based.

---

## Templates и references

- `references/diagnosis-checklists.md` — common root-causes + signature symptoms
- `references/condition-waiting-patterns.md` — расширенные wait-patterns

---

## Финальная заметка

Хороший debugging — медленный сначала, быстрый потом. Shotgun debugging — наоборот: быстро в начале, два дня rework'а в конце.

Discipline стоит 30 минут на bug. Окупается каждым ходом когда fix реально работает с первого раза.
