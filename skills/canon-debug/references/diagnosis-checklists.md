# Diagnosis checklists — common root causes

> Проверочные списки по типам багов. Используй в Phase 3 (diagnose) когда ищешь hypothesis.

---

## Симптом: Intermittent test fails (flaky)

**Подозреваемые причины:**

- [ ] **Race conditions** — два теста делятся state, порядок выполнения влияет
- [ ] **Время / Date.now()** — test зависит от текущего времени
- [ ] **Random / UUID** — non-deterministic data
- [ ] **External resources** — DB / network / file-system состояние влияет
- [ ] **Test ordering dependency** — test passes в isolation, fails в suite

**Investigation:**
- Run `pnpm test:run -- --concurrent=1` (sequential) — passes? = race condition
- Run `pnpm test:run -- --seed 42` дважды — same result? = deterministic; different = randomness involved
- Check test для `Date.now()` / `new Date()` без injection
- Check для shared module state между tests

---

## Симптом: «Работает локально, не работает в production»

**Подозреваемые причины:**

- [ ] **Environment variables** — отличаются prod / local
- [ ] **Database state** — local fresh, prod loaded
- [ ] **Caching** — CDN / Redis / browser кэш в проде
- [ ] **Build differences** — Next.js production build vs dev
- [ ] **Edge case data** — prod имеет данные которых нет локально
- [ ] **Permissions / auth** — локально admin, prod restricted user
- [ ] **Timezone** — local UTC, prod в другом TZ
- [ ] **Concurrent load** — local 1 user, prod много

**Investigation:**
- Compare env vars: `printenv | sort` локально vs prod
- Check production logs (Sentry, CloudWatch) — есть errors?
- Production replica DB локально — воспроизводится?
- Build prod-mode локально: `pnpm build && pnpm start` — воспроизводится?

---

## Симптом: «Постепенно стало медленным»

**Подозреваемые причины:**

- [ ] **N+1 queries** — растёт с количеством rows
- [ ] **Missing index** — query сделался полным scan
- [ ] **Cache miss / invalidation issue**
- [ ] **Memory leak** — long-running processes
- [ ] **Накопленные orphan records** — query фильтрует через всё
- [ ] **External service slow** — третий-party API

**Investigation:**
- DB: `EXPLAIN ANALYZE {query}` — full scan? Sequential scan на large table?
- Profiler / flame chart
- Check для loops with awaited DB-calls
- Server memory usage trend
- Check upstream services latency

---

## Симптом: Non-deterministic data corruption

**Подозреваемые причины:**

- [ ] **Race condition** — concurrent writes
- [ ] **Missing transaction boundaries** — partial commits
- [ ] **Optimistic concurrency не реализован** — last-write-wins
- [ ] **CHECK constraint missing** — invalid state allowed
- [ ] **Foreign key cascade unexpected**

**Investigation:**
- Run test scenario с N concurrent calls
- Check schema CHECK constraints для целевых таблиц
- Check что critical-flow в `db.transaction`
- Audit log: какой user / process делал writes в момент corruption?

---

## Симптом: «Иногда деньги не сходятся»

**Подозреваемые причины:**

- [ ] **Float precision** — суммы хранятся как float
- [ ] **Округление inconsistent** — разные функции округляют по-разному
- [ ] **Missing transaction** — одна часть double-entry committed, другая failed
- [ ] **Sum vs running balance** — derived и stored balance расходятся
- [ ] **Currency conversion error** — rate stale или wrong direction
- [ ] **Antidubli failure** — двойной счёт через два module

**Investigation:**
- Check schema: суммы integer-копейки или decimal/float?
- Find все места `Math.round` / `Math.floor` / `toFixed` — consistent?
- Reconcile сумму через два независимых path — расходится?
- Audit-log: можно ли восстановить sequence событий?

---

## Симптом: «User видит чужие данные»

🔴 **CRITICAL — security issue.** Не двигаться в обычной фазе debugging. Hotfix немедленно.

**Подозреваемые причины:**

- [ ] **Missing orgId фильтр** в query
- [ ] **JOIN без orgId через цепочку**
- [ ] **Authorization check на client only** (server не проверяет)
- [ ] **Cache leak** — shared cache между users
- [ ] **Direct API access** — endpoint доступен через ID-guessing

**Immediate action:**
1. Identify scope — сколько users затронуто
2. Disable affected endpoint (если возможно)
3. Hotfix
4. Audit logs — кто конкретно accessed что
5. User communication если применимо
6. Post-mortem

---

## Симптом: «Что-то меняется само»

**Подозреваемые причины:**

- [ ] **Cron / scheduled job** делает что-то периодически
- [ ] **Background worker** processed queue items
- [ ] **Webhook** от external service
- [ ] **Database trigger** меняет state
- [ ] **Cascade delete** удаляет parent → children
- [ ] **Timer-based logic** (e.g. session expiration, status auto-transition)

**Investigation:**
- Check список cron jobs / scheduled functions
- Check DB triggers: `\\df` in psql
- Check audit-log — кто-что-когда менял
- Check для `setInterval` / `setTimeout` в server-code

---

## Симптом: «Status застрял в неправильном состоянии»

**Подозреваемые причины:**

- [ ] **Lifecycle transition fails silently** — state changed locally, persistence failed
- [ ] **Missing transaction** — status updated, related entity not
- [ ] **Concurrent transitions** — race на одном entity
- [ ] **External callback never came** (e.g. payment processor confirmation)
- [ ] **Error swallowed somewhere** — ошибка в transition не проагрегировалась

**Investigation:**
- Audit-log entity — какие transitions actually happened?
- Логи around expected transition time
- Check для `try/catch` без re-throw в transition path
- Check для async operation без await

---

## Симптом: TypeScript кажется работает но runtime fails

**Подозреваемые причины:**

- [ ] **Type assertion** (`as Foo`) обманул compiler — runtime data другая
- [ ] **`any` type** где должен быть конкретный
- [ ] **Schema drift** — типы от ORM не совпадают с DB
- [ ] **API response type** угаданный, не actual
- [ ] **JSON parsing** без validation (Zod)
- [ ] **Optional properties** где обязательные ожидались

**Investigation:**
- Search для `as any` / `as ${Type}` в подозреваемых местах
- Validate input через Zod на entry-points
- Compare ORM schema с DB schema (`pnpm db:generate` показывает drift)

---

## Универсальные quick-checks

Перед формулированием hypothesis — пройди по quick-checks:

```
1. Reproduce: точно воспроизводится?
2. Recent changes: что merged за последнюю неделю?
3. Logs: есть errors / warnings около время бага?
4. Environment: identical с тем где работает?
5. Data: какие данные триггерят? edge cases?
6. Permissions: какая роль / org context?
7. Concurrency: один user / много?
8. Timing: время суток / месяц / endof-period влияет?
```

Если что-то из этого «странно» — там и копай.

---

## Когда не получается понять

Если 2+ часа в Phase 3 без hypothesis:

1. **Объясни проблему другому человеку (или rubber duck).** Акт формулировки часто сам выявляет.
2. **Step-by-step trace** вручную — каждый файл, каждая строка участвующая.
3. **Add temporary logging** в КАЖДУЮ branch suspected кода. Run, читай logs.
4. **Спроси за помощью.** В команде, на форуме, у LLM с full context.
5. **Take a break.** Mental fatigue ухудшает diagnose. Свежая голова часто видит за 5 минут что zombie-tired не видит за 2 часа.

Никогда не выходи из Phase 3 в Phase 4 без понимания. Fix-by-guess — anti-pattern.
