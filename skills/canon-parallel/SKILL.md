---
name: canon-parallel
description: Use when facing 2+ independent tasks that can be worked on without shared state or sequential dependencies. Determines whether parallel execution is appropriate (real independence vs hidden coupling), how to dispatch concurrent agents safely, how to merge results. For coordinator role — typically when fan-out across multiple modules, batch refactors, or independent investigations needed simultaneously.
---

# canon-parallel

🚀 Дисциплина параллельной работы в canon-dev pipeline

> Скилл для **координатора** который рассматривает запуск нескольких агентов одновременно. Не для агента-реализатора.

---

## Когда использовать

- 2+ задачи которые действительно независимы (не делятся state, не зависят последовательно)
- Batch refactor через несколько модулей с одинаковым pattern
- Independent investigations (audit разных модулей одновременно)
- Fan-out research (изучить разные альтернативы параллельно)
- Когда time pressure и distinct work streams

## Когда НЕ использовать

- Tasks share state или sequential dependencies → последовательно
- Один большой task разбит на «части» которые не делают sense изолированно
- Архитектурный design решение → один агент, один thread рассуждения
- Когда unsure про independence → сначала clarify, потом параллелить

---

## Философия

Parallelism — **multiplier когда настоящий**, **chaos когда фейковый**.

**Настоящая независимость:**
- Task A не нуждается в результатах task B
- Каждый agent видит/читает свою часть
- Merge результатов тривиален (или не нужен)

**Фейковая независимость:**
- «Они оба про Foo, можно параллельно»
- Но один меняет схему Foo, другой её читает
- Race conditions, conflicting changes
- В итоге — на 30% медленнее sequential потому что нужно untangle

**Правило:** если есть сомнение — sequential. Parallelism стоит coordination overhead. Чтобы окупиться — задачи действительно независимы.

---

## Phase 1 — Independence audit

Перед dispatch'ом — проверь что задачи реально независимы.

### Test 1: shared mutable state

- Будут ли agents трогать одни и те же файлы?
- Одни и те же DB-таблицы (если не read-only)?
- Один и тот же config?

Если да — **не параллельно**. Sequential, или re-разделить так чтобы не пересекалось.

### Test 2: sequential dependency

- Task B нуждается в результате task A?
- Task B читает то что A пишет?

Если да — sequential.

### Test 3: shared mental context

- Одно архитектурное решение охватывает обе задачи?
- Решение про А влияет на как делать B?

Если да — один агент должен делать обе (или один — оба design'а, потом split на parallel implementation).

### Test 4: merge complexity

- Если оба agent'а закончат — насколько сложно объединить?
- Conflict-rate ожидаемая большая?

Если да — sequential или re-divide work.

### Если все 4 теста — ОК → можно parallelёе

---

## Phase 2 — Task partition

Раздели работу на чёткие, изолированные tasks.

### Pattern A — By module

Каждый agent работает над разным модулем:

```
Agent 1: refactor module Foo (only files src/lib/foo/)
Agent 2: refactor module Bar (only files src/lib/bar/)
Agent 3: refactor module Baz (only files src/lib/baz/)
```

Контракт каждого agent: «не выходить за пределы своего module folder».

### Pattern B — By layer

Каждый agent работает над разным layer:

```
Agent 1: schema migrations
Agent 2: API routes
Agent 3: UI components
```

Опасно если layers зависят друг от друга. Использовать только если phases independent (например, schema уже final, API и UI можно параллелить от него).

### Pattern C — By investigation

Каждый agent делает отдельное research-задание:

```
Agent 1: audit module Foo for canon compliance
Agent 2: audit module Bar for canon compliance
Agent 3: audit cross-cutting concerns
```

Все independent, merge — просто concatenate findings.

### Anti-pattern

❌ **Pattern D — by file** — «agent 1 правит line 1-50, agent 2 правит line 51-100». Это всегда конфликтует. Никогда.

---

## Phase 3 — Dispatch

### Self-contained briefs

Каждому agent — **самодостаточный prompt**. Не «дополни context из чата». Все включено:

- Что делать
- Что НЕ трогать (явно: «не выходи за пределы src/lib/foo/»)
- Где результат
- Pre-merge checklist
- Branch name

### Branch isolation

Каждый agent → собственная feature branch. Использовать `canon-worktree` для физической изоляции (отдельная checkout):

```
worktree-1/feat/refactor-foo
worktree-2/feat/refactor-bar
worktree-3/feat/refactor-baz
```

Это означает каждый agent видит свою копию repo. Merge через PRs в main.

### Communication channel

Если agent'ам нужно coordinate (rare) — explicit shared file:

```
Agents записывают status в docs/parallel-{batch-id}/agent-{N}-status.md
Coordinator polls.
```

Не через чат. Каждый агент в своей сессии.

---

## Phase 4 — Coordinate

Пока agents работают — coordinator:

- Не «переключается» между agents постоянно. Это смена контекста стоит.
- Дождать когда все завершат свою задачу
- Тогда review каждый результат independently
- Тогда merge / sequence PRs

### Если один agent failed

- НЕ paniковать
- Cancel его work (если ещё running)
- Re-dispatch с уточнениями
- Не задерживай parallel — другие могут продолжать

### Если есть unexpected dependency между agents

- Stop affected agents
- Sequential re-plan
- Restart с правильным разделением

---

## Phase 5 — Merge

Когда все agents finished:

### Independent merge (best case)

PRs не пересекаются → merge each one после review. Order matters только для CI.

### Stacked merge

PR_B ожидает PR_A merged → PR_B rebases.

### Conflict resolution

Если два agent'а случайно затронули одинаковый file — это нарушение partition (Phase 2). Прежде чем resolve conflicts — проанализируй: были ли они реально independent?

---

## Когда parallel НЕ работает

Реальные ситуации где parallel attempted и не получилось:

### «Refactor 8 модулей под новые токены»

- Каждый модуль независим? Yes
- Но: shared `globals.css` где tokens живут
- Агенты могут все заодно править globals.css → conflicts
- **Solution:** один агент сначала finalizes globals.css, потом 8 параллельных делают refactor only их компонентов

### «Implement налоги и зарплату параллельно»

- Кажется independent (разные модули)
- Но: налоги читают данные ФОТ для НДФЛ-обязательств
- Architectural decision: «как именно ФОТ передаёт НДФЛ налогам» — должен быть **первым**
- **Solution:** один agent делает interface design (canon update), потом параллельно реализаций каждого модуля по этому interface

### «Audit всей системы parallel — 4 агента, разные модули»

- Это как раз case где parallel works
- Findings independent, merge тривиален (concatenate в один report)
- ✓ Good fit

---

## Anti-patterns

❌ **Parallel ради скорости когда работа sequential** — overhead делает медленнее.

❌ **Один agent пишет, другой чек'кает** — два чтения одного state, нужен sequential review.

❌ **«Агенты сами скоординируются»** — нет, они в isolated sessions. Coordinator координирует.

❌ **Skip Phase 1 independence audit** — основная причина parallel failures.

❌ **Dispatch без branch isolation** — оба agent'а push'ят в одну branch, конфликт неминуем.

---

## Critical principles

1. **Independence audit обязателен.** 4 теста.
2. **Self-contained briefs** для каждого agent.
3. **Branch isolation.** Worktrees или separate feature branches.
4. **Coordinator не переключается** между agents без причины.
5. **Один failed agent не должен blokировать остальных.**
6. **Conflict = partition violation.** Не просто resolve — analyze why.

---

## Templates и references

- См. `canon-worktree` для setup физической изоляции

---

## Финальная заметка

Parallelism — техника. Не убеждение. Используй когда задачи реально independent. Не используй ради «выглядит современно».

Sequential работа с хорошими промптами часто **быстрее** плохо координированной parallel.
