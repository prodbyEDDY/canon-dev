# Dimension checklist для canon-verify

> Расширенная матрица 16+ измерений с конкретными контрольными вопросами для аудита **реализации** (не канонов — для канонов используется `canon-review`).

Для каждого измерения — список вопросов. Если ответ «не реализовано в коде» или «реализовано неправильно» — это finding.

---

## D1. Lifecycle correctness

Для каждой сущности с состояниями (status enum):

- [ ] Все enum-значения из canon представлены в коде schema?
- [ ] State-machine в service-слое (если есть) соответствует canon-описанию переходов?
- [ ] Запрещённые переходы блокированы (либо service-level error, либо DB-level CHECK)?
- [ ] При попытке нелегального перехода — пользователь видит понятный error message (не silent fail)?
- [ ] «Closed» состояние действительно защищено (нельзя cardinal mutate)?
- [ ] Re-opening flow требует role + reason (если canon так предписывает)?
- [ ] Soft-delete vs hard-delete реализован per canon?
- [ ] Cascade behaviour при cancel parent — реализован per canon?

**Severity rule:** missing legal transition = Major. Missing illegal-transition block = Critical (data integrity).

---

## D2. Antidubli implementation

- [ ] Idempotency keys присутствуют на всех user-facing creates per canon?
- [ ] Format keys детерминированный (e.g. `{module}-{id}-{action}`)?
- [ ] Partial unique indexes на критических ключах?
- [ ] Re-import / retry verified non-duplicating (test scenario)?
- [ ] Race conditions защищены через `pg_advisory_xact_lock` или эквивалент?
- [ ] При duplicate key — graceful return existing, не error?

**Severity rule:** missing antidubli for monetary operations = Critical. For non-critical = Major.

---

## D3. Atomic transactions

- [ ] Critical-flow обёрнут в `db.transaction`?
- [ ] Если используется existing helpers (createOperation, postOperation) — поддерживают optional `tx` parameter (rerefactor если необходим)?
- [ ] Mental rollback test: если процесс crash'ит после INSERT — есть orphan-data или нет?
- [ ] Audit-log writes в той же transaction что main mutation?
- [ ] Cross-table updates atomic?

**Severity rule:** non-atomic critical-flow = Critical. Non-atomic helper updates = Major.

---

## D4. Multi-tenancy enforcement

Iterate каждый new endpoint:

- [ ] Session check via `auth.api.getSession`?
- [ ] orgId извлекается из session?
- [ ] **Все** SELECT/UPDATE/DELETE WHERE содержат `organizationId = orgId`?
- [ ] JOINs между таблицами безопасны (orgId через цепочку)?
- [ ] No way to access другой org через ID-guessing (e.g., `/api/foo/[id]` где id известен из другого org)?

**Severity rule:** any cross-tenant leak path = Critical (catastrophic if shipped).

---

## D5. Money correctness

- [ ] Все денежные columns — `bigint` или `integer` в копейках/центах?
- [ ] No `numeric` / `decimal` / `float` для денег?
- [ ] CHECK constraints `>= 0` где применимо?
- [ ] Округление consistent (Math.round / Math.floor — одинаково везде)?
- [ ] Display через `formatMoney` или эквивалент?
- [ ] Парсинг через `rublesToKopecks` или эквивалент?
- [ ] Деление на 0 в delta-расчётах — fallback не Infinity/NaN?
- [ ] Никаких `Number.parseFloat` на user input для money?

---

## D6. Posting / accounting invariants

(Применимо если product имеет accounting layer)

- [ ] `posting_date` устанавливается **каждый** раз при transition в status='posted'?
- [ ] Это invariant защищён CHECK constraint в schema?
- [ ] Нет UPDATE на posted-records кроме canonically allowed (storno chain)?
- [ ] ОПиУ-flag (`isOpiuParticipant`) правильный для каждого типа operation per canon?
- [ ] ОДДС-flag (`isOddsParticipant`) правильный?
- [ ] Antidubli с ФОТ / другими модулями source-aware?
- [ ] Storno создаёт **новую** operation, не reverse existing?

**Severity rule:** posting_date missing in any posting path = Critical (already burned us once).

---

## D7. Schema integrity

- [ ] CHECK constraints per ТЗ присутствуют?
- [ ] Indexes для frequently-queried (org+status, org+date_range, etc.)?
- [ ] Audit-log tables — нет UPDATE/DELETE-statements в коде?
- [ ] Partial unique indexes на conditional uniqueness?
- [ ] Foreign keys (или explicit cross-tenant guard) на все ссылки?
- [ ] Default values корректные?
- [ ] NOT NULL where required?

```bash
# Schema drift check
pnpm db:generate
# Should output "no schema changes" если migrations applied
```

---

## D8. API patterns (CLAUDE.md compliance)

Iterate каждый API endpoint:

- [ ] Session check присутствует?
- [ ] orgId извлечён?
- [ ] Body validation через Zod (для POST/PUT/PATCH)?
- [ ] Query params validation для GET с filters?
- [ ] Pagination на list endpoints (max limit определён)?
- [ ] Try/catch с правильными HTTP-кодами (400/401/403/404/409/422/500)?
- [ ] `logEvent` (или эквивалент) для critical mutations?
- [ ] Error messages понятные, не raw stack-trace?

---

## D9. Calculation accuracy

(Если расчётный модуль)

Iterate каждую расчётную функцию:

- [ ] Формула соответствует canon §X.Y (cross-reference в JSDoc)?
- [ ] Edge cases (zero base, negative input, missing data) обработаны?
- [ ] Результат — integer (если деньги)?
- [ ] Округление per canon style?
- [ ] Учебные примеры canon §Y действительно дают expected числа? **Run mentally или programmatically.**

**Test for each canonical example:**

```typescript
// Canon §26.1: УСН-Доходы 1М × 6% = 60k
const result = calculateUsnIncome({
  taxBase: 100_000_000,  // 1M в копейках
  rate: 6,
});
expect(result.amountKopecks).toBe(6_000_000);  // 60k в копейках
```

Если test fails — расчёт неправильный.

---

## D10. Cross-module integration

- [ ] Каждая cross-module hook реализована per spec?
- [ ] Hook вызывается в правильный момент (например, после approve, не до)?
- [ ] Failure of hook не ломает primary flow (или ломает per design)?
- [ ] Antidubli защита присутствует (если hook может срабатывать дважды)?
- [ ] Contract data shape совпадает с тем что second module expects?

---

## D11. Brand compliance (UI)

```bash
# Bold weights
grep -rn "font-bold\|font-semibold\|fontWeight: *[6789]\|font-[678]00" src/{paths-changed}/

# Off-brand colors
grep -rn "purple\|violet\|teal\|emerald\|orange\|amber\|fuchsia\|cyan\|rose" src/{paths-changed}/

# Hardcoded colors
grep -rn "text-black\|bg-black\|text-gray-[0-9]\|bg-gray-[0-9]" src/{paths-changed}/

# Inline hex
grep -rn "#[0-9a-fA-F]\{6\}\|#[0-9a-fA-F]\{3\}" src/{paths-changed}/_components/
```

Все должны вернуть 0 results. Иначе finding (severity Minor — кроме случаев когда это явное продуктовое решение).

- [ ] No эмодзи в production UI (manual scan)?
- [ ] Все размеры через токены (no inline px кроме canonical exceptions)?
- [ ] Иконки только из утверждённой icon-system?

---

## D12. UI correctness

Для каждой новой страницы:

- [ ] Loading state (Skeleton)?
- [ ] Empty state (с meaningful message + CTA если применимо)?
- [ ] Error state (с retry option)?
- [ ] Responsive: 375px, 768px, 1024px, 1440px?
- [ ] Focus rings видны на keyboard navigation?
- [ ] Alt-tags на images?
- [ ] ARIA-labels на icon-only buttons?
- [ ] Один h1 на странице, остальные h2/h3 hierarchy?
- [ ] `prefers-reduced-motion` respected (если есть animations)?

---

## D13. Documentation hygiene

- [ ] Inline comments на нетривиальной логике?
- [ ] Reference на canon §X.Y в комментариях ключевых функций?
- [ ] JSDoc на public API helpers?
- [ ] README в новых module-folders (где architecturally нетривиально)?
- [ ] Architectural deviations документированы in-code (TODO с reference на ROADMAP §10.4)?

---

## D14. Tests / acceptance

- [ ] T-сценарии из phase prompt — pass?
- [ ] Учебные примеры canon — verify программно или mentально?
- [ ] Edge cases coverage в коде (try/catch, defensive checks)?
- [ ] Acceptance criteria из ТЗ — measurable verified?

---

## D15. Performance hot spots

- [ ] N+1 queries предотвращены (использован JOIN или batch-load)?
- [ ] Pagination на list-endpoints с reasonable limits?
- [ ] Heavy queries индексированы (verify через `pnpm db:studio` или EXPLAIN)?
- [ ] Hot loops без unnecessary re-computations?
- [ ] Cache (где применимо) с TTL и invalidation?

---

## D16. Security

- [ ] No cross-tenant leak paths (overlap с D4 но specifically security-focused)?
- [ ] Input validation everywhere (Zod на user input)?
- [ ] No SQL injection vectors (Drizzle parameterized queries)?
- [ ] No XSS vectors (React по умолчанию escape, но проверь dangerouslySetInnerHTML)?
- [ ] Sensitive data not in logs (PII / payment info masking)?
- [ ] Rate limiting на login / password-reset / similar attack-prone endpoints?
- [ ] CSRF protection где applicable?
- [ ] Secrets — env vars или secret manager, не hardcoded?

---

## D17. Migration / deployment safety

- [ ] Migration file present для schema changes?
- [ ] Migration не drops columns с data (если так — backfill plan?)
- [ ] Backward compatibility soigne'но (если deploy is rolling)?
- [ ] No DESTRUCTIVE operations без user confirmation?

---

## Custom dimensions

Для специфичных доменов:

### D18. Регуляторика (если регулируемый домен)
- [ ] Compliance с применимым законодательством?
- [ ] Audit log retention per regulation?
- [ ] PII handling per GDPR / 152-ФЗ / etc.?

### D19. Real-time (если real-time критичный)
- [ ] WebSocket reconnection logic?
- [ ] Latency monitoring?

### D20. ML / AI (если LLM components)
- [ ] Prompt injection prevention?
- [ ] Hallucination boundaries?
- [ ] Fallback at LLM downtime?

---

## Использование

Для каждого dimension:
1. Прочитай вопросы
2. Найди соответствующий код
3. Verify каждый checkmark
4. Failed checks → findings с severity

Среднее время full audit одной фазы — 2-4 часа. Не сокращай.
