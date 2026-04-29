# ROADMAP — {Product name}

> Живой документ. Обновляется после каждой фазы реализации. При конфликте этого документа с фактическим состоянием кода — обновляется документ.

---

## 0. Executive summary

### 0.1. Что строим

{Одно-два предложения про продукт}

### 0.2. Стратегия реализации

Канон-driven. Каждая фаза:
- Опирается на canon из `docs/КАНОНЫ/`
- Начинается с явного scope и out-of-scope
- Завершается merge'абельным PR
- Документируется в §10 (live-track) и §11 (journal)

### 0.3. Порядок реализации

{Краткий обзор фаз — 5-10 строк, с указанием критического пути}

---

## 1. Текущее состояние (snapshot на {date})

### 1.1. Код
- Repository: {url}
- Stack: см. `docs/STACK.md`
- Migrations applied: {N}
- Modules implemented: {list}

### 1.2. Документация
- Canons: {list of files}
- ТЗ: {list of files}
- Design system: {ready / not yet — link if ready}

### 1.3. Active phase
{Если идёт работа — какая фаза, кто делает, ETA}

---

## 2. Целевое состояние

### 2.1. Карта модулей

```
01-{Module}     — {one-line description}
02-{Module}     — ...
...
0N-{Module}     — ...
```

### 2.2. Ключевые инварианты (сквозные)

(Скопировать из главного канона §2)

- {Инвариант 1}
- {Инвариант 2}
- ...

---

## 3. Архитектурные принципы (сквозные для всех фаз)

(Из главного канона + canon-dev философии)

### 3.1. Multi-tenancy
{Краткое описание стратегии}

### 3.2. Источник истины
{...}

### 3.3. Audit logging
{...}

### 3.4. Money handling (если применимо)
Все суммы в integer-копейках/центах. Никогда float.

### 3.5. Antidubli
Idempotency keys + partial unique indexes + флаги участия в отчётах.

### 3.6. Documentation discipline
Каноны выше кода. При расхождении — правится код.

---

## 4. Дерево зависимостей и критический путь

```
{module-A} → {module-B} → {module-C}
                       ↘
                         {module-D}
```

**Критический путь:** {ordered list of modules}

**Risk hotspots:**
- {Module X} — {почему рискованный — большой scope, денежная логика, etc.}
- {Module Y} — ...

---

## 5. Фазы реализации (детальное описание)

### Фаза 0 — Подготовка инфраструктуры

**Цель:** Базовая инфраструктура — repo, stack, deployment, auth-shell.

**Deliverables:**
- Next.js project initialized
- BetterAuth + organization plugin
- Database migrations workflow
- CI/CD basics (lint, type-check, build)
- Deployment to staging

**Зависимости:** none

### Фаза 1 — {Module}

**Цель:** ...

**Deliverables:**
- ...

**Зависимости:** Phase 0

### Фаза 2 — {Module}
...

(Повторить для каждой фазы)

---

## 10. Live-track (чек-лист прогресса)

> Обновляй по мере закрытия фаз. Формат: `[ ]` → `[x]` + дата + PR-номер.

- [ ] Фаза 0 — Подготовка инфраструктуры
- [ ] Фаза 1 — {Module}
- [ ] Фаза 2 — {Module}
- [ ] Фаза 3 — {Module}
- [ ] Фаза 4 — {Module}
- [ ] Фаза N — {Module}

---

## 10.1-10.N. Post-phase tech debt

> Здесь регистрируются acknowledged debts, обнаруженные в ходе фаз. Структура: код-Ф{N}, описание, статус, условия закрытия.

### 10.1. Post-Phase-1 tech debt

- [ ] **Ф1-1.** {Описание} — {почему отложено} — {когда закрывать}
- [ ] **Ф1-2.** ...

### 10.2. Post-Phase-2 tech debt

- [ ] **Ф2-1.** ...

(...и так далее по фазам)

---

## 11. Журнал решений

> Крупные развилки и решения, в обратном хронологическом порядке.

### {YYYY-MM-DD} — Phase X замёржен (PR #N)

**Контекст:** {краткое описание что делалось}

**Что замёржено:** {list}

**Расхождения с планом:** {acknowledged deviations с обоснованием}

**Open questions resolved:** {если что-то закрыто}

**Acknowledged debt added:** {Ф-codes из §10.4}

**Verification:** {что проверено локально, что на staging}

### {YYYY-MM-DD} — {Major decision}

**Контекст:** ...
**Решение:** ...
**Альтернативы рассмотрены:** ...
**Trade-offs:** ...

(Все решения которые меняют архитектуру / стек / scope — фиксируются здесь)
