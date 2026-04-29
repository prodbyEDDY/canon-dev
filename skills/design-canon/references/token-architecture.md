# Token architecture — теория и практика

> Глубокий guide по тому **почему** дизайн-токены устроены именно так и **как** их грамотно структурировать.

---

## Зачем токены

Без токенов:
- `color: #2C53C9;` встречается в 47 местах
- Изменить brand-цвет = touch 47 файлов
- Случайно получаются 4 «почти-синих» оттенка вместо одного

С токенами:
- `color: var(--brand-600);` встречается в 47 местах, токен определён в одном месте
- Изменить brand-цвет = одна строка
- Гарантированно один оттенок везде

Это **operational benefit**. Но есть и **архитектурный**:

С токенами **дизайн-система — это код**, не дизайн-документ. Дизайн-канон становится исполняемым: правки токена пропагируются автоматически через все компоненты. Дизайн-долг видно (если компонент использует hardcoded цвет — это нарушение, можно grep'ом найти).

---

## Иерархия токенов (3 уровня)

### Level 1 — Primitive tokens

«Сырой» уровень: цветовая шкала, размерная шкала, без семантики.

```css
--brand-25:  #F7FAFF;
--brand-50:  #F2F6FD;
--brand-100: #E8EFFB;
...
--brand-900: #0E2A66;

--gray-50:  #F9FAFB;
...
--gray-900: #111827;
```

Эти токены **не используются в компонентах напрямую** (правило, не запрет). Они — палитра из которой строятся semantic tokens.

### Level 2 — Semantic tokens

Семантический уровень: связь primitive с назначением.

```css
--bg:           #FFFFFF;       /* page bg */
--bg-soft:      var(--brand-25);  /* subtle alt-bg */
--bg-panel:     var(--brand-50);  /* panel bg */

--fg-1:         var(--brand-900); /* primary text */
--fg-2:         #4A5878;          /* body */
--fg-3:         #7A8AA6;          /* caption */

--line:         #E5ECF5;
--line-soft:    #EFF3F9;

--success:      #2BA672;
--success-bg:   #E7F5EE;
```

В компонентах используются **только** semantic tokens. Это даёт два уровня свободы:
- Можно изменить primitive (рефакторинг палитры)
- Можно перепрашить semantic (например, --fg-2 = более тёмный neutral) — все компоненты обновятся

### Level 3 — Component tokens

(Опционально) уровень компонент-специфичных.

```css
--btn-primary-bg:       var(--brand-700);
--btn-primary-fg:       var(--bg);
--btn-primary-hover-bg: var(--brand-800);

--card-bg:              var(--bg);
--card-border:          var(--line);
--card-shadow:          var(--shadow-1);
```

Используется когда нужна гранулярная настройка одного компонента без затрагивания других. Для маленьких систем — необязательно. Для больших (с темной темой, theming, white-label) — обязательно.

---

## Категории токенов

### Color

**11-step shade scale** для brand:

```
brand-25  ← almost-white tint
brand-50
brand-100 ← soft fill (icon plates, hover bg)
brand-200 ← card hover border
brand-300 ← divider on dark surface
brand-400
brand-500 ← bright accent (focus ring, chart line)
brand-600 ← link, accent number
brand-700 ← primary CTA
brand-800 ← hover/active CTA
brand-900 ← deepest, headings
```

**Math для генерации:**
- HSL: hold hue, vary lightness from 96% (50) to 16% (900)
- OKLCH: hold L0 + C0, vary L from 0.95 to 0.18
- Manual tweak для perceptual evenness (некоторые шаги могут быть «слишком близко»)

**Test**: каждые два соседних шага должны быть **различимы при normal vision** на одинаковых backgrounds. Если -300 и -400 неотличимы — переделать шкалу.

### Neutrals

Tinted в сторону brand:

```
bg / bg-soft / bg-panel — три уровня page-фона
line / line-soft — два уровня hairline-borders
fg-1..fg-4 — четыре уровня текста
```

**Tinted neutrals (not pure gray):** добавь 5-10% saturation в направлении brand. Чистые серые выглядят холодно и mechanical. Tinted gray — warm и cohesive.

### Semantic

Пять semantic-токенов с background-вариантами:

```
success / success-bg
warning / warning-bg
danger / danger-bg
info / info-bg (== brand-500 / brand-100)
```

**Правило:** semantic используется **только для семантики**, не для категоризации. Зелёный — для positive deltas в финансовом UI. Красный — для errors. Не для «зелёный таб лучше синего».

### Typography

```
font-sans / font-mono — families
weight-light/regular/medium — 300/400/500 (никогда 600+)
t-display..t-meta — 11 размеров
leading-tight..loose — 4 line-heights
tracking-tight..wide — 3 tracking
t-num-xl/lg — special для numbers (tabular-nums)
```

**Why size scale 12-72:**
- 12 (meta) — мелкий вспомогательный
- 13 (caption) — captions
- 14-15 (body-sm/body) — основной текст
- 17 (body-lg) — лид
- 18-20 (h5/h4) — мелкие заголовки
- 24 (h3) — section sub-headers
- 32 (h2) — section headers
- 44 (h1) — page-title
- 72 (display) — hero only

**Why только 3 веса:**
Bold (600+) даёт визуальный crashing, особенно на small text. Weight-driven hierarchy выглядит как «document», не «UI». Size + space — лучше.

### Spacing

4-base scale:
```
1=4 / 2=8 / 3=12 / 4=16 / 5=20 / 6=24 / 8=32 / 10=40 / 12=48 / 16=64 / 20=80
```

**Почему 4-base, не 8-base:**
- 4-base даёт больше granularity для micro-spacing (между иконкой и текстом — 4 или 8, не «между 8 и 16»)
- 8-base достаточен для general layout, но micro моменты страдают

**Не используй pixel values вне scale.** `padding: 13px` — нарушение. `padding: var(--s-3)` (12px) — правильно.

### Radii

```
sm=6 / md=10 / lg=16 / xl=20 / 2xl=28 / pill=999
```

**Когда какой:**
- sm — мелкие inputs, badges
- md — buttons, simple cards
- lg — default cards (16px — современный baseline)
- xl/2xl — hero cards, feature panels
- pill — tags, status pills, round buttons

**Consistency:** в одной view не смешивай xl и md без причины. Один радиус-уровень для main-card, другой для secondary.

### Shadows

3 elevation levels:
```
shadow-1 — at-rest cards (1px hairline)
shadow-2 — hover / floating
shadow-3 — modals / focus overlays
```

**Brand-tinted shadows (not black):**
```css
--shadow-1: 0 1px 2px rgba(brand-rgb, 0.04),
            0 0 0 1px rgba(brand-rgb, 0.04);
```

Чёрные тени выглядят суровыми и cheap. Brand-tinted (alpha 4-10%) — soft, premium.

### Motion

```
ease-out:    cubic-bezier(0.22, 0.61, 0.36, 1)  ← default
ease-in-out: cubic-bezier(0.65, 0, 0.35, 1)
dur-fast: 140ms / dur-base: 220ms / dur-slow: 420ms
```

**Why:**
- ease-out — calm acceleration, fits content-driven products
- 140-220ms — fast enough to feel responsive, slow enough to be perceived
- Duration > 420ms — too slow, feels janky

**Anti-curves:**
- linear — feels mechanical
- ease-in — feels delayed
- spring / overshoot — feels playful, not professional

---

## Dark mode

(Если поддерживается)

**Pattern:** `.dark` selector переопределяет semantic-токены, primitive — те же.

```css
:root {
  --bg: #FFFFFF;
  --fg-1: var(--brand-900);
}

.dark {
  --bg: #0F1419;
  --fg-1: #E8EEF7;
}
```

**Semantic token names не меняются** — компоненты не должны знать о теме. Они просто используют `var(--bg)`, который переключится.

**Что нужно изменить в dark:**
- Bg / fg colors (свапнуть в обратку)
- Shadow alpha (повысить так как тени не видны на тёмном)
- Brand-цвет может потребовать tint adjustment (light brand-цвет на dark может «жечь»)
- Borders (--line) — обычно светлее на dark

**Что НЕ нужно изменять:**
- Spacing
- Typography sizes / weights
- Radii
- Component structure

---

## Theming / White-label (advanced)

Если продукт нужно ребрендировать клиентам:

```
:root[data-theme="default"] {
  --brand-500: #3B6EE8;
  /* ... все brand shades */
}

:root[data-theme="acme-corp"] {
  --brand-500: #B91C1C; /* их корпоративный red */
  --brand-700: #991B1B;
  /* ... */
}
```

JS переключает `data-theme` атрибут — мгновенно вся UI меняет brand. Семантические компоненты не меняются.

---

## Token naming conventions

### Хорошо

- Семантика: `--bg`, `--fg-1`, `--line`, `--success`
- Шкалы: `--brand-700`, `--s-6`, `--t-h2`
- Производные: `--btn-primary-bg`

### Плохо

- Hex literals: `--blue-color: #3B6EE8` (где «blue»? lavender? navy?)
- Component-tied без семантики: `--my-special-card-bg`
- Дублирующие: `--brand-blue-500` + `--brand-500` (один из них устаревший?)

### Правило

Имя токена должно отвечать на вопрос **«где это используется?»** или **«где это в шкале?»**, не **«какого это цвета?»**.

`--brand-700` — где в brand-шкале (7-й шаг). `--success` — semantic. Оба good.
`--blue` — отвечает «какого цвета». Bad.

---

## Anti-patterns

❌ **Hardcoded цвета в компонентах** — `background: #2C53C9` в `.button`. Использовать токен.

❌ **Создание токена для одного места** — `--landing-hero-special-blue: #...`. Если используется только в одном месте — переделать на existing-токен или (если ничего не подходит) добавить в semantic-уровень.

❌ **Слишком много semantic-токенов** — `--my-page-section-1-bg`, `--my-page-section-2-bg`, ... Признак того что страница плохо структурирована, не что нужно больше токенов.

❌ **Token shadowing** — два токена с одинаковым значением и разными именами. Confused. Объединить.

❌ **Изменение primitive ради одного компонента** — «давайте brand-700 поярче чтобы кнопка была заметнее». Нет — это меняет brand для всего. Создать local override на уровне компонента или semantic-токен `--btn-primary-bg`.

---

## Migration: hardcode → tokens

Если у вас существующая кодовая база с hardcoded цветами:

1. **Audit:** `grep -rn "#[0-9a-fA-F]\{6\}" src/` → список всех hex
2. **Cluster:** группируй похожие цвета (4 «почти-синих» → один token)
3. **Map:** для каждого hex найди или создай подходящий token
4. **Replace:** один файл за раз, тестируй визуально после каждого
5. **Lint rule:** добавь stylelint правило `no-hex-colors` чтобы предотвратить регресс

Это длительная работа но окупается. Lint после миграции = guarantee что hardcode не вернётся.

---

## Финальная мысль

Хорошая token-архитектура — почти невидима. Никто не пишет статьи «как красиво устроены токены в Stripe». Они просто **работают**: дизайнер меняет brand → разработчик меняет один файл → весь UI обновляется → никто не пишет ad-hoc цвета потому что lint ругается.

Это и есть цель. Не toolset для гордости — невидимая infrastructure для высокого темпа итераций.
