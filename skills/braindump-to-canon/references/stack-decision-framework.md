# Stack Decision Framework

> Расширенный фреймворк для выбора и фиксации технологического стека в Phase 4 скилла `braindump-to-canon`.

Цель — не «рекомендовать модный стек», а провести **осознанный выбор с обоснованиями**, который потом не придётся переопределять через 6 месяцев.

---

## Принципы выбора

### 1. Stack-decisions длятся 3-7 лет

Стек выбирается под продукт, который проживёт минимум несколько лет. Выбор «сегодня модно» — без учёта поддержки, hiring, экосистемы — болезненно через 2 года.

Чек-вопросы:
- Этот фреймворк / БД / язык будет поддерживаться через 5 лет?
- На текущем рынке труда — легко ли нанять разработчика с этим стеком?
- Есть ли активная community и регулярные релизы?

### 2. Boring tech > exciting tech

Для production-критичных систем боринг-стек (PostgreSQL, Node.js, React) почти всегда правильнее, чем «новая горячая штука». Excitement окупается **только** если конкретная фича решает реальную проблему, которую boring не решает.

### 3. Один язык — меньше боли

Если есть выбор — лучше TypeScript end-to-end, чем TS-front + Go-back + Python-jobs. Полный type-safe path от UI до БД через одну экосистему.

Исключения: data-heavy ML (Python), система-критичные real-time (Rust/Go), legacy-интеграции.

### 4. Vendor-lock — вид tech-debt

Каждый managed-сервис который вы добавляете — это потенциальный lock. Это нормально для маленьких проектов и accelerates time-to-market. Но **осознанно**, не случайно.

Вопрос для каждого managed: «Если этот сервис закроется или поднимет цены 10x — что мы делаем?». Если ответа нет — это критическое риск, не tech-debt.

### 5. Type-safety — не опциональна для денег

Если продукт работает с деньгами / счетами / расчётами — TypeScript strict + ORM с типобезопасностью (Drizzle, Prisma в strict mode) — обязательно. Не «давайте быстро на JavaScript».

---

## Категории решений

### Frontend framework

**Defaults для современного web:**
- **Next.js 16+ (App Router, RSC)** — для SEO + perf + SSR-friendly
- **Vite + React** — если SPA без SSR-нужд
- **Remix** — альтернатива Next.js, хорош когда SSR-first философия
- **SvelteKit** — если есть Svelte-эксперты в команде, иначе hiring-риск

**Не выбирать:**
- Create React App (deprecated)
- Чистый React без framework для крупных приложений (роутинг + SSR придётся писать)

**Trade-offs:**
- Next.js — большой framework, lock-in на Vercel-features (хотя deploy-anywhere возможен)
- Remix — менее зрелая экосистема shadcn/forms
- SvelteKit — другая ментальная модель, не reusable экспертиза с React

### Frontend стилевая система

- **Tailwind CSS 4** — defaults для большинства, особенно с shadcn
- **CSS Modules** — если предпочитают традиционный CSS
- **Stitches / Vanilla Extract** — type-safe CSS-in-TS, но runtime cost

**Решение про токены:** независимо от выбора — все цвета/размеры/радиусы через CSS-переменные. Это первое что делает `design-canon` скилл.

### Backend stack

**Defaults:**
- **Next.js Route Handlers** — если фронт на Next.js, монорепо
- **Hono** — если хотим выделенный API, легковесный
- **NestJS** — для крупных enterprise-проектов с DI и архитектурными слоями

**Не выбирать без причины:**
- Express.js — устаревшая концепция middleware-pyramid
- Fastify — быстрый, но Hono обогнал по DX

### Auth

**Defaults:**
- **BetterAuth** — современный, type-safe, organization plugin
- **Clerk** — managed, удобный для быстрого старта, vendor-lock
- **Auth.js (NextAuth)** — широко используется, но DX слабее BetterAuth

**Не делать:**
- Свой auth с нуля. Никогда. Слишком много мест для ошибок.

### База данных

**Defaults:**
- **PostgreSQL 16+** — для большинства проектов. Зрелая, типы, json, partial indexes, advisory locks
- **SQLite** — для embedded / local-first
- **MySQL/MariaDB** — если уже есть инфраструктура, иначе PG лучше

**Не выбирать как первичный store:**
- MongoDB — для денежных систем плохо (отсутствие транзакций исторически, ACID-режим неполный)
- Redis — это cache, не primary store
- DynamoDB — если не AWS-only и не serverless-only

### ORM / Query builder

**Defaults:**
- **Drizzle ORM** — type-safe, лёгкий, миграции через SQL
- **Prisma** — больше абстракции, удобный, но N+1 ловушки чаще
- **Kysely** — query-builder без ORM

**Не выбирать:**
- TypeORM — устарел концептуально
- Sequelize — устарел
- Raw SQL без layer — непрактично для крупных проектов

### Validation

- **Zod 3+** — defacto стандарт. Используется везде: API, формы, env-vars
- **Valibot** — легче по bundle, но меньше экосистемы

### State management (frontend)

**Когда нужен:**
- Multi-page state (например, чат, корзина) → **Zustand** или **Jotai**
- Server state (cache, refetch) → **TanStack Query** (использовать всегда для async-data)
- Form state → **react-hook-form** + Zod

**Когда не нужен:**
- Простые странички с локальным state → useState достаточно
- RSC + Next.js — много state живёт на сервере

### File storage

- **S3-compatible** (AWS S3, R2, MinIO) — defaults для production
- **Local filesystem** — только для dev / single-server
- **Vercel Blob / Supabase Storage** — managed, vendor-lock

### Email

- **Resend** — современный, хороший DX, Email-as-React
- **SendGrid / Postmark** — традиционные
- **AWS SES** — самостоятельная инфраструктура, дёшево, но больше работы

### LLM / AI integration (если есть)

- **OpenRouter** — provider-neutral, можно менять модели без переписывания клиента
- **Прямой Anthropic / OpenAI SDK** — если нужны провайдер-специфичные фичи (например, prompt caching, tool use specifics)
- **Vercel AI SDK** — wrapper, удобный для streaming

### Деплой

**Зависит от стека:**
- Next.js → **Vercel** (managed) или **Railway** (managed cheaper) или **Coolify / Dokku / Docker** (self-hosted)
- Чистый Node → **Railway**, **Fly.io**, **Render**, **VPS+Docker**
- Statically prerendered → **Cloudflare Pages**, **Netlify**, **Vercel**

**Принцип:** для MVP — managed. Когда счёт превышает $200/мес — пересмотр. Self-host даёт контроль, но требует DevOps-времени.

### Background jobs / cron

- **Inline Next.js Route Handler с cron-trigger** (Vercel Cron, Railway Cron) — для простых задач
- **Trigger.dev** — managed background jobs с retry
- **BullMQ + Redis** — self-hosted

### Monitoring / errors

- **Sentry** — defacto стандарт error tracking
- **PostHog** — analytics + sessions
- **Axiom / Logtail** — структурированные логи

### Testing

Этот пункт деликатный. Чёткий совет:
- **Vitest** для unit-тестов — быстрее Jest, ESM-нативный
- **Playwright** для E2E — лучше Cypress сегодня
- **MSW** для мокирования API в тестах
- **Никаких snapshot-тестов** — превращаются в шум

Но: **тесты пишем избирательно**. Для каждого проекта определи где они окупятся:
- Денежные расчёты → 100% покрытие unit-тестами
- API-контракты → integration-тесты
- UI — много E2E дороже slow-roll, скриншоты + manual для starter
- Pure type-safety на TypeScript заменяет много тестов на naive ошибки

---

## Шаблон Stack Decision Document

```markdown
# Stack Decision

> Документ зафиксирован на дату {YYYY-MM-DD}. Решения о смене стека — через явный update этого файла + запись в журнал решений.

---

## Frontend

**Frontend framework:** Next.js 16 (App Router, RSC)
- Почему: SSR + RSC + perfect TypeScript-DX + большая экосистема
- Альтернативы рассмотрены: Remix (меньше экосистема shadcn), SvelteKit (hiring-риск)
- Trade-off: Vercel-friendly но deploy-anywhere возможен через Docker

**UI library:** shadcn/ui + Tailwind CSS 4
- Почему: copy-paste components без runtime npm-зависимости, полный контроль над стилями

**Forms:** react-hook-form + Zod
- Почему: type-safe end-to-end (Zod schema используется и в API)

**State (client):** Zustand для глобального, TanStack Query для server-state

## Backend

**API:** Next.js Route Handlers (монорепо)
- Альтернатива: Hono в отдельном репо (не выбрано — overhead монорепо vs split не оправдан для нашего размера)

**Auth:** BetterAuth + organization plugin
- Почему: type-safe, organization-plugin критичен для multi-tenant

**Validation:** Zod 3

## Data

**Primary DB:** PostgreSQL 18
- Почему: ACID, JSON, partial indexes, advisory locks, json operators

**ORM:** Drizzle
- Почему: type-safe, миграции через SQL (нет magical layer)

**Storage:** S3-compatible (Cloudflare R2 — дёшево, S3-API)

## Infrastructure

**Деплой:** Railway
- Почему: managed Postgres + Next.js + cron в одном дашборде, дёшево для SMB

**Monitoring:** Sentry для errors, Axiom для логов
- Почему: лучший DX в категории

**Email:** Resend

**LLM:** OpenRouter с OpenAI SDK (provider-neutral)

## Pinned versions

```json
{
  "next": "16.0.x",
  "react": "19.0.x",
  "typescript": "5.5.x",
  "drizzle-orm": "0.40.x",
  "tailwindcss": "4.0.x",
  "zod": "3.24.x"
}
```

---

## Полный список решений-к-замыканию

- [ ] Локализация: i18n или single-language?
- [ ] Дата/время: utc-only? с какой библиотекой работаем (date-fns vs Day.js)?
- [ ] Деньги: одна валюта или мульти? Хранение integer-копейки?
- [ ] Регулировка: GDPR/152-ФЗ-compliance?
- [ ] Backup-стратегия БД?

Это всё нужно решить до начала Phase 6 (per-module canons), потому что каноны опираются на эти решения.
```

---

## Финальное правило

**Stack decision — единственное решение этой фазы которое нельзя переписать без боли**. Каноны можно переписать. ТЗ можно расширить. Стек после 100k строк кода — переписывается полным rewrite'ом проекта.

Не спеши. Если пользователь говорит «решим потом» — вернись и заставь решить. Открытые вопросы по стеку идут в `OPEN-QUESTIONS.md` с пометкой **«BLOCKING для начала Phase 6»**.
