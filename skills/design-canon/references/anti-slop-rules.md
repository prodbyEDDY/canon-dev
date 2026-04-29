# Anti-AI-slop rules

> Расширенный guide по тому что **запрещено** в любой дизайн-системе создаваемой через `design-canon`. Используй как чек-лист при review preview.
>
> **Inspired by:** discipline of huashu-design and similar restraint-first methodologies.

---

## Что такое AI-slop

«AI-slop» — паттерн дизайна который видно как AI-сгенерированный или generic-template. Они узнаваемы по конкретным маркерам:

- Многоцветные градиенты
- Эмодзи как функциональные иконки
- Иллюстрации абстрактных людей в плоском стиле
- Bold weight для всего
- Microinteractions без orchestration
- Marketing-суперлативы в UI-копирайте
- «Карточка с цветной полоской слева»
- Inter-as-display для всего

Если хотя бы три маркера присутствуют — система выглядит как «AI-сгенерировано». Это убивает доверие пользователя в B2B-контексте.

---

## Запрещённый список (полный)

### Колор

❌ **Multi-stop gradients** на CTA-кнопках или больших поверхностях (purple→pink, blue→cyan, sunset-style).
- Допустимое исключение: один subtle radial-glow за hero-заголовком в brand-цвете на 10% opacity.

❌ **Цветная полоса по левому краю карточки** (paper-style cards с brand-цветной accent stripe).
- Это устаревший material-design pattern, выглядит как 2014.

❌ **Rainbow-палитра** для категорий / тегов (красный для X, зелёный для Y, синий для Z).
- Используй один brand-цвет с разной интенсивностью или semantic-tone (success=green, warning=orange, danger=red — но **только** для семантики, не для категоризации).

❌ **Pure black** (`#000000`) для текста.
- Используй deep brand-900 или near-black с brand-tint (`#0E2A66` style).

❌ **Pure white** (`#FFFFFF`) для всего фона.
- Используй tinted-white (`--bg-soft` `#F7FAFF` style) для page background, pure white только для floating cards.

### Иконки

❌ **Эмодзи в production UI** (✅ ❌ 🎉 в кнопках или статусах).
- Допустимое исключение: эмодзи в user-generated content (чат, комментарии).
- Запрещено: эмодзи в navigation, kpi-cards, button-labels, error-messages.

❌ **Mixed icon styles** (некоторые filled, некоторые outline, некоторые с разной stroke).
- Один style throughout. Lucide line-icons — defacto стандарт.

❌ **Heavy filled icons** (Material's filled style) для secondary actions.
- Outline-only для большинства cases.

❌ **3D-rendered icons** (за исключением hero-illustrations).
- В UI — только line / solid-fill flat.

### Типографика

❌ **Bold weight** для hierarchy.
- Используй размер. h1 — 44-72px Regular впечатляет сильнее чем 24px Bold.

❌ **Inter-everything** (inter для display + body + monospace).
- Если используешь Inter — используй его для body. Для display подбери что-то с характером (Onest, Manrope, Geist, или serif как Fraunces).

❌ **Italic** в UI.
- Italic — литературный приём, не UI. Если нужен emphasis — другой weight или color, не italic.

❌ **All-caps long text** (заголовки в uppercase больше 4 слов).
- Допустимо: 1-2 слова eyebrow-label uppercase tracking-wide. Запрещено: целые абзацы caps.

### Изображения

❌ **Vector illustrations абстрактных людей в flat-style** (типичные undraw.co иллюстрации).
- Используй реальные фотографии или продуктовые скриншоты.

❌ **CSS-shapes как stand-in для product mock-up** (рисованные «typing» bubbles, «browser» rectangles без реального контента).
- Используй реальные screenshots в browser-frame обёртке.

❌ **Stock-photography с обобщёнными бизнес-сценами** (handshake, smiling team meeting, looking at laptop).
- Если нужны фотографии — авторские или нишевые.

❌ **Hero-images с overlay-текстом без protection-gradient** при низкой контрастности.
- Если фото-фон — обязательно либо высокий контраст, либо protection-gradient, либо reposition текста на solid surface.

### Layout

❌ **Uniform 3/4-column grid** для всех секций лендинга.
- Используй asymmetric layouts: bento с разными weights, narrow text + wide media, etc.

❌ **Identical card sizes** в feature-секциях.
- Хотя бы один visual focal point — большая карточка / hero-card / feature-row.

❌ **Center-aligned all text** на длинных страницах.
- Center для hero. Left для body. Mixed alignment рассеивает читабельность.

❌ **Full-bleed photos везде где есть пустое место**.
- Whitespace ≠ unused space. Whitespace — instrument.

### Анимации

❌ **Bouncy / spring physics** на UI-компонентах (карточки которые «пружинят» при hover).
- Используй ease-out cubic-bezier(0.22, 0.61, 0.36, 1).

❌ **Overshoot transitions** (элемент переходит дальше цели и возвращается).
- Linear monotonic motion. Без overshoot.

❌ **Scroll-jacking** (захват scroll'а для slide-by-slide page).
- Native scroll. Всегда.

❌ **Random hover-microinteractions** на каждом элементе (rotation, скейл, glow).
- Один coherent motion-language: card hover = border + shadow change. Button hover = bg change. Не больше.

❌ **Auto-playing carousels** в hero.
- User-controlled или вообще без carousel.

### Копирайт

❌ **Восклицательные знаки** в UI и маркетинге.
- Calm tone без `!`.

❌ **Суперлативы** ("революционный", "лучший", "уникальный", "№1", "единственный").
- Декларация фактов: "Делает X через Y".

❌ **Marketing-pop фразы** ("просто как 1-2-3", "по щелчку пальцев", "killer feature").

❌ **Anglicisms-as-verbs** (на русском: "кликни", "залогинься", "забукай").
- Используй нейтрально: "нажмите", "войдите", "забронируйте".

❌ **Эмодзи в маркетинговом тексте** (рандомные 🚀 🎯 ✨).

---

## Sanity-test для preview

После генерации HTML-preview — пройди по списку:

```
Colors
- [ ] Один brand-цвет основной?
- [ ] Нет multi-stop gradients?
- [ ] Нет pure black text / pure white background?

Icons
- [ ] Все иконки одной системы (Lucide/Phosphor/Tabler)?
- [ ] Стек stroke-weight одинаковый?
- [ ] Нет эмодзи в UI?

Typography
- [ ] Bold weight отсутствует?
- [ ] Используется размер для hierarchy?
- [ ] Italic отсутствует в UI?

Layout
- [ ] Asymmetric layouts присутствуют?
- [ ] Whitespace используется намеренно?

Motion
- [ ] Все transitions ease-out, ≤220ms?
- [ ] Нет bouncy/spring effects?

Copy
- [ ] Без `!` в UI?
- [ ] Без супер-лативов?
- [ ] Без эмодзи в текстах?
```

Если хотя бы одно «нет» — fix перед окончательным hand-off.

---

## Когда правила нарушаются

**Иногда** — оправданно. Например:
- Игровой продукт может использовать яркую палитру (нарушение «один brand-цвет»)
- Детский продукт может использовать playful animations (нарушение «no bouncy»)
- Развлекательный контент-продукт может использовать эмодзи в UI

В таких случаях — нарушение должно быть **обдуманным и продуктово обоснованным**, документированным в дизайн-каноне с reasoning. «Случайно нарушили потому что выглядело красиво» — нет, всегда переделать.

---

## Финальная мысль

Этот anti-slop list — не про эстетический фетишизм. Это про **доверие**. B2B-пользователь смотрит на UI и подсознательно решает «можно ли доверить этому продукту реальные деньги / данные / процессы». AI-slop сигнализирует «это template / generic / shallow». Restrained design сигнализирует «это серьёзно / продуманно / деталями занимались».

Нарушение каждого пункта стоит долю доверия. Соблюдение всех — даёт чёткое впечатление профессионализма даже у новичка-пользователя который не сможет объяснить почему.
