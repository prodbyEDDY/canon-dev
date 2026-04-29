# Severity calibration — как классифицировать findings

> Конкретные критерии и калибровочные примеры для классификации severity findings в `canon-verify`.

---

## 🔴 Critical

### Критерии (хотя бы один)

1. **Production risk:** в production произойдёт катастрофа — потеря данных, неверные денежные расчёты, нарушение compliance
2. **Security:** уязвимость, leak PII, unauthorized access path, cross-tenant data exposure
3. **Money correctness:** float для денег, mutable journal, missing posting_date, неправильные ОПиУ/ОДДС-flags для accounting modules
4. **Antidubli failure для финансовых операций:** возможен двойной счёт денег
5. **Multi-tenant leak path:** один user может видеть другую org

### Калибровочные примеры

✅ **Critical** — деньги в float:
> Schema `tax_obligation.amount` defined as `numeric(10,2)` instead of `bigint` для копеек. Precision loss inevitable. Found in commit X, file Y:Z.

✅ **Critical** — missing posting_date:
> `postOperation` в `src/lib/finance/operations.ts:236` устанавливает status='posted' но не posting_date. CHECK `fo_posted_date_chk` violation на каждом auto-post. Same bug as Ф27.

✅ **Critical** — cross-tenant leak:
> `GET /api/users/[id]` not filtering by organizationId. Any authenticated user can fetch user details from другой org by guessing ID.

❌ **Не Critical** (Major or below):
> Lifecycle переход `draft → completed` напрямую (минуя `active`) разрешён в state machine, хотя canon §15.2 запрещает.

— Это lifecycle gap. Major. Not Critical потому что (а) можно catch в QA, (б) fix на service-layer, (в) не теряет деньги.

---

## 🟡 Major

### Критерии

1. **Архитектурный gap:** дальнейшая разработка опирается, потребует рефакторинг через несколько фаз
2. **Lifecycle hole:** не описан переход или edge case, потенциальный баг в QA
3. **Performance / scale:** ожидаемый объём не учтён (например, нет pagination на list-endpoint)
4. **Cross-module integration не работает:** контракты соблюдены частично
5. **Antidubli отсутствует** для не-критичных flows (например, документ может задвоиться, не деньги)

### Калибровочные примеры

✅ **Major** — pagination отсутствует:
> `GET /api/contracts` returns full list без pagination. Будет упасть на org с 5k+ contracts. Quick fix: add page/limit params + max 100 cap.

✅ **Major** — lifecycle gap:
> Canon §15.3 описывает 6 статусов tax_payment, но в коде state machine есть только 4. Missing: `error`, `canceled` transitions.

✅ **Major** — antidubli for documents:
> `POST /api/documents/upload` не проверяет на дубликат по filename + size + counterpartyId. User может accidentally upload one document twice.

❌ **Не Major** (Minor):
> Code uses `console.warn(err.message)` instead of `formatPgError(err)` — некоторые PG-error details обрезаются.

— Это logging quality, не functionality. Minor.

---

## 🟢 Minor

### Критерии

1. **Documentation quality:** terminology inconsistent, missing example
2. **Code style:** unused variables, suboptimal naming
3. **Brand violations** мелкие (hardcoded color в одном месте, font-bold в одном файле)
4. **Cross-link broken** в документации
5. **Logging suboptimal:** убогие error messages, missing context
6. **Edge case в UI:** скажем, empty state без CTA

### Калибровочные примеры

✅ **Minor** — brand violation:
> В `src/app/(dashboard)/foo/page.tsx:42` использован `text-gray-600` вместо `text-[var(--fg-2)]`. Cosmetic, fix simple.

✅ **Minor** — terminology:
> В `Канон_Финансы.md` термин «контрагент» используется. В `Канон_Налоги.md` — иногда «партнёр». Inconsistent.

✅ **Minor** — missing example:
> В canon §21.3 описана формула УСН-Доходы-Расходы с минимальным налогом, но нет примера расчёта для случая когда min_tax > regular_tax.

---

## 💡 Suggestion

### Критерии

1. **Improvement, not gap:** функциональность реализована, можно улучшить ясность
2. **Optional refactor:** дублирование кода, можно вынести в shared helper
3. **Performance optimization** на edge case
4. **Documentation enhancement:** дополнительный диаграмма / диаграмма / TOC

### Калибровочные примеры

✅ **Suggestion**:
> `src/lib/taxes/calculation/usn-income.ts` и `usn-income-minus.ts` дублируют ~50 строк rate-resolution logic. Можно вынести в `src/lib/taxes/calculation/shared/rate-resolver.ts`.

✅ **Suggestion**:
> ROADMAP.md §11 (журнал) растёт с каждой фазой. Через 20 фаз будет 1000+ строк, hard to navigate. Предложить разбить на `journal/{year}.md` files.

---

## Decision tree для классификации

```
Это finding (gap) или improvement?
├─ Improvement → 💡 Suggestion
└─ Gap
    │
    Production risk / security / money / cross-tenant?
    ├─ Yes → 🔴 Critical
    └─ No
        │
        Architectural / scale / integration / lifecycle gap?
        ├─ Yes → 🟡 Major
        └─ No → 🟢 Minor
```

---

## Anti-patterns

❌ **«Всё Critical»** — теряет приоритезацию. Если 30 Critical findings — пользователь не знает с чего начинать.

❌ **«Всё Suggestion»** — занижение severity, реальные риски пропадают в шуме.

❌ **Subjective severity** — «мне не нравится structure модуля». Это Suggestion в крайнем случае. Если действительно мешает — articulate WHY (e.g., «module structure prevents reuse — duplicates Y in 3 places»).

❌ **Severity по personal preference** — «я бы выбрал другую naming convention». Не финдинг если canon не предписывает specific naming.

❌ **Inflate severity для attention** — иногда хочется выделить finding. Не делай Major из Minor чтобы coordinator обратил внимание. Используй "Recommendations" section.

❌ **Defl ate severity** — иногда не хочется создавать problem. Не делай Minor из Critical чтобы избежать hotfix. Honest reporting — обязанность.

---

## Калибровочные сценарии

### Сценарий 1: Дублирование кода

**Finding:** В трёх местах копируется одна и та же 30-line логика resolution-фактора.

- **Сразу:** Suggestion (refactor opportunity)
- **Если эта логика — формула с canon-ref и есть risk что её обновят в одном месте а в других забудут:** Major (architectural risk)
- **Если эта логика для money calculations и uncatched bug приведёт к неверным числам:** Critical (через risk multiplication)

Severity зависит от **последствий**, не от cosmetic размера.

### Сценарий 2: Hardcoded value

**Finding:** В коде magic number `60_000_000` (60 млн) — лимит дохода для УСН.

- **Сразу:** Major (hardcoded threshold вместо registry — тех-долг при изменении законодательства)
- **Если этот hardcode — единственное место где УСН применяется:** Major (когда лимит изменится через год — баг)
- **Если такие hardcodes в нескольких местах с разными цифрами (например, в одном 60M, в другом 50M, drift):** Critical (already inconsistent)

### Сценарий 3: Missing CHECK constraint

**Finding:** Schema у `tax_obligation` не имеет CHECK `paid_kopecks <= amount_kopecks`.

- **Если в service layer есть check:** Major (defense in depth missing — DB не защищает от direct SQL bypass)
- **Если в service layer тоже нет:** Critical (paid > amount возможно через любой path)

### Сценарий 4: console.log в production

**Finding:** В нескольких файлах остались `console.log` debug statements.

- **Если в server-only code (logs только в server logs):** Minor
- **Если в client-side code (logs в browser console — visible to users):** Minor (unprofessional но не security)
- **Если log содержит PII or sensitive data:** Critical (compliance issue)

### Сценарий 5: Lifecycle missing transition

**Finding:** Canon §15.2 описывает переход `error → draft` (recovery from error). В коде такого перехода нет.

- **Если эта рекавери не automatic, а user-initiated:** Major (UX gap, user stuck в error state)
- **Если recovery должен быть automatic at система level:** Critical (data может застрять в bad state)

---

## Финальный совет

Severity — **dispute-able**. Иногда coordinator не согласится с твоей классификацией. Это норм. Главное — **explicit reasoning** в finding'е.

Если в finding ты явно объясняешь «это Critical потому что {criterion}» — даже если coordinator переклассифицирует в Major, decision based на reasoning, не на feeling.

Худший finding — без reasoning. Best — с конкретным criterion из списка.
