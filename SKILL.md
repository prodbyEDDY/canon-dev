---
name: using-canon-dev
description: Use when starting any work on a product that follows canon-driven development — establishes which sub-skill to invoke and in what order. Covers six sub-skills (braindump-to-canon, canon-review, design-canon, canon-roadmap, canon-execute, canon-verify) forming a full pipeline from product idea to production code.
---

# canon-dev — навигатор по библиотеке

🚀 **Канон → Архитектура → План → Реализация → Верификация**

Этот файл — точка входа. Прочитай и реши, какой sub-скилл нужен для текущей задачи. Все sub-скиллы лежат в `skills/`.

---

## Когда что использовать

### `braindump-to-canon`

Используй когда у пользователя есть **сырая идея продукта** или **отдельный модуль без формализации**:

- «Хочу сделать X для Y»
- «Опиши архитектуру для…»
- «Помоги расписать спецификацию модуля Z»
- Свободный текст / голосовая запись / список фич без структуры

Скилл проводит структурированное интервью, фиксирует решения, генерирует канон-документацию.

### `canon-review`

Используй когда **каноны/ТЗ уже существуют**, но нужно проверить на полноту:

- «У нас есть каноны, но я подозреваю что-то упущено»
- «Перед началом разработки — проверь архитектуру»
- «Найди дыры в проектировании БД»
- Enterprise-readiness check перед production

Скилл проходит по матрице измерений, находит пробелы, предлагает дополнения.

### `design-canon`

Используй когда нужна **визуальная дизайн-система продукта**:

- «У нас есть бренд-материалы — собери дизайн-систему»
- «Нет дизайн-системы — построй с нуля»
- «Существующая дизайн-система фрагментирована»

Скилл создаёт токены, типографику, иконки, motion-rules — и завершает **HTML-предпросмотром** дизайн-системы для проверки до кодинга.

### `canon-roadmap`

Используй когда **каноны и дизайн готовы**, нужен план реализации:

- «Каноны готовы — собери roadmap»
- «Какая следующая фаза по плану?»
- «Сгенерируй промпт для следующего этапа»

Скилл генерирует ROADMAP, snapshot состояния, промпт следующей фазы. На повторных запусках — продолжает по плану.

### `canon-execute`

Используй когда **нужна дисциплина реализации одной фазы**:

- Агент-реализатор получает промпт и должен корректно его выполнить
- Pre-flight проверки, branch hygiene, transactions, idempotency, pre-merge gates

Скилл — для **исполняющего агента в отдельной сессии**, не для оркестратора.

### `canon-verify`

Используй когда **реализация завершена**, нужен audit:

- После merge фазы — сверка с каноном
- Перед production-release — comprehensive audit
- Подозрение на regression в существующем коде

Скилл — read-only auditor. Находит расхождения, классифицирует, не правит.

### Cross-cutting discipline skills

Эти скиллы применяются **внутри** или **параллельно** основным фазам, не как отдельные стадии:

#### `canon-tdd` (TDD discipline)

Применяй внутри `canon-execute` когда фаза включает testable логику (расчёты, lifecycle, validators, antidubli). RED-GREEN-REFACTOR цикл. Bug-fix начинается с failing test.

#### `canon-debug` (systematic debugging)

Применяй когда что-то сломалось, до любого fix-attempt'а. 4-фазное расследование: reproduce → isolate → diagnose → fix → verify → defense-in-depth.

#### `canon-receive-review` (review feedback handling)

Применяй когда получил comments на свой PR. Three response modes: Implement / Push back / Document tradeoff. Verify before implement, batch responses, no performative agreement.

#### `canon-parallel` (dispatching parallel agents)

Применяй когда coordinator рассматривает 2+ independent tasks одновременно. Independence audit (4 теста), partition by module/layer/investigation, branch isolation через worktrees.

#### `canon-worktree` (git worktree utility)

Утилита — создание изолированных checkout'ов. Используется при parallel work или когда нужна physical isolation (hotfix во время feature work, debug на старом commit).

---

## Decision flow

```
Что у пользователя?
├─ Сырая идея                  → braindump-to-canon
├─ Каноны есть, но дыры        → canon-review
├─ Каноны готовы, нет дизайна  → design-canon
├─ Каноны+дизайн готовы        → canon-roadmap
├─ Промпт фазы получен         → canon-execute
│   ├─ Фаза включает testable logic → + canon-tdd параллельно
│   ├─ Что-то сломалось         → + canon-debug
│   └─ Получил review-comments  → + canon-receive-review
├─ Фаза реализована            → canon-verify
└─ Несколько independent tasks → canon-parallel + canon-worktree для isolation
```

---

## Pipeline (полный цикл)

Если работаешь над продуктом с нуля — порядок такой:

1. `braindump-to-canon` → готовы каноны и ТЗ модулей
2. `canon-review` → каноны доработаны до enterprise-ready
3. `design-canon` → дизайн-система с HTML preview
4. `canon-roadmap` → ROADMAP + промпт фазы 1
5. `canon-execute` (новая сессия) → реализована фаза 1
6. `canon-verify` (новая сессия) → audit фазы 1
7. `canon-roadmap` (повтор) → промпт фазы 2
8. ↺ повтор 5-7 до завершения roadmap

После каждого блока — обновление ROADMAP §10 (live-track) и §11 (журнал решений) самим оркестратором.

---

## Роли в системе

`canon-dev` явно различает **три роли**, чтобы избежать смешения:

| Роль | Кто | Что делает | Какие скиллы использует |
|---|---|---|---|
| **Архитектор** | Пользователь + Claude в брейнсторм-сессии | Формирует каноны, принимает архитектурные решения, утверждает стек | `braindump-to-canon`, `canon-review`, `design-canon` |
| **Координатор/Оркестратор** | Claude в долгоживущей сессии | Декомпозирует на фазы, пишет промпты, делает code-review, обновляет ROADMAP | `canon-roadmap` |
| **Реализатор** | Claude в отдельной сессии за фазу | Реализует одну фазу по промпту, открывает PR | `canon-execute` |
| **Аудитор** | Claude в отдельной чистой сессии | Read-only сверка реализации с каноном | `canon-verify` |

**Не смешивать роли в одной сессии.** Координатор не должен реализовывать (теряется фокус). Реализатор не должен писать промпты (теряется bird's-eye). Аудитор не должен править (теряется независимость).

---

## Принципы (краткая шпаргалка)

1. **Канон выше кода.** Конфликт → правим код или сознательно обновляем канон с записью в журнал.
2. **Документация — закон.** Решения существуют сначала текстом, потом кодом.
3. **Никаких хардкодов.** Цвета/размеры/пороги/формулы — через токены и реестры с версионностью.
4. **Самодостаточные промпты.** Агент-реализатор не должен догадываться или искать. Всё в брифе.
5. **Acknowledged debt.** Когда невозможно сделать каноничный — фиксируется как debt с условиями закрытия.
6. **Honest unavailable.** Когда модуль не готов — статус «активируется в фазе X», не fake-data.
7. **Antidubli как первоклассный инвариант.** Идемпотентность, partial unique indexes, флаги участия в отчётах.
8. **Append-only audit logs.** Журналы пишутся, не правятся.
9. **Atomic transactions.** Critical-flow обёрнут в `db.transaction`.
10. **Spec-gap reporting.** Если промпт расходится с реальностью — агент сообщает до кодинга.

---

## Что дальше

Открой нужный sub-скилл из `skills/` и следуй его SKILL.md. Каждый sub-скилл самодостаточен.

Если не уверен какой нужен — спроси пользователя одним коротким вопросом: «Что у тебя сейчас на руках?» — и сопоставь ответ с decision flow выше.
