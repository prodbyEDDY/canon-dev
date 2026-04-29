# Audit output format — детализированный шаблон

> Финальный отчёт audit'а должен следовать этому формату. Структура — для удобства координатора, который будет принимать решения по findings.

---

## Шаблон

```markdown
# Audit Report — {Phase {N} | Program-wide}

**Date:** {YYYY-MM-DD}
**Auditor:** Fresh agent in clean session
**Scope:** {Phase 4D.3 | Phases 4D.1-4D.3 | Full Phase 4 | Whole codebase}
**Effort:** {hours spent}

---

## Executive summary

- **Total findings:** {X}
  - 🔴 Critical: {A}
  - 🟡 Major: {B}
  - 🟢 Minor: {C}
  - 💡 Suggestions: {D}
- **Acknowledged debt confirmed:** {N}/{N total в §10.4}
- **Spec deviations review:** {acceptable / needs attention}
- **Brand compliance:** {clean / N violations}

**Verdict:** {one of}
- ✅ **Canonically clean.** Phase ready for next stage.
- ⚠️ **Issues to address — non-blocking.** Major findings should be in next phase scope.
- 🔴 **Critical issues found — hotfix required.** Stop and address before continuing.

---

## 🔴 Critical findings

> Block production. Fix перед next phase.

### C-01. {Concise описание}

- **Где:** `src/path/to/file.ts:line-line`
- **Что:** {точное описание бага}
- **Канон-reference:** {§X.Y of Canon_Y.md} — {цитата если применимо}
- **Симптом:** {что произойдёт в production если не fix}
- **Reproduce:** {как воспроизвести}
- **Fix-направление:** {short recommendation, не код}
- **Effort estimate:** {S | M | L} ({hours})

### C-02. ...

---

## 🟡 Major findings

> Серьёзные недосмотры. Желательно address до конца текущей фазы.

### M-01. {Название}
- **Где:** ...
- **Что:** ...
- **Канон-reference:** ...
- **Trade-off:** {почему это серьёзно но не critical}
- **Fix-направление:** ...

### M-02. ...

---

## 🟢 Minor findings

> Cosmetic / quality-of-code. Закрывать постепенно.

### m-01. ...

---

## 💡 Suggestions

> Improvements, не gaps. Backlog.

### S-01. ...

---

## Cross-canon inconsistencies

> Расхождения между модулями.

### X-01. {Название}
- **Между:** Canon_A.md (или Module A) и Canon_B.md (или Module B)
- **Расхождение:** ...
- **Severity:** {Critical | Major | Minor}
- **Recommendation:** ...

---

## Spec-vs-implementation расхождения

> Что обещано в phase prompt vs что реализовано.

### V-01. Phase {N} promised X, delivered Y

- **Промпт §{X.Y}:** {expected behavior}
- **Реализовано:** {actual behavior}
- **Acknowledged?:** {Yes — see §10.4 Ф{code} | No — undocumented deviation}
- **Severity:** {classification}
- **Note:** {если acknowledged — verify условия закрытия. Если нет — recommend документировать или fix}

### V-02. ...

---

## Acknowledged debt confirmation

> Verification что existing acknowledged debts реально присутствуют как описано.

| Code | Description (§10.4) | Code presence | Status |
|---|---|---|---|
| Ф27 | posting_date fix | TODO в `operations.ts:236` ✓ | ✅ Confirmed |
| Ф28 | formatPgError helper | Implemented `api-helpers.ts:45` | ✅ Confirmed |
| Ф29 | Dashboard canon-alignment | TODO в comment, deferred | ✅ Confirmed |
| Ф{N} | {description} | Couldn't find TODO | ⚠️ Doc-vs-code drift |
| Ф{M} | {description} | Implemented differently | ⚠️ Не как описано в §10.4 |

---

## New acknowledged debt suggested

> Findings которые worth converting in acknowledged debt rather than immediate fix.

### ND-01. {Название}
- **Description:** ...
- **Why this is debt не fix:** ...
- **Suggested closing condition:** {when this should be addressed}
- **Suggested code:** Ф{N}

---

## Coverage check

What was audited:

- [x] All {N} new files in Phase {X} read fully
- [x] Schema audit (D7): all CHECK constraints, indexes verified
- [x] API audit (D8): all {M} endpoints checked for session+orgId+Zod+pagination
- [x] Calculation audit (D9): all formulas vs canon, examples §Y verified
- [x] Antidubli audit (D2): idempotency keys, partial indexes
- [x] Atomic transactions (D3): all critical-flow wrapped
- [x] Brand grep (D11): 5 categories
- [x] Cross-canon checks (4 groups)
- [x] Spec-vs-implementation для phases {N}-{M}

What was NOT audited:

- [ ] Performance benchmarks — couldn't measure без production data
- [ ] Concurrency edge cases — couldn't reproduce без load test infrastructure
- [ ] {Other gaps with reasons}

---

## Recommendations to coordinator

In priority order:

### Immediate (this week)

1. {Critical action 1} — see C-01
2. {Critical action 2} — see C-02

### Next phase (within current sprint)

3. {Major fix 1} — see M-01
4. {Major fix 2} — see M-02

### Schedule (over next 2-3 phases)

5. {Major fix to backlog}
6. {Documentation update}

### Backlog

7. {Suggestion 1}

### Acknowledged debt actions

- Ф{N} status update needed (drift between description and code)
- Ф{M} ready to close (pre-conditions met)

---

## Methodology notes

- Audit method: dimension-by-dimension (16 dimensions per `references/dimension-checklist.md`)
- Time spent: ~{X} hours
- Files read: ~{N}
- Files spot-checked (not full read): ~{M}
- Tools used: grep, git log, manual code-review, mental simulation of T-scenarios

If specific finding requires deeper dive — happy to do focused audit на ту область.

---
```

---

## Заметки

### Severity classification (refresher)

- **🔴 Critical:** production risk / monetary correctness / security / cross-tenant leak. Stops everything.
- **🟡 Major:** lifecycle / integration / scale gap. Address within phase.
- **🟢 Minor:** documentation / style / non-blocking quality. Cleanup over time.
- **💡 Suggestion:** improvement, not gap. Backlog.

### Honest coverage

Если что-то не audited — явно отметь и почему. Маскировать «всё проверил» когда не всё — токсично. Координатор может запросить focused audit.

### Объём

Среднее время full-audit одной фазы — 2-4 часа.
Объём отчёта — 200-500 строк markdown. Не сжимай ради краткости. Концентрат теряет ценность.

### Honest verdict

В executive summary — **один из 3 verdicts**, не середина:
- ✅ clean
- ⚠️ non-blocking but address
- 🔴 critical, hotfix

Если сомневаешься между ⚠️ и 🔴 — выбери 🔴 (safe side).
Если сомневаешься между ✅ и ⚠️ — выбери ⚠️ (transparent side).
