---
name: canon-review
description: Use when canon documents already exist (from braindump-to-canon or other source) and need enterprise-grade audit for architectural holes, lifecycle gaps, missed integration concerns, schema-level issues, and data-quality blind spots. Outputs structured findings with severity classification. Read-only — does not modify canons, only flags what needs fixing.
---

# canon-review

🚀 Стадия 2 в pipeline canon-dev: **аудит канонов на полноту**

---

## Когда использовать

- После `braindump-to-canon` — обязательная проверка перед началом разработки
- Существующие каноны нужно проверить на enterprise-readiness
- Подозрение что архитектура «дырявая» — найти где
- Перед production-launch — финальный аудит

## Когда НЕ использовать

- Каноны ещё не написаны → сначала `braindump-to-canon`
- Нужно проверить **код**, а не каноны → `canon-verify`
- Нужен дизайн-аудит → `design-canon` имеет встроенный review

---

## Философия скилла

`canon-review` — это **критический читатель**. Он находит то что автор канона **не подумал** или **посчитал очевидным**. 

Большинство архитектурных багов рождается не из плохих решений, а из **отсутствия решений** — пропущен edge case, забыт integration point, не описан error path. Этот скилл систематически проходит по матрице измерений и подсвечивает пробелы.

Принцип: **жесткая, но конструктивная критика**. Не «у вас плохо», а «здесь пропуск, вот что упущено, вот варианты как закрыть». Уважение к работе автора, но без compromiseа на полноту.

---

## Структура процесса

```
┌─────────────────────────────────────────────────────────────┐
│  Phase 1 — Inventory (что есть в каноне-наборе)             │
└─────────────────────────────────────────────────────────────┘
                             ↓
┌─────────────────────────────────────────────────────────────┐
│  Phase 2 — Dimension audit (15+ измерений)                  │
└─────────────────────────────────────────────────────────────┘
                             ↓
┌─────────────────────────────────────────────────────────────┐
│  Phase 3 — Cross-canon consistency check                    │
└─────────────────────────────────────────────────────────────┘
                             ↓
┌─────────────────────────────────────────────────────────────┐
│  Phase 4 — Findings report (structured)                     │
└─────────────────────────────────────────────────────────────┘
                             ↓
┌─────────────────────────────────────────────────────────────┐
│  Phase 5 — Hand-off (recommendations к canon-update)        │
└─────────────────────────────────────────────────────────────┘
```

---

## Phase 1 — Inventory

Загрузи весь canon-set: `docs/КАНОНЫ/*.md`, `docs/ТЗ/*.md`, `docs/STACK.md`, `OPEN-QUESTIONS.md`, любая другая archeology.

Сделай таблицу что есть:

```
## Inventory of canon-set (review of {date})

| File | Type | Module | Status |
|---|---|---|---|
| 00-Главный-канон.md | Cross-cutting | — | exists |
| Канон_Система.md | Per-module | Система | exists |
| Канон_Финансы.md | Per-module | Финансы | exists |
| ТЗ_Финансы.md | TZ | Финансы | NOT FOUND |
| Канон_Аналитика.md | Per-module | Аналитика | exists |
| ... | | | |

**Gaps в inventory:**
- Модуль X упомянут в главном каноне, но нет Канон_X.md
- ТЗ для модуля Y отсутствует
- STACK.md не содержит решения по {category}
```

---

## Phase 2 — Dimension audit

Это сердце скилла. Пройди по каждому измерению и задай конкретные вопросы для текущего canon-set'а.

См. `references/audit-dimensions.md` для полной матрицы. Сводно по группам:

### 🔴 Critical dimensions (must-have)

#### D1. Lifecycle completeness

Для каждой сущности с состояниями:
- Все статусы перечислены и определены?
- Все переходы между статусами указаны?
- Запрещённые переходы явно зафиксированы?
- Что происходит при попытке нелегального перехода?
- Есть ли «closure»-состояние? Как переоткрывается?
- Cancellation — soft (status='cancelled') или hard (физическое удаление)?

#### D2. Antidubli и идемпотентность

- Где данные могут поступать из 2+ источников?
- Какие защиты от двойного импорта?
- Где есть idempotency keys, какой формат?
- Race-conditions — где, какая защита?

#### D3. Источники истины

- Каждый тип данных имеет однозначный источник?
- Если источников несколько — кто primary, кто replica?
- Read-only views явно отделены от писательных?
- Аналитика читает, не пишет?

#### D4. Multi-tenancy

- Все сущности имеют `organization_id` (или эквивалент)?
- Все queries фильтруются по orgId?
- JOIN'ы между таблицами безопасны (один orgId через всю цепочку)?
- Нет ли «глобальных» данных без owner-org? (если есть — обоснованно?)

#### D5. Деньги

- Все суммы в integer-копейках/центах?
- Валюта определена однозначно для каждой суммы?
- Округление формализовано?
- Двойная запись (если применимо) — обеспечена?
- Журнал движения средств — append-only?

### 🟡 Major dimensions

#### D6. Audit logging

- Какие события логируются обязательно — определено?
- Структура audit-log универсальна?
- Append-only enforced?
- Retention policy определена?
- Кто видит журнал — роль?

#### D7. Authorization model

- Роли определены и описаны?
- Permissions per action — список явный?
- Critical actions требуют доп. подтверждения?
- Cross-org permission leak невозможен?

#### D8. Versioning справочников и правил

- Если есть «ставки», «нормативы», «правила» — версионированы?
- Versioning по дате или по году?
- Как переходит между версиями?
- Старые периоды считаются по старой версии — или пересчитываются?

#### D9. Error handling и user feedback

- Все error-cases в lifecycle обработаны?
- User-facing error messages определены?
- Retryable vs terminal errors различаются?
- Что показываем когда внешняя система недоступна?

#### D10. Cross-module integration

- Каждый канон явно описывает что отдаёт другим?
- Каждый канон явно описывает что берёт у других?
- Контракты на стыках — формализованы?
- Что при failure на стыке?

### 🟢 Standard dimensions

#### D11. Data quality / unavailability

- Когда данные недоступны — как показываем?
- Partial data states — обработаны?
- «Honest unavailable» вместо fake-data?

#### D12. Migrations и backward compatibility

- Schema-evolution стратегия описана?
- Backfill для существующих данных продуман?
- Breaking changes в API — как обрабатываем?

#### D13. Performance и scale

- Тяжёлые queries идентифицированы?
- Indexes для критичных queries описаны?
- N+1 ловушки предотвращены?
- Pagination на list-endpoints?

#### D14. Testing и acceptance

- Тест-кейсы покрывают happy path?
- Edge cases — отдельно?
- Регрессионные сценарии — описаны?
- Acceptance criteria measurable?

#### D15. Documentation hygiene

- Brand-voice соблюдается?
- Структура канонов одинакова?
- Cross-links между канонами есть?
- Open-questions явно вынесены?

---

## Phase 3 — Cross-canon consistency

После проверки каждого канона отдельно — кросс-проверка:

### Терминология

- Одна и та же сущность называется одинаково везде?
- Например: «контрагент» vs «партнёр» vs «counterparty» — какое-то одно
- Если есть алиасы — задокументированы в глоссарии

### Lifecycle alignment

- Если модуль A создаёт сущность которую модуль B изменяет — статусы согласованы?
- Например: A создаёт `obligation`, B создаёт `payment` который меняет `obligation.paid` — это явно описано в обоих канонах?

### Антидубль на стыках

- Если данные могут поступить из A и из B одновременно — антидубль описан в **обоих** канонах с одинаковой логикой?
- Если A передаёт в B — кто отвечает за идемпотентность: A (детерминирует key), B (проверяет)?

### Источники истины — без дубликата

- Один тип данных не может быть «источником истины» в двух местах
- Если в каноне A написано «X — источник для Y», а в каноне B «X — производный из Z» — конфликт

### Стек consistency

- Все каноны опираются на тот же стек (см. `STACK.md`)?
- Нет ссылок на технологии не входящие в утверждённый стек?

---

## Phase 4 — Findings report

Сгенерируй отчёт в стандартизированном формате:

```markdown
# Canon Review Report — {date}

## Summary
- Inventory: N canons + M ТЗ
- Total findings: X (Critical: A, Major: B, Minor: C, Suggestions: D)
- Verdict: [✅ Enterprise-ready] | [⚠️ Major gaps to address before phase 1] | [🔴 Critical issues — block development]

## 🔴 Critical findings (block production)

### CR-01. {Название}
- **Где:** {Канон_X.md §N}
- **Что:** {точное описание пробела}
- **Симптом:** {что пойдёт не так если не исправить}
- **Фикс-направление:** {рекомендация — короткое предложение}
- **Эффорт:** {S/M/L} — оценка времени на доработку

### CR-02. ...

## 🟡 Major findings

### M-01. {Название}
[аналогичная структура]

## 🟢 Minor findings

### m-01. {Название}
[аналогичная структура]

## 💡 Suggestions (improvements, not gaps)

### S-01. {Название}
[аналогичная структура]

## Cross-canon inconsistencies

### X-01. {Название}
- **Между:** {Канон_A.md} и {Канон_B.md}
- **Расхождение:** ...
- **Рекомендация:** ...

## Open questions confirmed (already in OPEN-QUESTIONS.md)

| Q-id | Status | Comment |
|---|---|---|
| Q-01 | Still open | Не нашёл в каноне решения, всё ещё blocking для phase X |

## Recommendations

В порядке приоритета:
1. Закрыть Critical findings (см. список) — обязательно до начала phase 1
2. Адресовать Major findings — желательно до phase 1, иначе превратятся в hotfix'ы
3. Minor — закрывать постепенно по мере touch'а соответствующих канонов
4. Suggestions — полезные улучшения, по time/budget

## Coverage check

- [x] Все N канонов прочитаны полностью
- [x] Все 15 dimensions проверены
- [x] Cross-canon consistency проверен по 4 группам
- [ ] Не проверено: {что и почему}

```

---

## Phase 5 — Hand-off

В конце скажи пользователю:

```
Аудит завершён. Файл findings.md лежит в docs/CANON-REVIEW-{date}.md.

Резюме:
- {N} Critical findings — блокируют разработку
- {M} Major findings — приоритетные доработки
- {K} Minor findings — постепенно
- {L} Suggestions — улучшения

Следующий шаг:
- Если Critical=0 и Major≤2: можем переходить к design-canon или canon-roadmap
- Если Critical>0 или Major>5: вернуться к canon-update сессии (это другая задача — не сам review)
```

---

## Anti-patterns (агенту)

❌ **«Каноны хорошие, не нашёл проблем»** — почти невозможно для свежего canon-set. Если ничего не нашёл — копай глубже. 99% канонов имеют upgrade-возможности.

❌ **«Все findings — Critical»** — теряет смысл severity. Используй классификацию строго:
- Critical = система не будет работать или сломается в production
- Major = серьёзный недосмотр, приведёт к рефакторингу через 3 месяца
- Minor = improvement в качестве документации
- Suggestion = nice-to-have

❌ **«Я нашёл, теперь сам поправлю»** — нет, это другой скилл/задача. `canon-review` только находит, не правит.

❌ **«Игнорирую открытые вопросы из OPEN-QUESTIONS.md»** — нет. Включи их статус в отчёт. Open вопросы это часть canon-set'а.

---

## Critical principles

1. **Severity дисциплинирована.** Не раздавай Critical налево-направо.
2. **Findings конкретны.** «Эта секция расплывчата» — плохо. «§3.2 не отвечает на вопрос: что происходит при concurrent update X» — хорошо.
3. **Предлагай направление фикса, не пиши фикс.** Это работа `braindump-to-canon` или manual canon-update.
4. **Не суди стиль.** Если канон следует BRAND-VOICE.md — стиль OK. Замечание про язык можно делать только если он реально мешает пониманию.
5. **Уважай контекст.** Может быть выбор отказаться от фичи (out-of-scope) — это валидно. Не репорти как пропуск то что явно out-of-scope.

---

## Templates и references

- `references/audit-dimensions.md` — расширенная матрица 15+ измерений с конкретными вопросами
- `references/completeness-checklist.md` — чек-лист на каждое измерение
- `references/findings-classification.md` — как классифицировать severity
- `references/cross-canon-checks.md` — конкретные проверки на consistency между канонами

---

## Финальная заметка

Найденные сейчас пробелы стоят в 100× дешевле чем те же пробелы найденные в production через 6 месяцев. Один час потраченный на canon-review = месяцы сэкономленных hotfix'ов.

Не торопись. Канон-набор — это конституция продукта. Хороший audit — медленный.
