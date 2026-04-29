---
name: design-canon
description: Use when creating or refining a design system for a product. Generates a complete token-driven design system (colors, typography, spacing, radii, shadows, motion, icons), enforces restraint principles (single brand hue, size-driven hierarchy, real imagery over illustrations, anti-AI-slop discipline), and produces a runnable HTML preview gallery so the design can be reviewed visually before any production code is written. Inspired by Huashu design discipline plus practical canon-driven workflow additions.
---

# design-canon

🚀 Стадия 3 в pipeline canon-dev: **дизайн-система с HTML-preview**

> **Inspired by:** principles of [huashu-design](https://github.com/alchaincyf/huashu-design) (anti AI-slop discipline, restraint over elaboration, depth through structure not color). This skill expresses those principles in original prose, augmented with token-driven architecture and a runnable HTML-preview gallery — so the system can be **visually verified** before being implemented in production code.

---

## Когда использовать

- Новый продукт без дизайн-системы — собрать с нуля
- Существующая дизайн-система фрагментирована — пересобрать в когерентную
- Бренд-материалы есть (логотип, цветовое предпочтение, гайдлайн) — нужно их формализовать в токены
- Нужен HTML-preview существующей дизайн-системы для презентации команде или клиенту

## Когда НЕ использовать

- Каноны продукта ещё не сделаны → сначала `braindump-to-canon` (дизайн зависит от продукта)
- Нужно реализовать дизайн в коде → это `canon-execute` (с готовой дизайн-системой)
- Аудит существующего UI на соответствие дизайну → это `canon-verify`

---

## Философия скилла

`design-canon` строится на одном корневом принципе: **сдержанность как сила, не как пустота**.

Это означает:
- Один brand-цвет вместо палитры
- Размер вместо веса для иерархии
- Реальные продуктовые изображения вместо illustrations
- Asymmetric layouts вместо uniform grids
- Depth через слои и тени вместо градиентов и цвета
- Анимации subtle и быстрые вместо bouncy и playful

Каждое из этих правил отказывается от «привычного для UI» в пользу того что заметно как **профессиональное**. AI-сгенерированные UI обычно ошибаются именно тут — добавляют больше цветов, больше градиентов, больше иконок, больше анимаций. `design-canon` форсирует обратное.

### Anti AI-slop list

Эти паттерны **запрещены** в любой дизайн-системе создаваемой через этот скилл:

❌ Многоцветные градиенты на CTA (purple→pink, blue→cyan)
❌ Эмодзи как иконки в production UI
❌ Карточки с цветной полосой по левому краю
❌ Vector illustrations людей или абстрактных бизнес-сцен
❌ Inter as display font (используй с разнообразием — serif, geometric, neo-grotesque)
❌ CSS-шейпы вместо реальных продуктовых скриншотов
❌ Bouncy animations (overshoot, spring)
❌ Глобальные scroll-jacking effects
❌ Bold для всего подряд вместо размерной иерархии
❌ Random hover-microinteractions без orchestration

### Что взамен

✅ Один brand-цвет глубоко проработанный (10+ оттенков)
✅ Lucide / Phosphor иконки (или подобные line-системы) с одинаковым stroke-weight
✅ Карточки через `1px border + soft shadow + tonal background shift`
✅ Реальные продуктовые скриншоты в browser-frame обёртках
✅ Geometric humanist sans для display + system body (Onest / Manrope / Inter — но не для всего)
✅ Asymmetric bento grids с разной visual weight
✅ Subtle 140-220ms ease-out transitions, без overshoot
✅ Orchestrated page-load (один coherent reveal), не scattered hover-effects

---

## Структура процесса

```
Phase 0 — Fact verification (🛑 проверка реальности продукта/бренда)
   ↓
Phase 1 — Discovery (что есть от пользователя)
   ↓
Phase 2 — Brand parameters (один цвет, шрифт, общий тон)
   ↓
Phase 3 — Token architecture (полный набор CSS-переменных)
   ↓
Phase 4 — Component primitives (cards, buttons, inputs, etc. в токенах)
   ↓
Phase 5 — Junior Pass HTML (🛑 черновик preview, показать пользователю)
   ↓
Phase 6 — Full Pass HTML preview gallery (полная версия после feedback)
   ↓
Phase 7 — Brand voice document
   ↓
Phase 8 — Iteration (правки по preview)
   ↓
Phase 9 — Hand-off (готовая дизайн-система к canon-roadmap)
```

**🛑 = hardline checkpoint** — фазы где обязательно остановиться и дождаться явного подтверждения пользователя. Не просто «спросил и продолжил», а **реально подождал ответ**. Эти точки защищают от closed-loop auto-completion.

---

## Phase 0 — Fact verification (🛑 critical)

**Это новая фаза, добавленная по шаблону huashu-design.** Должна выполняться **до любых вопросов про дизайн**.

### Зачем

AI-агенты склонны хайлюциновать дизайн-параметры. Пользователь говорит «сделай для Stripe» — агент выдумывает их цвета вместо того чтобы достать реальные. Результат — generic-template который выглядит как Stripe но не Stripe.

Step #0 защищает: **сначала верификация фактов, потом дизайн**.

### Что делать

Если в задаче упомянут конкретный продукт / компания / существующий бренд — **первым делом верифицируй**:

1. **Поиск official assets:**
   - Главный сайт продукта
   - Press kit / brand guidelines page
   - GitHub если open-source (часто содержит logo + colors)
   - App store если продукт mobile

2. **Поиск product imagery:**
   - Реальные screenshots продукта
   - Не stock-photo, не AI-illustrations

3. **Поиск social proof:**
   - LinkedIn / Twitter компании — реальные ли это люди?
   - Customer reviews
   - Это реальная компания или гипотетическая?

4. **Если НЕ можешь найти official assets:**
   - Спроси пользователя: «Это существующий продукт или новый? Если существующий — где можно скачать brand-материалы?»
   - **Не угадывай цвета и стиль.** Лучше явный «не нашёл» чем правдоподобная выдумка.

### 🛑 Checkpoint

Перед переходом к Phase 1 — обязательно явная фраза пользователю:

```
Phase 0 verification:
- {Product name}: {existing/new}
- Official brand materials: {found at URL / not found}
- Product imagery: {found / need to be created}
- Verification confidence: {high / medium / low — почему}

Готов начать Phase 1 Discovery?
```

И **дождись ответа**. Не продолжай молча.

### Когда Phase 0 можно skipnut'ь

- Продукт совершенно новый, без брендинга → Phase 0 пуст, переход сразу к Phase 1
- Пользователь явно говорит «придумай brand с нуля» → skip
- Это итеративная сессия по существующей дизайн-системе → skip (но re-verify когда меняется brand)

Во всех остальных случаях — **не пропускай Phase 0**. Стоимость одна минута, защита от слопа массивная.

---

## Phase 1 — Discovery

Запроси и прочитай всё что есть:

- Логотип / brand mark
- Существующие цвета (если есть)
- Презентация / pitch deck
- Скриншоты конкурентов которые нравятся
- Скриншоты конкурентов которые НЕ нравятся — это критично
- Тон / voice предпочтения
- Любые ограничения (cyrillic-support, accessibility-first, b2b-conservative, b2c-warm)

Если ничего нет — задай 5-7 вопросов:

1. Какое **одно слово** описывает желаемое впечатление от продукта? (calm / sharp / warm / serious / playful)
2. B2B или B2C? Уровень формальности?
3. Аудитория — старше или младше 35?
4. Есть ли отраслевые traditions которые мы соблюдаем или ломаем?
5. Один или два контекста использования (только web? + mobile? + слайды?)
6. Поддержка тёмной темы — нужна сразу или потом?
7. Что из существующих продуктов вызывает положительные ощущения?

Не задавай 30 вопросов. 5-7 целенаправленных.

---

## Phase 2 — Brand parameters

Зафиксируй три параметра:

### 2.1. Brand hue (один цвет)

Выбери **один** цвет который будет brand-цветом. Это деликатное решение:

- Серьёзный B2B → deep navy / cobalt / forest green
- Современный SaaS → bright blue / indigo
- Финансовый → conservative blue / dark green
- Health-tech → calm teal или green
- Creative-tools → rich purple или orange
- Tech-infrastructure → slate / charcoal с accent

**Не выбирай:**
- Pure red (агрессия), pure yellow (низкая разборчивость на белом)
- Multi-color (нарушает принцип сдержанности)
- Trending colors which will date (millennial pink, gen-z mint) — выбирай вне-временное

После выбора — построй 11-step shade scale:
```
{brand}-25, -50, -100, -200, -300, -400, -500, -600, -700, -800, -900
```

Где -900 — самый тёмный, -25 — almost-white tint. Используй HSL-shifts или OKLCH-space для математически согласованной шкалы.

### 2.2. Typography

**One font family principle:** одна семья шрифтов для всего, иерархия через размер.

Выбор:
- **Onest** (Google Fonts) — geometric humanist, отличная Cyrillic поддержка
- **Manrope** — похожий, чуть более рукописный
- **Inter** — defacto стандарт SaaS (но overused)
- **Geist** (Vercel) — современный, чистый
- **Söhne / Söhne Mono** (commercial) — премиум-уровень

Для display можно отдельную семью если хочется magazine-уровня:
- **Fraunces / Newsreader** — modern serif с inktraps
- **PP Editorial / GT Sectra** — premium serif (commercial)

**Веса:** только 300/400/500. Bold (600+) — запрещён в системе. Иерархия через size + color + space.

### 2.3. Mood

Зафиксируй mood в 2-3 прилагательных. Это будет backstop для всех решений:

> "Calm, structural, slightly formal" → выбираем navy/cobalt + Onest + 1px hairlines + soft shadows
> "Sharp, modern, precise" → выбираем deep blue/black + Geist + sharp corners + crisp lines
> "Warm, approachable, trustworthy" → выбираем deep green + serif display + organic radii + softer shadows

---

## Phase 3 — Token architecture

Это сердце дизайн-системы. Все решения формализованы как CSS-переменные. **Никаких hardcoded цветов или размеров в компонентах** — только references на токены.

Используй шаблон `templates/colors_and_type.css` как стартовую точку. Адаптируй под выбранные brand-параметры. Финальная структура:

```css
:root {
  /* === Brand color scale (11 steps) ============================ */
  --brand-25:  #...;
  --brand-50:  #...;
  --brand-100: #...;
  --brand-200: #...;
  --brand-300: #...;
  --brand-400: #...;
  --brand-500: #...;
  --brand-600: #...;
  --brand-700: #...;
  --brand-800: #...;
  --brand-900: #...;

  /* === Neutrals (tinted toward brand) ========================== */
  --bg:         #FFFFFF;
  --bg-soft:    #...; /* very pale brand-tint */
  --bg-panel:   #...;
  --line:       #...;
  --line-soft:  #...;

  --fg-1: #...; /* primary text — same as brand-900 */
  --fg-2: #...; /* body */
  --fg-3: #...; /* caption */
  --fg-4: #...; /* placeholder/disabled */

  /* === Semantic ================================================= */
  --success:    #...;
  --success-bg: #...;
  --warning:    #...;
  --warning-bg: #...;
  --danger:     #...;
  --danger-bg:  #...;
  --info:       var(--brand-500);
  --info-bg:    var(--brand-100);

  /* === Type ==================================================== */
  --font-sans: '{font}', system-ui, -apple-system, sans-serif;
  --font-mono: ui-monospace, 'SF Mono', Menlo, Consolas, monospace;

  --weight-light:   300;
  --weight-regular: 400;
  --weight-medium:  500;

  --t-display:    72px;
  --t-h1:         44px;
  --t-h2:         32px;
  --t-h3:         24px;
  --t-h4:         20px;
  --t-h5:         18px;
  --t-body-lg:    17px;
  --t-body:       15px;
  --t-body-sm:    14px;
  --t-caption:    13px;
  --t-meta:       12px;
  --t-num-xl:     40px; /* large numbers in dashboards */
  --t-num-lg:     28px;

  --leading-tight:  1.15;
  --leading-snug:   1.3;
  --leading-normal: 1.5;
  --leading-loose:  1.7;

  --tracking-tight:  -0.02em;
  --tracking-normal: 0;
  --tracking-wide:   0.04em;

  /* === Radii ==================================================== */
  --r-sm:   6px;
  --r-md:   10px;
  --r-lg:   16px;
  --r-xl:   20px;
  --r-2xl:  28px;
  --r-pill: 999px;

  /* === Shadows (3 levels, brand-tinted) ========================= */
  --shadow-1: 0 1px 2px rgba(brand-rgb, 0.04),
              0 0 0 1px rgba(brand-rgb, 0.04);
  --shadow-2: 0 4px 14px rgba(brand-rgb, 0.06),
              0 0 0 1px rgba(brand-rgb, 0.04);
  --shadow-3: 0 10px 30px rgba(brand-rgb, 0.10),
              0 0 0 1px rgba(brand-rgb, 0.05);
  --shadow-focus: 0 0 0 4px rgba(brand-500-rgb, 0.18);

  /* === Spacing (4-base scale) =================================== */
  --s-1:  4px;
  --s-2:  8px;
  --s-3:  12px;
  --s-4:  16px;
  --s-5:  20px;
  --s-6:  24px;
  --s-8:  32px;
  --s-10: 40px;
  --s-12: 48px;
  --s-16: 64px;
  --s-20: 80px;

  /* === Motion =================================================== */
  --ease-out:    cubic-bezier(0.22, 0.61, 0.36, 1);
  --ease-in-out: cubic-bezier(0.65, 0, 0.35, 1);
  --dur-fast:    140ms;
  --dur-base:    220ms;
  --dur-slow:    420ms;
}
```

Все эти токены — обязательны. Не пропускай категории. Если для проекта что-то не нужно (например, --t-num-xl) — оставь токен с разумным default'ом, не используй в компонентах.

---

## Phase 4 — Component primitives

С токенами на руках — формализуй базовые компоненты на уровне CSS-классов или React-компонентов с props. Минимум:

### Cards

```css
.card {
  background: var(--bg); /* white */
  border: 1px solid var(--line);
  box-shadow: var(--shadow-1);
  border-radius: var(--r-lg);
  padding: var(--s-6);
  transition: border-color var(--dur-base) var(--ease-out),
              box-shadow var(--dur-base) var(--ease-out);
}
.card:hover {
  border-color: var(--brand-200);
  box-shadow: var(--shadow-2);
}
```

### Buttons

```css
.btn-primary {
  background: var(--brand-700);
  color: var(--bg);
  padding: var(--s-3) var(--s-6);
  border-radius: var(--r-md);
  font-size: var(--t-body);
  font-weight: var(--weight-medium);
  transition: background-color var(--dur-fast) var(--ease-out);
}
.btn-primary:hover { background: var(--brand-800); }
.btn-primary:active { opacity: 0.92; }
```

### Inputs, badges, pills, etc.

Аналогично — все через токены, минимум hardcode.

### Иконки

Стандартизируй на одной icon-системе:
- **Lucide** — defacto стандарт open-source (используется в shadcn)
- **Phosphor** — больше вариантов weights
- **Tabler** — нейтральная

Правила:
- 16px / 20px / 24px размеры
- Stroke 1.5-2px
- Цвет через `currentColor` (наследуется от текста)
- Когда стоят на цветной плашке — `--brand-100` плашка `rounded-full` `40×40px` + иконка `--brand-700`

---

## Phase 5 — Junior Pass HTML (🛑 show-early checkpoint)

**Перед** генерацией полной HTML-galley — сначала **Junior Pass**: упрощённая версия с явными placeholder'ами для всего что не уверен.

### Зачем Junior Pass

Полная gallery — 15-20 HTML-страниц, генерация ~1-2 часа. Если в Phase 2 brand-параметрах есть ошибки — все страницы придётся пересобирать. Junior Pass решает: показать каркас на одной странице, получить feedback, **только после** делать полный.

### Что включает Junior Pass

Одна HTML-страница `preview/_junior.html`:
- Brand-color scale (только swatches)
- 3-4 примера типографики (display + h1 + body + caption)
- 2-3 sample-карточки
- 1 button-примеры
- 1 example-mockup (например, hero-section landing'а)

Явные placeholder'ы:
- «{здесь будет реальный логотип}»
- «{здесь real product screenshot}»
- «{spacing finalize after Phase 6}»

### 🛑 Checkpoint

Покажи Junior Pass пользователю:

```
Junior Pass preview готов: preview/_junior.html
Откройте в браузере.

Проверьте:
1. Brand-color правильный? (#XXXX)
2. Шрифт ощущается правильным?
3. Tone — calm/sharp/warm — соответствует ожиданиям?
4. Что-то выглядит явно неправильно?

Готов делать Full Pass после ваших правок.
```

Дождись ответа. Если есть правки — внеси в `tokens.css`, регенерируй `_junior.html`, снова покажи. Не двигайся в Full Pass пока пользователь не сказал «да, продолжай».

---

## Phase 6 — Full Pass HTML preview gallery

**Это новый функционал** не существующий в большинстве дизайн-скиллов. Цель — чтобы пользователь **увидел дизайн до того как разработчик начал кодить production**.

Скилл генерирует папку `docs/DESIGN-SYSTEM/preview/` с runnable HTML-страницами:

```
preview/
├── index.html                ← мастер-индекс с навигацией
├── colors.html               ← все brand + semantic + neutral токены визуально
├── typography.html           ← все t-* размеры + weights demonstrated
├── spacing.html              ← spacing scale visually
├── radii.html                ← radii scale
├── shadows.html              ← 3 уровня elevation
├── icons.html                ← icon-system showcase
├── components-cards.html     ← card-варианты (default, hover, dark accent)
├── components-buttons.html   ← все button-варианты (primary, secondary, ghost, danger)
├── components-inputs.html    ← inputs, textareas, selects
├── components-badges.html    ← pills, badges, status-индикаторы
├── components-nav.html       ← top nav, sidebar nav
├── motion.html               ← демо transitions (hover, focus, click)
├── mockups-dashboard.html    ← пример dashboard layout (asymmetric grid + bento)
├── mockups-form.html         ← пример формы
├── mockups-table.html        ← пример таблицы
└── tokens.css                ← скопированные токены (импортируется всеми preview-страницами)
```

Каждая страница — самодостаточная HTML, открывается двойным кликом. Используется тот же `tokens.css` что и в production — гарантирует pixel-perfect parity.

См. `templates/design-system-preview.html` для шаблона мастер-индекса.

### Пример: components-cards.html

```html
<!DOCTYPE html>
<html>
<head>
  <link rel="stylesheet" href="tokens.css">
  <style>
    body { font-family: var(--font-sans); padding: var(--s-12); background: var(--bg-soft); }
    .grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: var(--s-6); }
    .card { /* see Phase 4 */ }
  </style>
</head>
<body>
  <h1>Cards</h1>

  <h2>Default</h2>
  <div class="grid">
    <div class="card"><h3>Card title</h3><p>Body content goes here.</p></div>
    <div class="card card--hover"><h3>Hover state</h3><p>...</p></div>
    <div class="card card--accent"><h3>Brand accent</h3><p>...</p></div>
  </div>

  <h2>Dark accent variant</h2>
  <div class="grid">
    <div class="card card--dark">
      <h3 style="color: var(--bg)">Dark card</h3>
      <p style="color: rgba(255,255,255,0.8)">Body on dark background.</p>
    </div>
  </div>

  <!-- ... -->
</body>
</html>
```

После генерации — пользователь открывает `preview/index.html` в браузере. Видит **всю систему сразу**. Может ткнуть на компонент, скопировать CSS, прислать скриншот клиенту, попросить правки.

**Это критический момент в pipeline.** До этого момента дизайн жил в голове / в Figma / в текстовом каноне. Теперь — в живых пикселях которые можно поправить дёшево.

---

## Phase 6 — Brand voice document

Дизайн-канон — это не только цвета и шрифты. Это и **голос продукта**:

- Как пишутся заголовки (sentence case / title case)?
- Какие слова используем / избегаем?
- Punctuation rules (em-dash? guillemets?)
- Числа: разделители разрядов, decimal separator
- Tone: formal / conversational / playful
- Эмодзи в UI: запрещены / разрешены / контекстуально

Создай `docs/DESIGN-SYSTEM/BRAND-VOICE.md`. Шаблон в `references/brand-voice-template.md`.

Этот файл становится reference'ом для **любого** копирайта в продукте — UI-тексты, ошибки, маркетинг, документация.

---

## Phase 7 — Iteration

Покажи preview пользователю. Спроси:

- Что нравится сразу?
- Что вызывает «не уверен»?
- Что точно не подходит?

Корректируй в токенах (одна правка → все компоненты обновляются автоматически в preview). Это быстро — токен изменён, страница перезагружена, видно сразу.

Не торопись завершить. Лучше 5 итераций preview сейчас, чем переделывать все компоненты production-кода через 2 месяца.

**Stop условия:**
- Пользователь явно одобряет: «Да, это финальный вид»
- 3 последовательные итерации без существенных правок (только cosmetic)
- Бюджет времени исчерпан и текущий вариант acceptable

---

## Phase 8 — Hand-off

Финальный артефакт:

```
docs/DESIGN-SYSTEM/
├── README.md                ← философия, правила, anti-slop list
├── tokens.css               ← все CSS-переменные (production-ready)
├── BRAND-VOICE.md
├── preview/                 ← runnable HTML gallery
│   ├── index.html
│   └── ... (15-20 страниц)
└── assets/                  ← логотип, иконки, mockup-images
```

Скажи пользователю:

> Дизайн-система готова. Препрошу команду открыть `preview/index.html` и пройти по всем страницам — это финальная sanity-проверка. После одобрения — этот файл `tokens.css` копируется в production-проект как `src/app/globals.css` (или эквивалент). Все production компоненты ссылаются на токены, никаких hardcoded цветов в коде.
>
> Следующий шаг: `canon-roadmap` — декомпозиция канонов на фазы реализации.

---

## Anti-patterns (агенту)

❌ **Multi-color palette из коробки** — нет. Сначала один brand-цвет. Если действительно нужен второй (semantic accent) — добавляй позже как exception, не как rule.

❌ **«Давайте использовать тренды 2026»** — тренды устареют через год. Дизайн-система должна жить 5+ лет.

❌ **HTML preview — лишняя работа** — нет. Это самый дешёвый способ итерации. Без preview правки идут в production коде, дорого.

❌ **«Brand-voice сделаем потом»** — нет. Если voice не зафиксирован — UI-тексты будут расходиться по тону между разработчиками.

❌ **Bold weight для hierarchy** — нет, это нарушение принципа size-driven hierarchy. Используй размер.

❌ **Иконки как декорация** — нет. Каждая иконка должна нести информацию. Декоративные иконки удалять.

---

## Critical principles

1. **Один brand-цвет.** Не несколько.
2. **Размер выше веса.** h1 — 44px Regular, не 24px Bold.
3. **Hairline borders + tonal layering.** Не цветные карточки.
4. **Реальные изображения.** Не vector illustrations.
5. **HTML preview обязателен.** Не пропускай Phase 5.
6. **Токены везде.** Никаких hex в компонентах.
7. **Anti-AI-slop list актуален.** Сверяйся регулярно.
8. **Brand voice документирован.** Не только визуальное.

---

## Brand asset protocol (когда есть существующий бренд)

Если в Phase 0 верифицирован существующий продукт — **обязательно** соблюсти 5-step protocol перед Phase 2. Без этого design-system строится на догадках.

### Step 1 — Ask

Задай пользователю **явный** запрос на assets:

```
Для design-system нужны материалы реального бренда:
1. Логотип (SVG предпочтительно, или PNG transparent ≥ 2x)
2. Brand guidelines / style guide (PDF или ссылка)
3. Product screenshots (3-5 штук — главные экраны)
4. Базовая палитра если есть документация
5. Шрифт — название + где взять (Google Fonts / commercial / custom)

Что есть на руках?
```

### Step 2 — Search

Если пользователь не предоставил часть — поищи:
- Press kit / Brand guidelines / Media kit на сайте
- Open-source GitHub repo продукта (часто `assets/`, `branding/`)
- App store страница (для иконки и screenshots)

### Step 3 — Download / collect

Сложи всё в локальную структуру:

```
docs/DESIGN-SYSTEM/source-assets/
├── logo.svg (или logo.png)
├── brand-guidelines.pdf (если получили)
├── screenshots/
│   ├── home.png
│   ├── feature-1.png
│   └── ...
└── reference-products/   ← скриншоты конкурентов (positive + negative refs)
    └── ...
```

### Step 4 — Verify

Проверь что собранное **достаточно**. Чек-лист:
- [ ] Логотип в высоком разрешении (≥512px) или vector
- [ ] Минимум 1 product screenshot
- [ ] Известно какой шрифт (или явный выбор сделать)
- [ ] Известен base brand color (или явный выбор сделать)

Если что-то критическое отсутствует — **НЕ продолжай**. Вернись к пользователю с явным «без X нельзя двигаться, что делаем».

### Step 5 — Codify

Создай `docs/DESIGN-SYSTEM/brand-spec.md` — single source of truth:

```markdown
# Brand spec — {Product name}

## Verification
- Source: {где брали — official press kit / user-provided / approximated}
- Confidence: {high / medium / low}
- Date verified: {YYYY-MM-DD}

## Logo
- Files: source-assets/logo.{svg,png}
- Variants: {primary / mark-only / wordmark}
- Min size: {рекомендация по minimum display size}

## Colors
- Primary brand: {#hex} {name}
  - Source: {где найдено}
- Secondary (если есть): {#hex}
- Notes: {любые brand-rules про колор}

## Typography
- Display: {font name} {weight}
- Body: {font name} {weight}
- Source: {Google Fonts / commercial license / custom}

## Product imagery
- Available screenshots: {list}
- License: {usage rights}

## Tone / voice references
- Existing materials voice: {observation}
- Examples: {quotes from real product}
```

Этот файл — **источник истины** для всех Phase 2-9 решений. Если что-то противоречит `brand-spec.md` — правится дизайн, не spec.

---

## Tri-state fallback для assets

Когда нужен asset (логотип / screenshot / иконка / illustration) — действуй по уровням приоритета:

### Level 1 — Official source

Лучший вариант. Скачано из press kit / official repo. Не требует дополнительной валидации.

### Level 2 — User-provided

Пользователь прислал. Проверь качество (resolution, не сжатый). Зафиксируй source в `brand-spec.md`.

### Level 3 — AI-generated с reference

Если ни 1, ни 2 недоступно — можно AI-сгенерить **только** имея real reference. Например, иконку в стиле существующих иконок продукта.

**Категорически:** не AI-генерь логотипы. Не AI-генерь photoreal product images. Эти случаи — Level 4.

### Level 4 — Honest placeholder

Если real-asset невозможно получить — **явный placeholder** в HTML:

```html
<div class="image-placeholder">
  <span class="ph-label">Logo placeholder</span>
  <span class="ph-detail">Real logo to be inserted at /assets/logo.svg</span>
</div>
```

Не выдавай placeholder за реальное. Не stretch generic stock-photo «как замену». Честно: «здесь должно быть real X, пока его нет».

**Запрет:** silent fallback. Никогда не подменяй пустоту generic'ом без явного объявления.

---

## Embedded exception handler

Когда наблюдаешь одно из ниже — у тебя **готовый response**, не improvise:

| Ситуация | Response |
|---|---|
| Пользователь говорит «сделай как у Stripe» без brand-материалов | Step #0: «Это design-inspiration или экстракция Stripe? Если inspiration — какие конкретно паттерны нравятся? Если экстракция — нужны их official assets.» |
| Brand-color документации нет, нужно выбрать | Спросить: «Один цвет на основе чего? Логотипа? Industry-конвенции? Mood?». Дать 2-3 варианта с rationale. |
| Шрифт указан commercial (e.g. Söhne) но лицензии нет | Предложить free-substitute с похожим характером (Onest для Söhne). Зафиксировать в brand-spec как «substitution». |
| Пользователь хочет multi-color палитру | Объяснить single-hue principle. Если настаивает — документировать как deviation в brand-spec. Не молча подчиниться. |
| Запрос «добавьте playful animations» | Объяснить calm-tone defaults. Спросить: target-audience детский / casual? Если да — отдельная customization фаза. Если нет — придерживаться defaults. |
| Логотип некачественный (размытый PNG, low-res) | Step 4 verify: попросить улучшенный. Если недоступно — Honest placeholder в preview, флаг в brand-spec. |
| Product screenshots отсутствуют | Phase 0 stop: без screenshots невозможно делать realistic mockups. Спросить или попросить разрешения на placeholder в HTML. |
| Brand guidelines противоречат canon-dev anti-slop правилам (например, требуют bold) | Документировать конфликт. Спросить пользователя: brand-guidelines override (тогда добавить в brand-spec как exception) или canon-dev override (тогда не следовать guidelines). |

Эти ситуации — **предсказуемые**, не surprises. Подготовленный response быстрее ad-hoc reasoning'а.

---

## Templates и references

- `templates/colors_and_type.css` — стартовый шаблон CSS-токенов
- `templates/design-system-preview.html` — мастер-индекс preview gallery
- `templates/preview-colors.html` — preview-страница colors
- `templates/preview-typography.html` — preview-страница typography
- `templates/preview-components.html` — preview-страница components
- `references/anti-slop-rules.md` — расширенный anti-AI-slop guide
- `references/token-architecture.md` — глубокая теория почему токены так устроены
- `references/brand-voice-template.md` — шаблон BRAND-VOICE.md

## References router

Когда нужно — куда обращаться:

| Задача | Reference |
|---|---|
| Создать tokens.css с нуля | `templates/colors_and_type.css` |
| Понять почему токены устроены так | `references/token-architecture.md` |
| Что запрещено в дизайне | `references/anti-slop-rules.md` |
| Создать brand-voice doc | `references/brand-voice-template.md` |
| Сгенерировать HTML preview | `templates/preview-*.html` (3 шаблона страниц) |
| Master-index preview gallery | `templates/design-system-preview.html` |

---

## Финальная заметка

Дизайн-система — это **самый высокий рычаг** в продукте. Хорошая система делает каждый последующий компонент дешёвым (просто комбинация токенов). Плохая — делает каждый компонент болью (ad-hoc решения накапливаются как tech-debt).

Не сокращай Phase 5 (HTML preview). Это фаза которая отделяет **профессиональную дизайн-систему** от **набора цветов в Figma**.
