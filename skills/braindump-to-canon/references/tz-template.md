# Шаблон ТЗ модуля

> ТЗ — техническая спецификация модуля. Детальнее канона: содержит конкретные имена полей, API-контракты, lifecycle с переходами и проверками. Это документ для **разработчика**, чтобы он мог реализовать модуль без многократных уточнений.
>
> ТЗ не противоречит канону. ТЗ детализирует то что в каноне выражено принципами.

---

```markdown
# ТЗ: модуль «{Имя модуля}»

> Версия v1. При расширении функциональности — создать v2 с явной ссылкой на v1.

---

## 1. Контекст и scope

Краткая выжимка из канона §1-2 (назначение модуля, что делает, чего не делает). Цель этой секции — чтобы разработчик при чтении ТЗ не должен был параллельно открывать канон.

---

## 2. Структура страницы / экрана

### 2.1. Главная страница модуля

URL: `/{module-slug}`

Layout (если применимо):
- Заголовок и сводка
- Панель фильтров
- Панель быстрых действий
- Карточки верхнего уровня (KPI)
- Вкладки: {список}
- Боковая панель: {содержание}

### 2.2. Дочерние страницы

- `/{module-slug}/{sub}` — назначение, layout

---

## 3. Сущности и поля схемы

### 3.1. Таблица `{entity_name}`

Назначение: {одно предложение}

| Поле | Тип | Обязательность | Default | Назначение |
|---|---|---|---|---|
| `id` | serial | NOT NULL PK | auto | Технический идентификатор |
| `organization_id` | text | NOT NULL FK→organization | — | Multi-tenancy |
| `name` | text | NOT NULL | — | Имя сущности |
| `status` | enum | NOT NULL | 'draft' | Lifecycle |
| `amount_kopecks` | bigint | NOT NULL | — | Сумма в копейках |
| ... | ... | ... | ... | ... |
| `created_at` | timestamp | NOT NULL | now() | Аудит |
| `created_by_user_id` | text | NULL | — | Аудит |

**Индексы:**
- `(organization_id)` для multi-tenant фильтрации
- `(organization_id, status)` для list-фильтров
- `(organization_id, due_date)` для overdue checks

**CHECK constraints:**
- `amount_kopecks >= 0`
- `effective_to IS NULL OR effective_to > effective_from`
- (специфичные для домена)

**Unique constraints:**
- Partial unique index `(organization_id, name)` — name уникален в рамках org
- ...

### 3.2. Audit-log таблица `{entity_name}_audit_log`

Append-only лог изменений критических полей. Структура:
- `id` serial PK
- `organization_id` text NOT NULL
- `{entity_name}_id` integer NOT NULL FK
- `action` text NOT NULL — `'created' | 'updated' | 'status_changed' | ...`
- `old_status` / `new_status` (если применимо)
- `old_value_jsonb` / `new_value_jsonb` (для arbitrary fields)
- `reason` text
- `changed_at` timestamp NOT NULL
- `changed_by_user_id` text NOT NULL
- `metadata` jsonb

---

## 4. API-контракты

### 4.1. `POST /api/{module}/{entity}` — создание

**Request body (Zod):**
```typescript
const createSchema = z.object({
  name: z.string().min(1).max(120),
  amountKopecks: z.number().int().positive(),
  // ...
});
```

**Response 200:**
```typescript
{ data: Entity }
```

**Errors:**
- 400 — invalid body
- 401 — нет сессии
- 403 — нет прав (если применимо)
- 409 — конфликт (например, дубль)
- 422 — нарушение бизнес-правила (с понятным `error.message`)

**Логика handler'а:**
1. Session check → orgId
2. Validate Zod
3. Бизнес-проверки
4. INSERT в транзакции
5. Audit log
6. Return

### 4.2. `GET /api/{module}/{entity}?status=...&page=1&limit=50` — list

Query параметры:
- `status` — фильтр (опционально)
- `page` — пагинация (default 1)
- `limit` — размер страницы (max 100)

Response 200:
```typescript
{ data: Entity[], total: number, page: number, limit: number }
```

### 4.3. `GET /api/{module}/{entity}/[id]` — detail

### 4.4. `PATCH /api/{module}/{entity}/[id]` — update

С защитой: можно править только определённые поля в определённых статусах. Список:
- В status='draft' можно: name, description, amountKopecks
- В status='approved' можно: только description
- В status='closed' нельзя ничего

### 4.5. `POST /api/{module}/{entity}/[id]/transition` — смена статуса

Body:
```typescript
{ newStatus: '...', reason?: string }
```

Логика:
1. Проверить разрешённость перехода (state machine)
2. Проверить пред-условия для целевого статуса
3. UPDATE в транзакции + audit log

### 4.6. `DELETE /api/{module}/{entity}/[id]` — отмена / soft-delete

Soft-delete через `status='cancelled'`, не физическое удаление (если применимо).

---

## 5. Бизнес-формулы (если применимо)

### 5.1. Расчёт {название}

Формула:
```
result = func(input1, input2, ..., context)
```

Где:
- `input1` — определение
- `input2` — определение
- `context` — контекст: налоговый год, регион, etc.

**Округление:** до целой копейки по правилам {banker's | half-up | half-down}.

**Edge cases:**
- Если `input1 = 0` — результат `null` со статусом «нет базы»
- Если `func(...) < 0` — результат `max(0, ...)` (нет отрицательных обязательств)

**Примеры:**
- `func(100, 6, ...)` = 6
- `func(0, 6, ...)` = 0 со статусом «нет базы»

---

## 6. Lifecycle и переходы

(Подробнее чем в каноне — с пред-условиями каждого перехода)

```
draft → active:
  Pre-conditions:
    - amountKopecks > 0
    - dueDate >= today

draft → cancelled:
  Pre-conditions: none

active → completed:
  Pre-conditions:
    - paidKopecks >= amountKopecks
    - все child-сущности в финальных статусах

closed → reopened:
  Pre-conditions:
    - role in [director, admin]
    - reason.length >= 10
```

---

## 7. Обязательные поля по статусу

Матрица: для какого статуса какое поле обязательно (NOT NULL логически, не только в схеме).

| Поле | draft | active | completed |
|---|---|---|---|
| name | required | required | required |
| amountKopecks | optional | required | required |
| dueDate | optional | required | required |
| financeAccountId | optional | optional | required |

CHECK constraints в схеме обеспечивают это где возможно. Где невозможно (например, условная обязательность по другому полю) — service-level validation.

---

## 8. Антидубли (на уровне схемы и сервиса)

### 8.1. На уровне схемы

- Partial unique index: `UNIQUE (organization_id, source_entity_type, source_entity_id) WHERE source = 'external'`
- Partial unique index: `UNIQUE (organization_id, idempotency_key) WHERE idempotency_key IS NOT NULL`

### 8.2. На уровне сервиса

- Idempotency key обязателен для мутаций инициируемых из UI: формат `{module}-{entity}-{id}-{action}`
- Проверка race-conditions через `pg_advisory_xact_lock` на критических операциях

---

## 9. Аудит-логирование

Какие действия обязательно логируются:
- Создание сущности → `action='created'`
- Изменение критического поля → `action='updated'`, с old/new
- Смена статуса → `action='status_changed'`, с old/new status + reason
- Закрытие/переоткрытие периода (если применимо) → `action='closed'/'reopened'` + reason

Какие действия НЕ логируются (избыточно):
- Чтение
- Изменение метаданных (description, notes — если не критичны)

---

## 10. Edge cases и обработка

| Сценарий | Поведение |
|---|---|
| Пользователь дважды нажал submit | Idempotency key, второй запрос возвращает первый результат |
| Внешняя система недоступна | Локальное состояние сохраняется, запись `error` со статусом, retry-механизм |
| Закрытый период, попытка изменения | 409 с message «Период закрыт. Переоткройте через {action}» |
| Concurrency: две сессии одновременно создают | Advisory lock или uniqueIndex на ключ → второй получает ошибку или ждёт |
| Race на increment счётчика | Транзакция с lock |

---

## 11. Связи с другими модулями

### 11.1. Контракт с модулем {X}

- Что мы получаем: {описание}
- Через какой механизм: {hook / event / direct read}
- Что мы предоставляем: {описание}

### 11.2. Контракт с модулем {Y}
...

---

## 12. UI-детали

### 12.1. Брендовые правила
(ссылка на дизайн-канон — обязательная)

### 12.2. Состояния UI
- Loading: Skeleton
- Empty state: {описание}
- Error state: {описание}
- Success: toast / inline

### 12.3. Responsive
- Desktop ≥1024px: {layout}
- Tablet 640-1024px: {layout}
- Mobile <640px: {layout}

### 12.4. A11y
- Heading hierarchy: один h1, остальные h2/h3
- Focus rings, aria-labels на иконках
- Reduce-motion поддержка

---

## 13. Out-of-scope

Что НЕ делает этот ТЗ (откладывается / другой ТЗ / не делается):
- ...

---

## 14. Acceptance criteria

Итоговый чек-лист для приёмки:
- [ ] Все таблицы созданы со всеми CHECK constraints и индексами
- [ ] Все API-эндпоинты возвращают правильные коды и формат
- [ ] Lifecycle переходы работают согласно §6, запрещённые блокируются
- [ ] Антидубли §8 — невозможно создать дубль в нормальных условиях
- [ ] Audit-log пишется для всех событий §9
- [ ] UI соответствует §12
- [ ] Тест-кейсы из канона §12 — все проходят

---

## 15. Открытые вопросы

(Если что-то ещё не решено — фиксировать здесь, не пропускать)

- Q1: ...
```

---

## Заметки для генерации ТЗ

- ТЗ конкретнее канона. Если канон говорит «срок уплаты обязателен» — ТЗ говорит «`due_date date NOT NULL`».
- Не дублируй канон полностью. Ссылайся на него где уместно.
- Используй конкретные имена (snake_case для DB, camelCase для TS) — это начнёт работать в коде сразу.
- Минимум один пример формулы / API-вызова / edge case на каждую важную тему.
- Обновляй ТЗ когда меняется реализация — расхождение между ТЗ и кодом = тех-долг.
