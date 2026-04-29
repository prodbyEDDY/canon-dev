---
name: canon-execute
description: Use when an implementing agent receives a phase prompt and needs to execute it correctly. Enforces pre-flight checks (read CLAUDE.md, verify state), branch hygiene (feature branch, never main), spec-gap reporting (flag mismatches BEFORE coding), implementation discipline (atomic transactions, idempotency, brand compliance), and pre-merge gates (tsc/lint/db:generate/build + manual T-scenarios). For the agent that executes a single phase in a single session.
---

# canon-execute

🚀 Стадия 5 в pipeline canon-dev: **дисциплина реализации одной фазы**

> Этот скилл — для **агента-реализатора в отдельной сессии**, не для координатора. Если ты координатор — твой скилл `canon-roadmap`. Если получил phase prompt и собираешься писать код — это твой скилл.

---

## Когда использовать

- Получил самодостаточный phase prompt от координатора
- Нужно реализовать одну фазу за одну сессию и открыть PR
- Любая работа над production-кодом из канон-driven проекта

## Когда НЕ использовать

- Нет промпта, есть только идея → координатору нужен `canon-roadmap`
- Нужно проверить готовый код → `canon-verify`
- Нужно создать каноны → `braindump-to-canon`

---

## Философия скилла

Реализатор — **дисциплинированный исполнитель**. Не «спонтанный творец». Это означает:

1. **Самодостаточный промпт — закон.** Не догадывайся, не improvise за пределы. Что в промпте — реализуй; чего нет — не добавляй.

2. **Spec-gaps — флаг ДО кодинга.** Если промпт расходится с реальностью кода/схемы — остановись, доложи координатору, жди подтверждения. Это спасает день.

3. **Branch hygiene.** Никогда на main. Feature branch. Чистый pull main first.

4. **Atomic discipline.** Все critical-flow в `db.transaction`. Все user-facing creates с idempotency-key.

5. **Pre-merge gates обязательны.** `tsc`, `lint`, `build`, brand-grep — пройти ДО открытия PR.

6. **Honest reporting.** В PR description указать deviations от плана, acknowledged debt, T-сценарии прошедшие/упавшие. Не приукрашать.

---

## Структура процесса

```
┌───────────────────────────────────────────────────────┐
│  Phase 1 — Pre-flight (read context, verify state)    │
└───────────────────────────────────────────────────────┘
                          ↓
┌───────────────────────────────────────────────────────┐
│  Phase 2 — Spec-gap reporting (если есть расхождения) │
└───────────────────────────────────────────────────────┘
                          ↓
┌───────────────────────────────────────────────────────┐
│  Phase 3 — Branch setup (feature branch from main)    │
└───────────────────────────────────────────────────────┘
                          ↓
┌───────────────────────────────────────────────────────┐
│  Phase 4 — Implementation (с дисциплиной)             │
└───────────────────────────────────────────────────────┘
                          ↓
┌───────────────────────────────────────────────────────┐
│  Phase 5 — Self-verification (T-scenarios + gates)    │
└───────────────────────────────────────────────────────┘
                          ↓
┌───────────────────────────────────────────────────────┐
│  Phase 6 — PR open (с honest description)             │
└───────────────────────────────────────────────────────┘
```

---

## Phase 1 — Pre-flight

### Step 1. Read CLAUDE.md (или эквивалент)

Найди и прочитай **полностью**:
- `CLAUDE.md` в корне репо
- `BUSINESS-RF/CLAUDE.md` если monorepo
- Любой `.cursorrules` / `.codiumrules` / project-specific instructions

Цель: понять стек, конвенции, миграционный workflow, дизайн-правила. Не пропускай — это база.

### Step 2. Read the phase prompt

Прочитай промпт **полностью**, не сокращённо. Особенно:
- §0.4 (out-of-scope) — что НЕ трогать
- §0.5 (canon sources) — какие правила применяются
- §1 (brand rules) — visual constraints
- §N+2 (out-of-scope повтор) — для verification

Сделай ментальный inventory:
- Какие файлы создаются?
- Какие файлы изменяются?
- Какие НЕ трогать?
- Какие проверки выполнить локально?

### Step 3. Verify current state matches prompt assumptions

Промпт предполагает что определённые файлы / таблицы / API уже существуют. Проверь:

```bash
# Например, если промпт ссылается на src/lib/finance/operations.ts:
ls src/lib/finance/operations.ts && grep -n "export function postOperation" src/lib/finance/operations.ts

# Например, если предполагается определённая таблица:
grep -n "export const taxProfile" src/db/schema/

# Проверь актуальный git state:
git status --short
git log --oneline -5
git branch --show-current
```

### Step 4. Verify dependencies merged

Промпт может говорить «зависит от Phase {prev}». Проверь что:
- Соответствующий PR действительно merged в main
- main fast-forwarded локально
- Нет uncommitted changes которые помешают

```bash
git fetch --all --prune
git checkout main
git pull --ff-only
git status --short  # should be clean
```

---

## Phase 2 — Spec-gap reporting (если есть)

Если на Phase 1 нашёл расхождения между промптом и реальностью — **остановись**. Не пиши код. Сообщи координатору:

### Шаблон отчёта

```markdown
Перед началом реализации обнаружил {N} расхождений между промптом и текущим состоянием:

1. **API signature mismatch:**
   - Промпт §X.Y предполагает `createFoo({ field1, field2 })`
   - Актуально в `src/lib/foo/bar.ts:42`: `createFoo({ fieldOne, fieldTwo })`
   - Адаптация: использую актуальные имена

2. **Schema deviation:**
   - Промпт §Z говорит «колонка `code` существует на `categories`»
   - Актуальная схема не содержит `code` — есть только `name` (см. `src/db/schema/categories.ts`)
   - Адаптация: использую name-based lookup matching pattern из `ensureCategoriesByName`

3. **Major architectural finding:**
   - Промпт §W предполагает создание `financial_operation` с null financeAccountId для accruals
   - Schema constraint: `financialOperation.financeAccountId NOT NULL`
   - Этого не описано в промпте
   - **Предлагаемое решение:** {описать workaround} — нужно явное подтверждение координатора

Подтверди адаптации 1-2 и реши по #3 — после ответа начну.
```

**Не начинай кодинг** пока не получил подтверждение. Это правило золотое — нарушение стоит дня rework'а.

---

## Phase 3 — Branch setup

### Standard branch naming

```
feat/{module}-{phase-slug}
```

Например:
- `feat/taxes-4d1-foundation`
- `feat/auth-onboarding-redesign`
- `feat/landing-v2-rework`

### Setup commands

```bash
git fetch --all --prune
git checkout main
git pull --ff-only
git checkout -b feat/{module}-{phase-slug}
```

### Verify clean start

```bash
git status --short  # should be empty
git log --oneline -1  # should match remote main
```

### Anti-patterns

❌ Работа на main — никогда. Если случайно закоммитил на main:
```bash
# Save commit:
git log --oneline -1  # запомни SHA
# Reset main back:
git reset --hard origin/main
# Cherry-pick on new branch:
git checkout -b feat/{name}
git cherry-pick {sha}
```

❌ Branch from feature-branch другого PR — только если этот PR merged или explicitly базовая ветка для последовательности (например, 4D.2 базируется на 4D.1).

---

## Phase 4 — Implementation

### General discipline

**Read before write.** Перед изменением файла — прочитай его. Не предполагай содержимое.

**Reference canons.** В комментариях к нетривиальной логике — ссылка на canon-секцию:
```typescript
/**
 * Канон §16: НДС квартальный split на 3 равные доли
 * с balancing на 3-ю для precision-loss защиты.
 */
function splitNds(total: number): [number, number, number] {
  const part1 = Math.floor(total / 3);
  const part2 = Math.floor(total / 3);
  const part3 = total - part1 - part2;  // balancing
  return [part1, part2, part3];
}
```

**Atomic transactions.** Critical-flow обёрнут в `db.transaction`:
```typescript
await db.transaction(async (tx) => {
  // 1. UPDATE existing entity (mark old version)
  await tx.update(table).set({ isCurrent: false }).where(...);
  // 2. INSERT new entity
  const [newRow] = await tx.insert(table).values(...).returning();
  // 3. Audit log
  await tx.insert(auditLog).values(...);
});
```

**Idempotency keys** для user-facing creates:
```typescript
const idempotencyKey = `${module}-${entityId}-${action}`;
// Pre-check or rely on unique partial index
```

**Money as integer.** Никогда float для денег. Все суммы в копейках/центах:
```typescript
amountKopecks: bigint("amount_kopecks", { mode: "number" }).notNull(),
// CHECK constraint:
check("amount_non_negative", sql`amount_kopecks >= 0`),
```

### Brand compliance

Перед коммитом каждого UI-файла — mental scan:
- Использую ли только токены (`var(--brand-700)`), не hex literals?
- Никаких `font-bold`/`font-semibold`?
- Никаких эмодзи в production UI?
- Цвета только из brand-палитры (no purple/teal/orange)?

### Migration workflow

Если изменил `src/db/schema/`:

```bash
pnpm db:generate
# Создаёт src/db/migrations/{N}_{name}.sql + meta/{N}_snapshot.json
```

**Закоммить и schema, и migration файл вместе** в одном коммите. Не push schema без migration — production deploy сломается.

### Documentation alongside code

- Сложная функция → JSDoc с canon-ref
- Новый module → README в module-folder если требуется (для нетривиальных архитектур)
- Workaround / deviation → in-code комментарий `// TODO(Phase X): ...` со ссылкой на ROADMAP §10.4

### Anti-patterns

❌ **«Я добавлю эту фичу заодно — она маленькая»** — нет. Out-of-scope = out-of-scope. Если действительно нужно — спроси координатора.

❌ **«Перепишу старый legacy код раз уж рядом»** — нет. Refactor = отдельный PR. Этот PR — только scope текущей фазы.

❌ **«Skip this CHECK constraint, добавлю позже»** — нет. CHECK constraints критичны для data integrity. Добавляй сразу.

❌ **«Use `any` тут потому что некогда разбираться с types»** — нет. Если type-safety мешает — это сигнал что архитектура где-то расходится. Разберись сразу.

---

## Phase 5 — Self-verification

### Step 1. Run pre-merge gates

```bash
pnpm exec tsc --noEmit       # 0 errors
pnpm lint                     # 0 errors, 0 warnings
pnpm db:generate              # "no schema changes" если schema не трогал;
                              # либо создаёт migration если трогал
pnpm build                    # success
```

Каждый шаг — должен пройти. Если упал — fix перед continue.

### Step 2. Brand-compliance grep

Из промпта возьми grep-команды. Запусти. Все должны вернуть 0 results:

```bash
grep -rn "font-bold\|font-semibold" src/{paths-changed}/ 2>/dev/null
grep -rn "purple\|violet\|teal\|emerald\|orange\|amber" src/{paths-changed}/ 2>/dev/null
grep -rn "text-black\|bg-black" src/{paths-changed}/ 2>/dev/null
grep -rn "console\.log" src/{paths-changed}/ 2>/dev/null  # log-cleanup
```

### Step 3. T-сценарии

Промпт содержит T-сценарии (T-01..T-N). Пройди каждый локально:
- Запусти `pnpm dev`
- Открой соответствующий URL
- Выполни сценарий
- Проверь ожидаемый результат

Записывай статус: ✅ pass / ⚠️ partial / ❌ fail. Если ❌ — fix перед PR.

**Note:** если auth-gated UI и dev-окружение не имеет рабочей auth — отметь как «verifiable on staging only» и упомяни в PR description.

### Step 4. Verify deliverables list

Промпт обещал N deliverables. Пройди по списку:
- [ ] Schema: все таблицы созданы
- [ ] API endpoints: все работают (curl-test или DevTools)
- [ ] UI: все pages/components рендерятся
- [ ] Migration: файл закоммичен
- [ ] Cross-module hooks: интегрированы

---

## Phase 6 — PR open

### Commit message

```
feat({module}): {краткое описание} (Phase {N})

{Несколько строк деталей если нужно}

Co-Authored-By: {standard footer}
```

### Push branch

```bash
git push -u origin feat/{module}-{phase-slug}
```

### Open PR

Используй промпт §10 (PR-описание шаблон) как baseline. Honest filling:

```markdown
## Контекст

PR Phase {N}.{X} из фазы {parent}. Зависит от Phase {prev} (PR #{num}).

## Что сделано

- Schema: {list}
- API: {list endpoints}  
- UI: {list components}
- Tests: T-01..T-N — {N pass / M partial / 0 fail}

## Acknowledged deviations

(если есть)
1. **Plan §X.Y предполагал {expected}** — реализовано как **{actual}** из-за **{constraint}**. См. inline-комментарий в `{file}:{line}`. Acknowledged debt — TODO в `{file}` для будущего пересмотра.

## Pre-merge gates

- [x] `pnpm exec tsc --noEmit` — 0 errors
- [x] `pnpm lint` — 0 errors, 0 warnings
- [x] `pnpm build` — success
- [x] `pnpm db:generate` — migration file committed
- [x] Brand grep clean
- [x] T-сценарии: {N}/{N} pass

## Coordinator review notes

(если есть что-то требующее внимания координатора)
- Architectural decision: {описание}, see `{file}` README
- Verify on staging: T-{N} cannot be tested locally due to {reason}

## Screenshots

(desktop / tablet / mobile)
```

### Wait for review

Не делай дополнительных коммитов после открытия PR пока не получил code-review feedback. Если случилась срочная правка (e.g., CI упал) — отдельный коммит с понятным message.

---

## Anti-patterns (общие)

❌ **«Промпт в общих чертах понял, начну, по ходу разберусь»** — нет. Полное чтение перед началом.

❌ **«Sounds reasonable, я знаю как сделать лучше чем в промпте»** — нет. Если есть улучшение — флаг spec-gap, обсуди с координатором.

❌ **«Брендовые правила — necesary evil, главное чтобы работало»** — нет. Brand-compliance — это про доверие пользователя к продукту.

❌ **«Pre-merge gates — formality, я уже проверил вручную»** — нет. Gates запускаются. Если упали — fix.

❌ **«PR description короткий, важно code»** — нет. PR description — единственный способ для ревьюера понять что сделано без полного чтения diff'а.

---

## Critical principles

1. **Промпт — самодостаточен.** Не дополняй из памяти, не сокращай.
2. **Spec-gaps — стоп-сигнал.** Доложи, не угадывай.
3. **Feature branch always.** Никогда на main.
4. **Atomic + idempotent.** Critical-flow в transaction, creates с key.
5. **Pre-merge gates — closeable list.** Все ✅ перед PR.
6. **Honest PR description.** Deviations явно, T-scenarios честно.
7. **Out-of-scope — святой.** Не делай больше чем просили.

---

## Templates и references

- `references/agent-discipline.md` — расширенные anti-patterns + примеры
- `references/pre-merge-gates.md` — детальный gate-checklist
- `references/spec-gap-reporting.md` — шаблоны spec-gap отчётов

---

## Финальная заметка

Хороший реализатор — **предсказуемый**. Координатор передаёт промпт и знает: код будет таким как в промпте, deviations будут зафиксированы явно, gates пройдены, PR honest.

Это даёт координатору возможность параллельно работать над next-phase planning, не отвлекаясь на «что там агент мутит». А значит — pipeline ускоряется и качество растёт.

Не сокращай дисциплину. Каждый shortcut окупается shortcuts'ом ревьюера, который не доверяет твоему PR.
