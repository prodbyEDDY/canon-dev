# Phase {N} — {Имя фазы}

> {Одно предложение: что делается + canon-источник + dependency на предыдущие фазы}

---

## 0. Контекст и scope

### 0.1. Что за продукт

{Краткое описание продукта в 2-3 предложения. Стек ссылкой на CLAUDE.md или STACK.md.}

### 0.2. Что строим в этой фазе

{Скоп текущей фазы в 4-6 предложений. Какие сущности, какой UI, какие API.}

### 0.3. На чём строимся (зависимости)

{Какие предыдущие фазы должны быть merged. Какие конкретные файлы / таблицы / API уже существуют.}

```
✅ Phase {prev}: schema X, таблицы Y/Z, API /api/x/, UI /pages/x/
✅ Phase {prev-1}: ...
```

### 0.4. Чего НЕ делаем (out-of-scope)

{Явно перечислить что в этой фазе **не делается**:}

- ❌ {Функциональность 1} — отложена в Phase {next}
- ❌ {Функциональность 2} — out-of-scope всей программы (было решение)
- ❌ Не трогаем {existing module} — backward compatibility
- ❌ Не менять {file} — это будет затронуто в другой фазе
- ❌ ROADMAP / progress / journal — координатор обновляет сам

### 0.5. Канон-источники

{Ссылки на конкретные секции канонов с цитатами ключевых положений. Чтобы агент НЕ открывал .docx.}

> Канон §X.Y: «{цитата}»

---

## 1. Жёсткие правила (бренд + код)

### 1.1. Брендовые правила (v2 design system)

{Краткий список из дизайн-канона — никакого Bold, single brand color, no эмодзи, etc.}

### 1.2. Code conventions

- TypeScript strict, никакого `any` без обоснования
- Money = integer kopecks/cents, никогда float
- All API endpoints: session check → orgId → orgId-фильтрация → log
- Все mutations критичные → `db.transaction()`
- Idempotency keys для user-facing creates (детерминированный формат)

---

## 2. Schema additions

{Если есть schema-изменения. Полные TypeScript-определения таблиц с CHECK constraints, indexes, partial unique indexes для антидублей.}

```typescript
export const newTable = pgTable("new_table", {
  id: serial("id").primaryKey(),
  organizationId: text("organization_id").notNull(),
  // ...
}, (t) => [
  index("idx_new_table_org").on(t.organizationId),
  check("amount_non_negative", sql`amount_kopecks >= 0`),
  // ...
]);
```

После schema — обязательно `pnpm db:generate` создаст миграцию. **Закоммить и schema, и migration файл вместе** (см. CLAUDE.md «Database Migrations CRITICAL»).

---

## 3. {Business logic / Engine / Расчётный движок}

{Если есть расчётная логика. Pure-функции в `src/lib/{module}/`. Каждая формула — со ссылкой на canon-секцию + примеры значений.}

```typescript
/**
 * Канон §X.Y — {название формулы}.
 * 
 * formula: result = func(input1, input2)
 * 
 * Edge cases:
 * - input1 = 0 → return null with status="no base"
 * - input2 < 0 → throw (invalid input)
 */
export function compute({ ... }: Input): Result {
  // ...
}
```

---

## 4. API contracts

### 4.1. POST /api/{module}/{entity}

```typescript
// Body (Zod)
const schema = z.object({
  field1: z.string().min(1),
  field2: z.number().int().positive(),
});

// Response 200
{ data: Entity }

// Response 400/422
{ error: { code, message, field? } }
```

**Логика handler'а:**
1. Session check → orgId
2. Validate body
3. Бизнес-проверки (referenced canon §X)
4. INSERT в `db.transaction`
5. Audit log via `logEvent`
6. Return 200

### 4.2. ...

(Каждый endpoint детально)

---

## 5. UI changes

{Компоненты + layout + состояния. Использовать существующую дизайн-систему — ссылка.}

### 5.1. Page `/path/to/page`

Layout: {описание}
Компоненты: {list}
States: loading (Skeleton) / empty / error / success

### 5.2. Component `<NewComponent />`

```typescript
type Props = {
  // ...
};
```

---

## 6. Cross-module integration

{Hook'и в существующие endpoint'ы или API. Antidubli механизмы. Cross-module contracts.}

---

## 7. T-сценарии (агент проверяет локально)

1. **T-01.** {Сценарий} → {ожидаемый результат}
2. **T-02.** {Edge case} → {expected behavior}
3. **T-03.** {Error case} → {expected error message}
4. ...

(Минимум 8-15 сценариев)

---

## 8. Pre-merge чек-лист

```bash
pnpm exec tsc --noEmit       # 0 errors
pnpm lint                     # 0 errors, 0 warnings
pnpm db:generate              # СОЗДАЁТ миграцию (если есть schema-изменения)
pnpm build                    # success
pnpm dev                      # http://localhost:3000/{path} — рендерится
```

**Brand-compliance grep:**
```bash
grep -rn "font-bold\|font-semibold" src/{paths-changed}/ 2>/dev/null
# Expected: 0

grep -rn "purple\|violet\|teal\|emerald\|orange\|amber" src/{paths-changed}/ 2>/dev/null
# Expected: 0

grep -rn "text-black\|bg-black\|text-gray-[0-9]" src/{paths-changed}/ 2>/dev/null
# Expected: 0
```

**Проверки:**
- [ ] Schema: все таблицы созданы с CHECK constraints + indexes
- [ ] Migration файл закоммичен
- [ ] T-01..T-N все проходят локально
- [ ] Расчёты на учебных примерах canon §Y верны
- [ ] Brand-compliance grep чистый

---

## 9. Out-of-scope (НЕ делать)

{Повтор раздела 0.4 для агента который читает только середину промпта.}

---

## 10. PR-описание шаблон

**Title:** `feat({module}): {краткое описание} (Phase {N})`

**Body:**
1. Контекст (depends on Phase {prev}, см. ROADMAP §5)
2. Что сделано:
   - Schema: {list}
   - API: {list endpoints}
   - UI: {list components}
3. T-сценарии {1-N} — пройдены: ✅ / ⚠️ с пояснением
4. Скриншоты: desktop / tablet / mobile
5. Acknowledged deviations: {если есть}

**Co-Authored-By:** {standard footer}

---

## 11. T-сценарии для координатора (post-merge на staging)

(Сценарии что координатор проверяет на staging-prod после merge)

- **TC-01.** {Сценарий happy path}
- **TC-02.** {Регрессия — что не должно сломаться}
- **TC-03.** {Mobile / responsive flow}

---

## 12. Финальные акценты для агента

{1-3 наиболее важных риск-зоны для этой фазы. Например:}

1. **Atomic transactions критичны.** Не оставляй частичные состояния — оборачивай в `db.transaction`.

2. **Idempotency keys на mutations** — формат `{module}-{id}-{action}`. UI-формы генерят один раз на mount.

3. **Используй existing helpers** ({пример имени}). Не дублируй INSERT в DB напрямую.

Удачи. Когда закроешь PR — координатор сделает code-review и подготовит promtp следующей фазы.
