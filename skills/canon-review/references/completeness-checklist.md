# Completeness checklist для canon-review

> Сводный чек-лист для быстрой оценки полноты canon-set'а. Используй после прохождения по `audit-dimensions.md` для финальной самопроверки.

---

## Top-level (главный канон)

- [ ] Назначение продукта в одно предложение
- [ ] Аудитория явно описана
- [ ] Что НЕ делает продукт (out-of-scope)
- [ ] Список главных инвариантов (5-12 штук)
- [ ] Карта модулей с зависимостями
- [ ] Поток данных формализован
- [ ] Сквозные правила (audit, naming, multi-tenancy, error handling)
- [ ] Стек документирован отдельным файлом или ссылкой

## Per-module canon (на каждый модуль)

- [ ] Назначение в одно предложение
- [ ] Что делает / Чего не делает
- [ ] Главные сущности перечислены и разведены
- [ ] Lifecycle статусов полный (см. D1)
- [ ] Источник истины определён
- [ ] Связь с другими модулями (что отдаёт / берёт)
- [ ] Антидубли описаны (если применимо)
- [ ] Out-of-scope явно
- [ ] Тест-кейсы / acceptance (8-15 пунктов)

## Per-module ТЗ (на каждый модуль)

- [ ] Контекст-выжимка из канона
- [ ] Структура страниц / экранов (для UI-модулей)
- [ ] Поля схемы данных детально
- [ ] CHECK constraints
- [ ] Indexes (для performance)
- [ ] API-контракты на каждый endpoint
- [ ] Бизнес-формулы (если применимо)
- [ ] Lifecycle с пред-условиями переходов
- [ ] Антидубли на уровне схемы и сервиса
- [ ] Audit-logging описано
- [ ] Edge cases матрица
- [ ] UI-детали (state machine, responsive, a11y)
- [ ] Acceptance criteria measurable

## Cross-cutting

- [ ] Stack decision documented (`STACK.md`)
- [ ] Open questions extracted (`OPEN-QUESTIONS.md`)
- [ ] Глоссарий терминов (если домен сложный)
- [ ] Brand-voice consistent across all docs
- [ ] Версия / дата на каждом документе
- [ ] Журнал решений (как ROADMAP §11 в реальном проекте)

---

## «Зелёный свет» для перехода к следующей стадии

Каноны готовы для `design-canon` или `canon-roadmap` если:
- 0 Critical findings
- ≤ 2 Major findings (с планом фикса)
- Open questions не блокируют первую фазу

Каноны готовы для `canon-execute` (phase 1 implementation) если:
- 0 Critical findings
- 0 Major findings
- Все open questions для phase 1 закрыты
- Дизайн-система собрана (если phase 1 включает UI)

---

## «Красный свет»

Не двигаться дальше если:
- Есть хоть один Critical finding
- Есть Major findings без assigned-плана фикса
- Stack decision не документирован
- Multi-tenancy gaps присутствуют
- Денежная логика имеет hardcoded ставки или float-суммы

---

## Полезные команды для самопроверки

```bash
# Все каноны на месте?
ls docs/КАНОНЫ/*.md
ls docs/ТЗ/*.md

# Cross-links работают?
grep -rn "\\[.*\\](.*\\.md)" docs/ | head -50

# Stack documented?
test -f docs/STACK.md && echo "OK" || echo "MISSING"

# Open questions есть отдельно?
test -f docs/OPEN-QUESTIONS.md && cat docs/OPEN-QUESTIONS.md | head -20

# Brand-voice violations (грубая проверка)
grep -rn "!!!\\|🎉\\|🚀\\|революционн\\|самый лучший" docs/ | head
```

---

## Финальная мысль

Чек-лист — это **не замена прохождению по audit-dimensions**. Это финальная самопроверка после того как уже прошёл весь dimension audit. Если по чек-листу всё OK, но dimension audit не проводился — это false-positive, реально каноны не проверены.
