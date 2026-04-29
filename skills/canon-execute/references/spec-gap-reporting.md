# Spec-gap reporting — шаблоны

> Когда промпт расходится с реальностью — стоп, доложи координатору. Это спасает день. Шаблоны ниже — для разных типов расхождений.

---

## Когда репортить

**Репортить:**
- API-функция с другим именем / сигнатурой
- Отсутствующая колонка в schema
- Endpoint существует но с другим контрактом
- Файл который должен existsовать, не existsует
- Пакет не установлен (хотя промпт его упоминает)
- Конвенция отличается от описанной в промпте
- Любая constraint-нарушающая просьба (e.g. NOT NULL field в место где промпт хочет null)

**НЕ репортить (просто адаптируй):**
- Опечатка в имени переменной (если очевидно что имеется в виду)
- Косметические recommendations промпта которые не работают (e.g., промпт предлагает icon X, но в lucide иконка называется Y — используй Y)
- Незначительные стилевые расхождения которые не меняют семантику

**Граница:** если изменение влияет на behavior / API / DB — репортить. Если только на naming или style — адаптируй.

---

## Шаблон 1 — API signature mismatch

```markdown
**Spec gap detected:**

Промпт §{X.Y} references function `{expected-name}({expected-params})`.

Actual API in `{file:line}`:
```typescript
export function {actual-name}({actual-params}): {actual-return} { ... }
```

**Difference:**
- Function name: {expected} → {actual}
- Param names: {expected list} → {actual list}
- Return type: {expected} → {actual}

**Adaptation I propose:**
Use {actual-name} with mapping: {how to map old params to new}.

**Confirm OK to proceed?**
```

---

## Шаблон 2 — Schema mismatch

```markdown
**Spec gap detected:**

Промпт §{X} assumes column `{expected-column}` on table `{table}`.

Actual schema in `src/db/schema/{file}.ts`:
```typescript
export const {table} = pgTable("{table}", {
  // ... no `{expected-column}` field
});
```

**Difference:**
- Expected: `{column} text NOT NULL`
- Actual: column doesn't exist

**Adaptation options:**
1. Add the column (schema migration) — but this is broader scope
2. Use existing column `{alternative}` which provides similar information
3. Use a join with another table to derive the value

**My recommendation:** {option N}, because {reasoning}.

**Confirm OK to proceed with option {N}?**
```

---

## Шаблон 3 — Missing file/module

```markdown
**Spec gap detected:**

Промпт §{X} expects helper `{path/to/file.ts}` to exist.

Actual: `{path}` doesn't exist or доступно по другому пути.

**Investigation:**
```bash
find src/ -name "*{relevant-keyword}*"
# Returns: {actual paths found}
```

**Adaptation:**
Most likely the helper is at `{actual-path}` based on naming pattern. I'll use that.

OR:

Helper doesn't exist at all. **Should I create it as part of this phase**, or is this expected from previous phase (which means I'm on the wrong branch)?

**Confirm:** {options}
```

---

## Шаблон 4 — Constraint conflict

```markdown
**Spec gap detected — schema constraint:**

Промпт §{X} asks me to insert `financialOperation` with `financeAccountId: null`.

But schema has:
```typescript
financeAccountId: integer("finance_account_id").notNull(),
```

This means `null` will violate NOT NULL constraint.

**Options:**
1. **Make schema field nullable** — schema migration, broader scope, может затронуть other usages
2. **Use synthetic "accrual" account** — create one per org, point all accruals to it
3. **Defer creation** — move logic to a different time when real account is known
4. **Use existing account** — even if semantically less precise

**My recommendation:** {N} because {reasoning, including trade-offs}.

This is a design decision, not just adaptation. **Need explicit coordinator confirmation.**
```

---

## Шаблон 5 — Architectural deviation needed

```markdown
**Architectural deviation needed:**

Промпт §{X} specifies architecture A. During implementation, discovered constraint that makes A impossible/expensive.

**The constraint:**
{Explain technical reality, e.g. "Schema requires NOT NULL field that would need to be NULL for plan A"}

**Plan A (as specified):**
{Diagram or pseudocode of original plan}

**Plan B (proposed):**
{Diagram or pseudocode of adapted approach}

**Trade-offs:**
- ✅ B preserves: {what semantic property is preserved}
- ⚠️ B differs from A: {what changes}
- 📋 Edge case where A and B differ: {когда поведение recombines or diverges}

**Canonical correctness:**
Plan B {still satisfies / partially satisfies / deviates from} canon §{Z}. {Reasoning}.

**Suggested PR description note:**
"Architectural deviation: instead of {A}, implemented {B} due to {constraint}. {Property} is preserved; {trade-off} acknowledged."

**Need coordinator decision:** approve plan B, or specify alternative?
```

---

## Шаблон 6 — Multiple small gaps (batched)

Если нашёл несколько мелких — репорти batch'ем, не по одному:

```markdown
**Multiple spec gaps detected — batched report:**

Pre-flight investigation surfaced {N} issues. None blocking individually,
but want confirmation before adapting:

1. **API name:** `formatMoney` → `formatCurrency` (helper renamed). 
   Adaptation: use actual name.

2. **Token name:** `--brand-primary` doesn't exist; use `--brand-700`.
   Adaptation: use --brand-700.

3. **Module path:** `src/lib/finance/operations.ts` → `src/lib/finance/operations/index.ts`
   (refactored to folder).
   Adaptation: imports updated.

4. **Conventions:** existing pattern uses `bigint` mode "number", промпт
   uses "bigint". 
   Adaptation: use mode "number" matching codebase.

**All adaptations are mechanical mappings — no architectural changes.**

**OK to proceed with these adaptations?**
```

---

## Чего НЕ делать

❌ **Silent adaptation** — заметил расхождение, тихо изменил, продолжил. Координатор узнает только при code-review. Стоимость = день rework если adaptation неправильна.

❌ **Excessive detail для тривиальностей** — если расхождение чисто mechanical (rename), не пиши страницу анализа. Один-два sentence достаточно.

❌ **«Я подожду пока можно сделать без расхождения»** — не ждёшь. Репортишь и продолжаешь работу над частью где расхождений нет (если есть таковая) или ждёшь.

❌ **Fix без репорта** — «я сам fix'нул spec gap в промпте». Нет, промпт = source-of-truth, он не fix'ится агентом-реализатором. Fix — координаторская задача.

---

## После получения подтверждения

```
1. Записать adaptation в комментарий в коде:
   ```typescript
   // Adapted from spec §X: {expected} → {actual}. 
   // Approved by coordinator on {date}.
   ```

2. Включить в PR description секцию "Adaptations":
   ```markdown
   ## Spec adaptations (approved by coordinator)
   1. {gap 1} — adapted to {solution}
   2. {gap 2} — ...
   ```

3. Не записывать как "deviation" — это не deviation, это правильная адаптация.
   Deviation — это когда сам код отличается от plan по design-choice.
   Adaptation — когда плановое имя/structure не совпадает с актуальным state.
```

---

## Resume work after confirmation

Получил подтверждение → продолжай. Не делай больше адаптаций без новых spec-gap reports.

Если по ходу работы найдёшь ещё расхождения — снова стоп, новый report. Агент пересылает много spec-gap reports — это нормально, **лучше** чем silent adaptation.

Координатор за это благодарен. Каждый spec-gap report = бесплатный апдейт для координатора который улучшает следующий промпт.
