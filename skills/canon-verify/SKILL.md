---
name: canon-verify
description: Use after a phase has been implemented (merged or pre-merge) and needs comprehensive audit against canons and ТЗ. Read-only auditor — finds discrepancies between code and canon, classifies severity (Critical / Major / Minor / Acknowledged debt), produces structured findings report. Does not modify code. For a fresh agent in a clean session — independence от implementor critical.
---

# canon-verify

🚀 Стадия 6 в pipeline canon-dev: **сверка реализации с каноном**

> Этот скилл — для **аудитора в отдельной сессии**. Принципиально независимый от того кто реализовывал. Read-only — только проверяет и отчитывается.

---

## Когда использовать

- После merge фазы — обязательная сверка с каноном
- Перед production-launch — финальный audit
- Подозрение на regression в существующей системе
- Comprehensive review всей программы периодически (раз в N фаз)

## Когда НЕ использовать

- Нужно правит код по результатам audit'а → это другая роль (реализатор + canon-execute)
- Каноны сами под подозрением → `canon-review`
- Просто нужно code-review одной фичи → стандартное PR review, не нужен canon-verify

---

## Философия скилла

`canon-verify` — это **дисциплинированный, независимый, read-only аудитор**. 

Принцип: «независимость глаз». Тот же агент который реализовывал — не должен делать audit. У него blind-spot: то что он считает очевидным — может быть багом. Свежий глаз находит.

Принцип: «severity дисциплинирована». Не всё Critical. Корректная классификация — половина value report'а. Critical = блокирующий. Major = серьёзный. Minor = косметический. Acknowledged debt = уже известный, не репортить как новый.

Принцип: «findings конкретны». Не «эта секция расплывчата» — а «§3.2 не отвечает на: что происходит при concurrent X».

Принцип: «не правит, только репортит». Audit — это input для координатора. Координатор решает: hotfix, ROADMAP §10.4, ignore. Аудитор — собирает данные, не принимает решения.

---

## Структура процесса

```
Phase 1 — Bootstrap (read CLAUDE.md, canons, ROADMAP, recent commits)
   ↓
Phase 2 — Inventory (что было сделано в фазе/программе)
   ↓
Phase 3 — Dimension audit (16+ измерений, каждое — checkmark)
   ↓
Phase 4 — Cross-cutting checks (consistency между modules)
   ↓
Phase 5 — Spec-vs-implementation review (что обещано vs сделано)
   ↓
Phase 6 — Acknowledged debt confirmation (что уже известно)
   ↓
Phase 7 — Findings report (structured)
   ↓
Phase 8 — Hand-off (recommendations координатору)
```

---

## Phase 1 — Bootstrap

Прочитай **обязательно**:
- `CLAUDE.md` — стек, конвенции
- `docs/ROADMAP.md` (особенно §10, §10.4, §11)
- `docs/CURRENT-STATE.md`
- Все каноны в `docs/КАНОНЫ/` (или эквивалент)
- Все ТЗ в `docs/ТЗ/`
- `docs/STACK.md`
- `OPEN-QUESTIONS.md`

**Если фаза-specific audit:** дополнительно прочитай:
- Phase prompt (`docs/prompts/PHASE-{N}-{name}.md`)
- Progress journal (`docs/progress/PHASE-{N}-{name}.md`)
- PR description (через `gh pr view {N}`)

Цель: понять **что должно быть** (по canon + spec) до того как смотреть **что есть** (код).

---

## Phase 2 — Inventory

Что было сделано в audit-scope:

```bash
# Если audit одной фазы:
git log --oneline {since-tag-or-commit}..HEAD

# Что изменено:
git diff {base}..HEAD --stat

# Какие новые файлы:
git diff {base}..HEAD --name-only --diff-filter=A
```

Сделай ментальную таблицу:
- Schema-changes: какие таблицы, колонки
- Logic: какие новые модули в `src/lib/`
- API: какие новые endpoints
- UI: какие новые pages/components
- Cross-module: какие интеграции

Это база для checking.

---

## Phase 3 — Dimension audit

Это сердце скилла. Для каждого измерения — проходишь по checkmark'ам и собираешь findings. Полный список измерений — `references/dimension-checklist.md`. Сводно:

### 🔴 Critical dimensions

#### D1. Lifecycle correctness
- Все lifecycle transitions в коде совпадают с canon-описанием?
- Запрещённые transitions блокированы (на DB-level или service-level)?
- Race conditions защищены (advisory locks, optimistic concurrency)?

#### D2. Antidubli
- Idempotency keys присутствуют где должны?
- Partial unique indexes на ключевых полях?
- Re-import / retry не дуплицирует?

#### D3. Atomic transactions
- Critical-flow обёрнут в `db.transaction`?
- Рефакторинг существующих helpers поддерживает optional `tx` parameter (когда необходимо)?
- Rollback тестировался mentально?

#### D4. Multi-tenancy
- Все queries фильтруются по orgId?
- Cross-org leak невозможен через ID-guessing?
- JOINs безопасны (orgId через цепочку)?

#### D5. Money correctness
- Все суммы в integer?
- Округление consistent?
- Деление на 0 в delta-расчётах обработано (no Infinity / NaN)?

#### D6. Posting / accounting invariants
- ОПиУ-flag правильный для каждого типа operation?
- ОДДС-flag правильный?
- Antidubli с другими модулями (например, ФОТ) работает?
- При проведении — `posting_date` обязательно установлен?

### 🟡 Major dimensions

#### D7. Schema integrity
- CHECK constraints на месте per ТЗ?
- Indexes для frequent queries?
- Audit-log tables append-only?
- Foreign keys или namespace-restricted?

#### D8. API patterns (CLAUDE.md compliance)
- Session check + orgId в каждом endpoint?
- Zod validation?
- Pagination на list-endpoints?
- Try/catch с правильными HTTP-кодами?
- logEvent для critical mutations?

#### D9. Calculation accuracy (если расчётный модуль)
- Формулы соответствуют canon §X.Y?
- Учебные примеры canon §Y дают ожидаемые числа?
- Edge cases (zero base, negative input) обработаны корректно?

#### D10. Cross-module integration
- Hooks встроены правильно?
- Contracts между модулями соблюдены?
- Antidubli на стыках работает?

### 🟢 Standard dimensions

#### D11. Brand compliance
- No font-bold/font-semibold?
- No off-brand colors?
- No эмодзи в production UI?
- Tokens used everywhere (no hardcoded hex)?

#### D12. UI correctness
- Loading states (Skeleton)?
- Empty states?
- Error states?
- Responsive (mobile/tablet/desktop)?
- A11y (focus rings, alt-tags)?

#### D13. Documentation
- Inline comments reference canon-секции?
- Architectural deviations documented?
- README в новых модулях (где нетривиально)?

#### D14. Tests / acceptance
- T-сценарии из промпта прошли?
- Учебные примеры canon §Y verified?

#### D15. Performance hot spots
- N+1 queries предотвращены?
- Pagination используется на large lists?
- Heavy queries индексированы?

#### D16. Security
- No cross-tenant leak paths?
- Sensitive data not in logs?
- Input validation everywhere (Zod schemas)?

---

## Phase 4 — Cross-cutting checks

После проверки каждого dimension'а — кросс-checks:

### Consistency между модулями

- Терминология одинакова?
- Lifecycle alignment между связанными модулями?
- Источники истины без duplicates?

### Recently added vs existing patterns

- Новый код использует existing helpers (не дублирует)?
- Новый module follows конвенции (naming, folder structure)?
- Common patterns (e.g. lifecycle state machine) реализованы единообразно?

### Build / lint hygiene

```bash
pnpm exec tsc --noEmit
pnpm lint
pnpm db:generate
```

Если есть warnings/errors — finding.

---

## Phase 5 — Spec-vs-implementation review

Для каждой фазы в audit-scope:

1. Прочитай phase prompt
2. Прочитай PR description
3. Прочитай progress journal

Сравни **что обещано** vs **что сделано**:

- Все promised deliverables реализованы?
- Все out-of-scope items действительно НЕ сделаны?
- Все T-сценарии из промпта pass'ятся?
- Acknowledged deviations явно зафиксированы?
- Architectural decisions имеют inline-комментарии с rationale?

Любое расхождение — finding (severity зависит от того насколько material).

---

## Phase 6 — Acknowledged debt confirmation

Прочитай ROADMAP §10.4 (acknowledged tech debt). Для каждой записи:

- Подтверждается ли она в коде? (есть TODO-comment, есть workaround там где описано?)
- Условия закрытия еще актуальны? Или debt уже should be closed?
- Появились ли новые debts которые не зафиксированы в §10.4?

**Важно:** acknowledged debts — **НЕ findings**. Они уже известны. Просто verify что:
- Действительно реализованы как описано
- Не превратились в hidden debts (потеряли TODO)
- Готовы к closing если pre-condition met

---

## Phase 7 — Findings report

Сгенерируй финальный отчёт. Структура (пример):

```markdown
# Audit Report — Phase {N} / {Program}
**Date:** {YYYY-MM-DD}
**Auditor:** {fresh agent in clean session}
**Scope:** {Phase N | Phase 1-N | All current state}

---

## Summary
- Total findings: {X} (🔴 Critical: {A}, 🟡 Major: {B}, 🟢 Minor: {C}, 💡 Suggestions: {D})
- Acknowledged debt confirmed: {N}/{N} entries в §10.4
- Verdict: 
  - ✅ Canonically clean — phase ready for next
  - ⚠️ Issues to address — non-blocking, but should be in next phase
  - 🔴 Critical issues found — hotfix required before continuing

## 🔴 Critical findings

### C-01. {Название}
- **Где:** `{file:line}`
- **Что:** {точное описание}
- **Canon-reference:** §{X.Y} ({цитата если применимо})
- **Симптом:** {что пойдёт не так в production}
- **Fix-направление:** {short rec}

### C-02. ...

## 🟡 Major findings

### M-01. {Название}
[аналогично]

## 🟢 Minor findings

### m-01. ...

## 💡 Suggestions

### S-01. ...

## Cross-canon inconsistencies (если есть)

### X-01. {Название}
- **Между:** канонами / модулями
- **Расхождение:** ...

## Spec-vs-implementation расхождения

### V-01. Phase {N} promised X, delivered Y
- **Промпт §{X}:** {expected}
- **Реализовано:** {actual}
- **Acknowledged?:** да / нет
- **Severity:** {classification}

## Acknowledged debt confirmation

| Code | Status | Comment |
|---|---|---|
| Ф{N} | ✅ Confirmed in code (`{file:line}`) | TODO present + canon-ref |
| Ф{M} | ⚠️ Implemented differently than в §10.4 description | {detail} |
| Ф{K} | ❓ Couldn't verify | {reason} |

## Coverage check

What was audited:
- [x] All {N} new files in scope read fully
- [x] All {M} dimensions checked
- [x] Cross-canon checks (4 groups) done
- [x] Spec-vs-implementation для phase {N}
- [x] Brand grep (5 categories)
- [ ] Couldn't audit: {что и почему}

## Recommendations to coordinator

In priority order:
1. {Critical action 1}
2. {Critical action 2}
3. {Major fix to schedule}
4. {Minor — when convenient}
5. {Suggestion — backlog}
```

---

## Phase 8 — Hand-off

Финальная коммуникация координатору:

```
Audit completed.

Summary:
- {X} Critical (block production) — see Section "Critical findings"
- {Y} Major (address before next phase) — see "Major"
- {Z} Minor (cleanup over time) — see "Minor"
- {N} acknowledged debts confirmed

Recommendation:
{1 параграф}

If 🔴 > 0: stop and hotfix.
If 🟡 > 5: address current Phase Major findings before next phase.
If 🟡 ≤ 5 и 🟢 only: ok to proceed to next phase, schedule Major fixes in next 2-3 phases.
```

---

## Anti-patterns

❌ **«Каноны хорошие, ничего не нашёл»** — почти невозможно для свежего audit'а. Если ничего не нашёл — копай глубже. 99% audit'ов имеют upgrade-возможности.

❌ **«Всё Critical»** — теряет смысл severity.

❌ **«Я нашёл, теперь сам поправлю»** — нет, это другая роль. Audit only finds, не fixes.

❌ **«Acknowledged debt — это finding»** — нет, это уже известно. Если в §10.4 запись есть — не репорти как новое. Verify confirmation.

❌ **Subjective findings** — «мне не нравится naming». Не finding если не мешает correctness. Может быть Suggestion в крайнем случае.

❌ **Skip dimensions** — «эти 3 dimensions кажутся не релевантными». Все 16 измерений проходи. Если действительно не применимо — отметь в "couldn't audit" с reason.

---

## Critical principles

1. **Read-only.** Не редактируешь код, только репортишь.
2. **Severity disciplined.** Используй criteria, не feelings.
3. **Findings конкретны.** Где, что, почему, направление fix'а.
4. **Не acknowledged-debt-новости.** Verify, don't re-discover.
5. **Independence от implementor.** Если ты ИИ-агент — это тебе помогает (нет эмоциональной привязки к коду).
6. **Honest coverage.** Что не audited — явно отметить, не маскировать.

---

## Templates и references

- `references/dimension-checklist.md` — расширенная матрица 16+ измерений
- `references/audit-output-format.md` — детализированный шаблон отчёта
- `references/severity-calibration.md` — калибровка severity с примерами

---

## Финальная заметка

Хороший audit — invisible value. Если findings «тривиальные» — это **отлично**, программа устойчива. Если findings critical — лучше найти их в audit чем в production.

Не ленись. Audit — это страховка для координатора. Хорошая страховка стоит часов работы. Плохая — стоит дней rollback'а.
