# Current State — {Product name}

> Snapshot текущего состояния проекта на {date}. Обновляется после каждой фазы.

---

## Repository
- URL: {git-url}
- Default branch: main
- Last commit: {sha} ({date})

## Stack (см. STACK.md для полного)

- Framework: {e.g. Next.js 16}
- Database: {e.g. PostgreSQL 18}
- ORM: {e.g. Drizzle}
- Auth: {e.g. BetterAuth}
- Deployment: {e.g. Railway}

## Schema state

- Migrations applied: {N} (last: `{filename}`)
- Tables: {count}
- Total columns: {approx}
- Active enums: {list}

## Modules implemented

| Module | Status | Last touched |
|---|---|---|
| {Module 1} | ✅ Phase {N} | {date} |
| {Module 2} | ⏳ In progress (Phase {M}) | {date} |
| {Module 3} | ⬜ Not started | — |

## Recently merged (last 5 PRs)

| PR | Title | Date | Phase |
|---|---|---|---|
| #{X} | {title} | {date} | {phase} |
| ... | | | |

## Acknowledged tech debt (from ROADMAP §10.4)

| Code | Description | Phase to close |
|---|---|---|
| Ф{N} | {summary} | {phase target} |

## Open questions blocking phases

| Q | Phase blocked | Status |
|---|---|---|
| Q-{N} | {description} | {open / under discussion / closed} |

## Active work

{Если есть активный agent в work — какая фаза, кто, ETA}

## Next phase

**Phase {N} — {name}**

- Status: {ready to start / blocked by Q-X / waiting for design}
- Promtp: `docs/prompts/PHASE-{N}-{name}.md`
- Estimated agent-effort: {S/M/L}

## Recent decisions (from ROADMAP §11)

(Последние 3-5 entries из журнала)

- {date}: {decision summary}
- {date}: {decision summary}
- ...

---

## How to update

После каждого merge'а phase-PR — координатор обновляет этот файл:

1. Repository: новый last commit
2. Schema state: новые миграции
3. Modules implemented: помечает фазу как ✅ + дата
4. Recently merged: добавляет новый PR в топ
5. Acknowledged tech debt: новые Ф-codes из progress journal
6. Open questions: closes resolved, добавляет новые если выявлены
7. Active work: clear (если фаза закрыта)
8. Next phase: новая фаза + ссылка на промпт

Цель — чтобы любой человек открыл этот файл и за 2 минуты понял где проект находится.
