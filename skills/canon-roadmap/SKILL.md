---
name: canon-roadmap
description: Use when canons and design system are ready and you need to decompose them into a phased implementation plan. Generates ROADMAP.md (with live-track, tech debt registry, decisions journal), CURRENT-STATE snapshot, and a self-contained prompt for the first phase. On subsequent invocations — checks existing ROADMAP, finds next un-checked phase, generates the next prompt continuing from current state.
---

# canon-roadmap

🚀 Стадия 4 в pipeline canon-dev: **каноны → план → промпт фазы**

---

## Когда использовать

- Каноны готовы (проверены через `canon-review`), нужен план реализации с нуля
- Дизайн-система готова (или явно решено что её нет)
- Уже идёт разработка — нужен промпт следующей фазы по существующему ROADMAP
- Закончилась фаза — нужно обновить snapshot и промпт следующей фазы

## Когда НЕ использовать

- Каноны ещё не проверены → сначала `canon-review`
- Уже есть промпт фазы и нужен агент-реализатор → `canon-execute`
- Закончилась фаза и нужен audit → `canon-verify`

---

## Философия скилла

`canon-roadmap` — это **роль координатора**. Не реализатор, не аудитор. Он:

1. Видит **граф зависимостей** между модулями
2. Определяет **критический путь**
3. Декомпозирует на фазы с ясными входами/выходами
4. Пишет **самодостаточные промпты** для агентов-реализаторов
5. Поддерживает **живой ROADMAP** — после каждой фазы обновляет snapshot
6. Ведёт **журнал решений** (когда что и почему было решено)
7. Регистрирует **acknowledged debt** при отклонениях

Принцип: **промпт самодостаточный**. Агент-реализатор в чистой сессии должен мочь выполнить промпт без обращения к чату или другим документам.

---

## Два режима работы

### Mode A — First run (создание ROADMAP'а с нуля)

```
1. Read all canons and design system
2. Build dependency graph between modules
3. Identify critical path and risk hotspots
4. Decompose into phases with clear deliverables
5. Generate ROADMAP.md (with §10 live-track, §10.4 debt, §11 journal)
6. Generate CURRENT-STATE.md (snapshot of what exists)
7. Generate first phase prompt (self-contained)
```

### Mode B — Subsequent run (continuation)

```
1. Read existing ROADMAP.md and CURRENT-STATE.md
2. Read latest progress journals
3. Identify next un-checked phase from §10
4. Read previous phase's PR + deviations
5. Update CURRENT-STATE.md with new state
6. Update §10 (mark previous phase done)
7. Add §11 journal entry for previous phase
8. Generate next phase prompt
```

Скилл **сам определяет** в каком режиме запущен — проверяя наличие `ROADMAP.md` в директории документации.

---

## Mode A — Первый запуск

### Step 1. Inventory canon-set

Прочитай:
- Все файлы в `docs/КАНОНЫ/`
- Все файлы в `docs/ТЗ/`
- `STACK.md`
- `OPEN-QUESTIONS.md`
- Дизайн-система (если есть): `docs/DESIGN-SYSTEM/`

Сделай ментальную таблицу: какие модули есть, какие у них зависимости.

### Step 2. Build dependency graph

Для каждого модуля определи:
- **Hard dependencies:** модули которые **должны** существовать раньше (например, Аналитика читает Финансы — Финансы first)
- **Soft dependencies:** модули которые **желательно** до этого (UI-зависимости)
- **Cross-cutting:** модули которые трогают всех (Auth, Audit, Settings — обычно first wave)

Граф визуализируй текстом:

```
Модуль                    Hard deps               Soft deps
────────────────────────────────────────────────────────────
01-Система                —                       —
02-{Domain core}          01                      —
03-{Cross-domain entity}  01                      02
04-{Domain features 1}    01, 02, 03              —
05-{Domain features 2}    01, 02, 03, 04          —
0N-Аналитика              all above               —
```

### Step 3. Critical path

Найди **критический путь** — самую длинную цепочку зависимостей. Это определяет минимальное время до production.

Также пометь **risk hotspots** — модули где:
- Большой scope (>1k LoC ожидается)
- Денежная логика
- Множество edge cases
- Мало конвенций (придётся изобретать)
- Внешние интеграции

### Step 4. Decompose into phases

Каждая фаза = атомарная единица работы которая:
- Может быть реализована **одним агентом** в **одной сессии** (≤ ~1500 LoC промпт + ~3000 LoC код)
- Имеет ясные входы (что должно существовать)
- Имеет ясные выходы (что появится)
- Может быть merged без зависимости от будущих фаз

Если модуль большой — разбей на sub-фазы (`4D.1`, `4D.2`, `4D.3` стиль). Логика разбивки:
- `.1` — foundation: schema + базовые сущности + onboarding-интеграция
- `.2` — engine: расчёты / lifecycle / API
- `.3` — final: интеграции / cross-module / финальный UI

Пример для среднего продукта (5-7 модулей):

```
Фаза 0 — Подготовка инфраструктуры (Next.js setup, auth, multi-tenancy, deployment)
Фаза 1 — Система (auth, users, roles, settings, audit-log)
Фаза 2 — {Cross-domain entity} (например, контрагенты или клиенты)
Фаза 3 — {Documents / Files} (если применимо)
Фаза 4A — {Domain core entity} (foundation)
Фаза 4B — {Domain core operations} (engine)
Фаза 4C — {Domain reports} (analytics on top)
Фаза 5 — {Secondary domain}
...
Фаза N — Аналитика (обзорный слой над всеми)
```

### Step 5. Generate ROADMAP.md

Используй шаблон `templates/roadmap-template.md`. Структура:

```markdown
# ROADMAP

## 0. Executive summary
## 1. Текущее состояние
## 2. Целевое состояние
## 3. Архитектурные принципы (сквозные)
## 4. Дерево зависимостей и критический путь
## 5. Фазы реализации (детальное описание каждой)
## 10. Live-track (чек-лист прогресса) ← обновляется после каждой фазы
## 10.4. Acknowledged tech debt ← регистрируется по ходу
## 11. Журнал решений ← обновляется после каждой фазы
```

Live-track:
```markdown
## 10. Live-track

- [ ] Фаза 0 — Подготовка
- [ ] Фаза 1 — Система
- [ ] Фаза 2 — Контрагенты
...
```

### Step 6. CURRENT-STATE.md

Snapshot текущего состояния. На первом run — почти пустой:

```markdown
# Current State — {date}

## Code
- Repository created: {date}
- Stack: {summary from STACK.md}
- Migrations applied: 0
- Modules implemented: 0

## Documentation
- Canons: {list}
- ТЗ: {list}
- Design system: {ready / not yet}
- Open questions: {N count, see OPEN-QUESTIONS.md}

## Next phase
Phase 0 — Подготовка инфраструктуры. См. `docs/prompts/PHASE-0-INFRASTRUCTURE.md`.
```

### Step 7. First phase prompt

Используй `templates/phase-prompt-template.md`. Промпт должен быть:
- Самодостаточный
- ~500-1000 строк
- С явным scope, out-of-scope, pre-merge checklist
- С брендовыми правилами (цитата из дизайн-системы)
- С PR-описанием шаблоном

Сохрани как `docs/prompts/PHASE-0-{name}.md`.

### Step 8. Hand-off

Скажи пользователю:

> Roadmap собран. ROADMAP.md содержит {N} фаз, критический путь — {M} фаз. Acknowledged debt пока пусто (естественно, разработка не начата). Open questions: {K} штук, из них {L} блокируют phase 0.
>
> Следующий шаг:
> - Закройте {L} блокирующих open questions
> - Откройте `docs/prompts/PHASE-0-{name}.md`
> - Запустите agent в **новой сессии** с этим промптом + skill `canon-execute`
> - После merge — вернитесь сюда для phase 1 prompt'а

---

## Mode B — Continuation

### Step 1. Read state

Прочитай:
- `docs/ROADMAP.md` (особенно §10, §10.4, §11)
- `docs/CURRENT-STATE.md`
- `docs/progress/*.md` (если есть журналы прошлых фаз)

### Step 2. Identify last merged phase

Из git history (`git log --oneline -50`) найди последний merge'нутый phase-PR. Сверь с §10:
- Помечен ли там phase как `[x]`? Если нет — обнови.
- Есть ли journal entry в §11? Если нет — создай.

### Step 3. Audit deviations

Прочитай PR description и progress journal последней фазы. Найди:
- **Architectural deviations** — где реализация отклонилась от плана
- **Acknowledged debt** — какие новые tech-debt записаны
- **Bugs found** — критические ли, нужны hotfix?

Зафиксируй в §10.4 ROADMAP.

### Step 4. Update CURRENT-STATE

Обнови snapshot:

```markdown
# Current State — {date}

## Code
- Migrations: {N} applied
- Modules implemented: {list}
- Total LoC: {approx}

## Recently merged (since last update)
- {PR #X — что доставлено}
- {PR #Y — что доставлено}

## Acknowledged debt (new since last update)
- {Ф-code} — {описание + when to close}

## Next phase
Phase {N} — {name}. См. `docs/prompts/PHASE-{N}-{name}.md`.
```

### Step 5. Identify next phase

Из ROADMAP §10 — следующая `[ ]` фаза.

Проверь её **готовность к старту**:
- Все hard-dependencies выполнены?
- Все blocking open-questions закрыты?
- Дизайн-система покрывает нужные UI-компоненты?

Если не готова — flag user, объясни что блокирует.

### Step 6. Generate next phase prompt

Используя `templates/phase-prompt-template.md`. Особенности continuation-промпта:
- Ссылки на actual file paths из текущего состояния (не «как должно быть», а «вот как сейчас»)
- Список того что **не нужно делать** (уже сделано в прошлых фазах)
- Явное указание dependency на предыдущие фазы

### Step 7. Hand-off

```
Phase {N} prompt готов: `docs/prompts/PHASE-{N}-{name}.md`.

Pre-flight summary:
- Зависит от: phases {prev}
- Touches: {list of new files/modules}
- Не трогает: {list of stable areas}
- Acknowledged debt to acknowledge: {из §10.4}
- Estimated agent-effort: {S/M/L}

Запустите в новой сессии с canon-execute skill.
```

---

## Phase prompt — структура

Каждый phase prompt должен содержать:

### Header

```
# Phase {N} — {имя фазы}

> {Одно предложение про что делается + ссылка на canon-секцию}
```

### Section 0 — Context

```
## 0. Контекст

### 0.1. Что строим
{scope в 2-4 предложения}

### 0.2. На чём строимся (зависимости)
{что уже есть в main}

### 0.3. Чего НЕ делаем
{out-of-scope явно}

### 0.4. Канон-источники
{ссылки на конкретные §, цитаты ключевых положений}
```

### Section 1 — Brand rules

(Краткий повтор брендовых правил из дизайн-канона)

### Section 2-N — Implementation

```
## 2. Schema additions
{конкретные таблицы, поля, CHECK constraints}

## 3. {Business logic}
{формулы, lifecycle, state machine}

## 4. API contracts
{endpoints с Zod-схемами и response shapes}

## 5. UI changes
{компоненты, layout, состояния}

## 6. Cross-module integration
{hooks, contracts, antidubli}
```

### Section N+1 — Verification

```
## N+1. Pre-merge checklist

```bash
pnpm exec tsc --noEmit       # 0 errors
pnpm lint                     # 0 errors, 0 warnings
pnpm db:generate              # Создаёт миграцию
pnpm build                    # success
```

Brand-compliance grep:
{команды + ожидаемые results}

T-сценарии (агент проверяет локально):
1. ...
2. ...

## N+2. Out-of-scope (НЕ делать)
{явно перечислить}

## N+3. PR-описание шаблон
{Title + Body структура}

## N+4. Финальные акценты
{критические риск-зоны, акцент на atomic transactions, idempotency, etc.}
```

---

## Templates и references

- `templates/roadmap-template.md` — шаблон ROADMAP.md
- `templates/phase-prompt-template.md` — шаблон phase prompt
- `templates/current-state-template.md` — шаблон CURRENT-STATE.md
- `templates/journal-entry-template.md` — шаблон journal entry в §11

---

## Anti-patterns

❌ **«Фаза = месяц работы»** — нет, фаза = одна сессия агента (≤3000 LoC). Если больше — разбивай на sub-фазы.

❌ **«ROADMAP — статичный»** — нет, живой документ. После каждой фазы — update §10 + §11. Без update'а через 3 фазы он устаревает и теряет силу.

❌ **«Промпт — пара абзацев»** — нет, промпт самодостаточный (~500-1000 строк). Агент в чистой сессии должен выполнить.

❌ **«Координатор тоже реализует»** — нет, разделение ролей. Координатор пишет промпты + делает code-review + обновляет ROADMAP. Реализует — agent в отдельной сессии.

❌ **«Acknowledged debt — это плохо»** — нет. Acknowledged debt лучше неосознанного. Главное — регулярно пересматривать §10.4.

---

## Critical principles

1. **Самодостаточные промпты.** Агент должен мочь выполнить без обращения к чату.
2. **Живой ROADMAP.** Update после каждой фазы, не «один раз создали».
3. **Atomic phases.** Каждая фаза — merge'абельна без будущих.
4. **Critical path первым.** Не строй вторичные модули раньше core.
5. **Документируй deviations.** Каждое отклонение от плана → §10.4 + §11 запись.

---

## Финальная заметка

Хороший координатор-skill превращает каноны в **управляемый pipeline**, где каждая фаза — отдельный atomic delivery. Плохой — превращает в jumble из tasks без порядка.

Не сокращай ROADMAP-планирование. Час потраченный на правильную декомпозицию = недели сэкономленные на rework.
