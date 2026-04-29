# Матрица измерений canon-review

> Полная матрица 15+ измерений с конкретными контрольными вопросами для каждого. Используется в Phase 2 скилла `canon-review`.

Для каждого измерения проходишь по всем вопросам. Если ответ «не описано в каноне» — это finding.

---

## D1. Lifecycle completeness

Для каждой сущности с состояниями (status enum):

- [ ] Перечислены все статусы (draft / active / completed / cancelled / archived / etc.)?
- [ ] Для каждого статуса описано что разрешено / запрещено?
- [ ] Перечислены все легальные переходы (`draft → active`, `active → completed`)?
- [ ] Запрещённые переходы явно указаны (например, нельзя `draft → completed` напрямую)?
- [ ] Что происходит при попытке запрещённого перехода — error, silent ignore, retry?
- [ ] Если есть «closed»-состояние — как переоткрывается? Кем? С какими условиями?
- [ ] Soft-delete (`cancelled`) vs hard-delete — выбор сделан и обоснован?
- [ ] При cancellation что происходит со связанными сущностями (cascade или nullify)?
- [ ] Concurrent state-transitions защищены (locks, optimistic concurrency)?

**Severity rule:** отсутствие любого из первых 4 пунктов = Critical. Остальные = Major.

---

## D2. Antidubli и идемпотентность

- [ ] Идентифицированы места где одна логическая операция может прийти из 2+ источников?
- [ ] Для каждого такого места — описан механизм антидубля?
- [ ] Idempotency keys — формат описан? Детерминированный (можно повторно сгенерировать)?
- [ ] Partial unique indexes на ключевых полях?
- [ ] Race conditions для increment'ов (счётчики, sequence) защищены через advisory locks?
- [ ] При retry duplicate key — gracefully вернуть существующую запись или error 409?
- [ ] Если используются webhook'и от внешних — антидубль продуман?

**Severity rule:** отсутствие антидубля для денежной операции = Critical. Для не-критичной = Major.

---

## D3. Источники истины

- [ ] Для каждого типа данных явно указан **единственный** источник истины?
- [ ] Производные / агрегированные данные явно отмечены как «derived from X»?
- [ ] Read-only views отделены от writable таблиц?
- [ ] Аналитический слой явно read-only по отношению к operational?
- [ ] Cache имеет явный TTL и invalidation logic?
- [ ] Backup-источники для disaster recovery определены?

**Severity rule:** конфликт «X — источник» в одном каноне и «X — derived» в другом = Critical (это означает что один из них врёт).

---

## D4. Multi-tenancy

- [ ] Все главные сущности имеют `organization_id` (или эквивалент)?
- [ ] Все API-endpoints фильтруют по orgId из session?
- [ ] JOIN'ы между таблицами безопасны (orgId через всю цепочку)?
- [ ] «Глобальные» сущности (например, справочники) явно отмечены как cross-org?
- [ ] Cross-org permission модель определена (например, super-admin может ли видеть всё)?
- [ ] Cross-org leak через ID-guessing предотвращён (например, все queries `WHERE id=X AND organizationId=orgId`)?
- [ ] Тесты на cross-tenant leak предусмотрены?

**Severity rule:** любой gap здесь = Critical. Cross-tenant leak — катастрофический баг.

---

## D5. Деньги

(Только если продукт имеет денежную логику)

- [ ] Все суммы в integer-копейках (или центах для валют без копеек)?
- [ ] Валюта определена однозначно для каждой суммы (поле `currency` или single-currency-system)?
- [ ] Конвертация валют формализована? (если мульти-валютность)
- [ ] Округление формализовано (banker's / half-up / half-down) и **последовательно** применяется?
- [ ] Двойная запись или single-entry — выбор сделан и обоснован?
- [ ] Если single-entry — как обеспечивается consistency (sum баланса)?
- [ ] Журнал денежных движений — append-only, иммутабельный?
- [ ] Закрытые периоды защищены от изменений?
- [ ] Reverse-операции (storno, refund) — отдельная сущность, не UPDATE existing?

**Severity rule:** float для денег = Critical. Mutable journal = Critical. Прочее = Major.

---

## D6. Audit logging

- [ ] Список обязательно логируемых действий определён?
- [ ] Структура audit-log универсальна (одинаковая для всех модулей)?
- [ ] Поля audit-log: who / when / what / old / new / reason / metadata?
- [ ] Append-only enforced (DB-level CHECK или триггер защищающий от UPDATE/DELETE)?
- [ ] Retention policy определена (бесконечно, 1 год, 7 лет)?
- [ ] Доступ к журналу — какая роль? Read-only?
- [ ] Можно ли удалить персональные данные из старых записей (right-to-be-forgotten compliance)?

**Severity rule:** отсутствие append-only защиты = Major. Отсутствие retention policy = Minor.

---

## D7. Authorization model

- [ ] Роли определены и описаны (что может делать каждая)?
- [ ] Permissions per action — список явный (CRUD на каждой сущности + custom actions)?
- [ ] Critical actions (close period, change tax regime, delete user) требуют доп. подтверждения?
- [ ] Cross-org permission leak невозможен на уровне API-pattern'а?
- [ ] Default permissions для нового пользователя — restrictive (least privilege)?
- [ ] Permissions хранятся в БД или hardcoded? (БД лучше для гибкости)
- [ ] Audit-log для permission changes ведётся?

---

## D8. Versioning справочников и правил

(Если есть нормативные данные — налоговые ставки, тарифы, лимиты)

- [ ] Справочники имеют версионность (по году / дате)?
- [ ] Поля `validFrom` / `validTo` обязательны?
- [ ] При расчёте за прошлый период — берётся версия справочника актуальная **на тот период**?
- [ ] Изменение справочника НЕ пересчитывает прошлые периоды (immutable history)?
- [ ] Auditable — кто и когда изменил справочник?
- [ ] Правила хранятся в `rules_registry` (или эквивалент), не в коде?

**Severity rule:** hardcoded ставки/лимиты в коде = Major (всегда плохо для регулируемых доменов).

---

## D9. Error handling и user feedback

- [ ] Все error-cases в lifecycle обработаны (не silent fail)?
- [ ] User-facing error messages определены (русский / английский — на языке продукта)?
- [ ] Retryable vs terminal errors различаются?
- [ ] Что показываем когда внешняя система недоступна — определено?
- [ ] HTTP-коды правильные: 400 / 401 / 403 / 404 / 409 / 422 / 500?
- [ ] Server logs включают достаточно контекста для debug (но без leak'а PII)?
- [ ] Sentry / error-tracker интеграция предусмотрена?

---

## D10. Cross-module integration

Для каждой пары модулей где есть взаимодействие:

- [ ] Канон A явно описывает что отдаёт модулю B?
- [ ] Канон B явно описывает что берёт у модуля A?
- [ ] Контракт на стыке формализован (data shape, frequency, error handling)?
- [ ] Что при failure A — как ведёт себя B?
- [ ] Антидубль на стыке (см. D2) описан в **обоих** канонах одинаково?
- [ ] Eventual consistency или strict consistency — выбор?

---

## D11. Data quality / unavailability

- [ ] Когда данные недоступны (модуль не построен, справочник пуст, недостаточно истории) — что показываем?
- [ ] Partial data states обработаны?
- [ ] «Honest unavailable» vs fake-data — какая стратегия?
- [ ] Drilldown в источник недоступности (objejective `unavailableReason`)?
- [ ] User'у явно сказано что показатель недоступен?

---

## D12. Migrations и backward compatibility

- [ ] Schema-evolution стратегия описана?
- [ ] Backfill для существующих данных продуман (или явно skipped с обоснованием)?
- [ ] Breaking changes в API — как обрабатываем (versioning, deprecation period)?
- [ ] Rollback-стратегия для миграций?
- [ ] Pre/post-deploy миграции различаются?

---

## D13. Performance и scale

- [ ] Тяжёлые queries идентифицированы (отчёты, агрегаты, full-table scans)?
- [ ] Indexes для critical queries определены?
- [ ] N+1 ловушки выявлены (e.g. list page с per-row JOIN)?
- [ ] Pagination на всех list-endpoints (max limit определён)?
- [ ] Background jobs для тяжёлых операций (отчёты, batch-импорт)?
- [ ] Rate limiting на API?
- [ ] Caching strategy (где, на какой срок)?

---

## D14. Testing и acceptance

- [ ] Тест-кейсы покрывают happy path?
- [ ] Edge cases (пустой ввод, max values, concurrency) — отдельно?
- [ ] Регрессионные сценарии описаны?
- [ ] Acceptance criteria measurable (не «работает хорошо», а «X запрос возвращает результат за <500ms»)?
- [ ] Manual QA scenarios для UI flows предусмотрены?

---

## D15. Documentation hygiene

- [ ] Brand-voice соблюдается (calm, structural, no эмодзи, sentence case)?
- [ ] Структура канонов одинакова (см. canon-template.md)?
- [ ] Cross-links между канонами работают?
- [ ] Open-questions явно вынесены в отдельный файл?
- [ ] Глоссарий терминов есть (если домен сложный)?
- [ ] Примеры расчётов / сценариев включены где сложная логика?

---

## D16. Security (если применимо)

- [ ] Хранение паролей через bcrypt/argon2 (не plain, не MD5/SHA1)?
- [ ] CSRF-protection на mutating endpoints?
- [ ] XSS-protection (output encoding везде где user content рендерится)?
- [ ] SQL-injection prevented (parameterized queries / ORM, не string concat)?
- [ ] Rate limiting на login / password-reset?
- [ ] Sensitive data masking в логах (PII / payment info)?
- [ ] HTTPS enforced (не HTTP)?
- [ ] Secrets management (env vars, secret manager — не в коде)?

---

## Custom dimensions

Для специфичных доменов могут быть дополнительные измерения. Примеры:

### D17. Регуляторика (если регулируемый домен)
- [ ] Compliance с применимым законодательством (152-ФЗ, GDPR, HIPAA) описан?
- [ ] Data residency requirements соблюдены?
- [ ] Pre-required certifications спланированы?

### D18. Real-time (если real-time-критичный продукт)
- [ ] Low-latency пути идентифицированы?
- [ ] WebSocket / SSE infrastructure спланирована?
- [ ] Reconnection logic продумана?

### D19. ML / AI (если есть ML-компоненты)
- [ ] Hallucination risks для LLM identified?
- [ ] Prompt-injection prevention?
- [ ] Какие данные пойдут в провайдера — задокументировано?
- [ ] Fallback при недоступности LLM?

---

## Priority matrix

Финальная оценка каждого finding'а:

| Severity | Кто блокируется | Когда фиксить |
|---|---|---|
| 🔴 Critical | Production / pre-launch | Перед началом phase 1 — обязательно |
| 🟡 Major | Phase implementation | До конца phase 1 — желательно |
| 🟢 Minor | Documentation quality | Постепенно по мере touch |
| 💡 Suggestion | Quality improvement | Когда есть свободное время |

---

## Использование матрицы

В Phase 2 скилла `canon-review`:

1. Взять конкретный канон (например, `Канон_Финансы.md`)
2. Пройти по каждому измерению (D1-D15+)
3. Для каждого измерения — checkmark пройти все вопросы
4. Найденные пробелы записать как findings с указанием:
   - Где (канон + раздел)
   - Что (точное описание пробела)
   - Severity
   - Направление фикса
5. Перейти к следующему канону, повторить

Среднее время full-audit одного канона — 1-2 часа. Не сокращай.
