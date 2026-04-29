<p align="center">
  <img src="assets/cover.jpg" alt="canon-dev — пиксельная ракета с Claude летит сквозь canon pipeline" width="100%" />
</p>

<div align="center">

  # canon-dev

  **Canon-driven development pipeline для Claude Code, Codex и совместимых платформ.**

  [![English](https://img.shields.io/badge/lang-English-grey?style=for-the-badge)](README.md)
  [![Русский](https://img.shields.io/badge/lang-Русский-1E3A8A?style=for-the-badge)](README.ru.md)

  ![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)
  ![Status: alpha](https://img.shields.io/badge/status-alpha-orange.svg)
  ![Skills: 11](https://img.shields.io/badge/skills-11-1E3A8A.svg)
</div>

---

## Что это

`canon-dev` — библиотека из 11 скиллов, которая превращает сырую продуктовую идею в production-ready код через цепочку версионируемых, проверяемых документов — **канонов**. Вместо спецификации, которая живёт в чате и теряется, каждое архитектурное решение существует сначала как текст, потом как код, и проверяется на каждом шаге.

Полезно продуктовым инженерам и архитекторам, которым нужно чтобы документация вела реализацию, а не догоняла её. Работает в Claude Code (основная цель), OpenAI Codex CLI, Cursor, Continue.dev и любой платформе, поддерживающей skill-манифесты.

**Ключевая выгода:** следующему агенту, который коснётся проекта — человек или AI — не нужно реверс-инжинирить твой замысел. Канон уже говорит, что система делает и чего она не делает.

---

## Пирамида

```
                 ┌─────────────┐
                 │   КАНОНЫ    │  ← верхнеуровневые принципы и инварианты
                 └──────┬──────┘
                        │
                ┌───────▼────────┐
                │       ТЗ       │  ← модульные технические спецификации
                └───────┬────────┘
                        │
                ┌───────▼─────────┐
                │   ROADMAP / PLAN │  ← фазы, зависимости, критический путь
                └───────┬──────────┘
                        │
                ┌───────▼─────────┐
                │  PHASE PROMPTS   │  ← самодостаточные брифы для агентов
                └───────┬──────────┘
                        │
                ┌───────▼─────────┐
                │      КОД         │  ← реализация
                └───────┬──────────┘
                        │
                ┌───────▼─────────┐
                │     АУДИТ        │  ← сверка реализации с каноном
                └─────────────────┘
```

При конфликте побеждает верхний уровень. Канон выше ТЗ. ТЗ выше плана. План выше промпта. Промпт выше кода. Аудит сверяет каждый уровень с тем, что выше.

---

## Установка

Одна команда, любая платформа:

```bash
npx skills add prodbyEDDY/canon-dev
```

Использует [`skills`](https://skills.sh) CLI от Vercel Labs. Клонирует репо, спрашивает в какие агенты установить (Claude Code, Codex, Cursor, Continue, OpenCode, Cline, Amp, Gemini CLI — поддерживается 50+), и симлинкует все 12 скиллов в нужную директорию.

### Часто используемые варианты

```bash
# Все скиллы во все обнаруженные агенты, без промптов
npx skills add prodbyEDDY/canon-dev --all

# В конкретный агент
npx skills add prodbyEDDY/canon-dev -a claude-code
npx skills add prodbyEDDY/canon-dev -a codex
npx skills add prodbyEDDY/canon-dev -a cursor

# Глобально (в домашнюю директорию) вместо текущего проекта
npx skills add prodbyEDDY/canon-dev -g

# Только один конкретный скилл
npx skills add prodbyEDDY/canon-dev --skill braindump-to-canon

# Превью без установки
npx skills add prodbyEDDY/canon-dev --list

# Обновление до последней версии позже
npx skills update
```

### Ручная установка (без Node.js)

<details>
<summary>Развернуть для git-based install</summary>

```bash
# Claude Code (mac/Linux)
git clone https://github.com/prodbyEDDY/canon-dev.git ~/canon-dev
mkdir -p ~/.claude/skills
ln -s ~/canon-dev/skills/* ~/.claude/skills/

# Codex CLI
mkdir -p ~/.codex/skills
ln -s ~/canon-dev/skills/* ~/.codex/skills/
```

**Windows (PowerShell от Администратора):**

```powershell
git clone https://github.com/prodbyEDDY/canon-dev.git $HOME\canon-dev
New-Item -ItemType Directory -Force -Path $HOME\.claude\skills | Out-Null
Get-ChildItem $HOME\canon-dev\skills | ForEach-Object {
  New-Item -ItemType SymbolicLink -Path "$HOME\.claude\skills\$($_.Name)" -Target $_.FullName
}
```

Обновление через `cd ~/canon-dev && git pull` — симлинки подхватят изменения автоматически.

</details>

---

## Быстрый старт

После установки:

1. **Начни с `braindump-to-canon`**, если есть продуктовая идея, но нет формальной документации. Скилл проводит структурированное интервью и выдаёт каноны, ТЗ и stack decision.
2. **Начни с `canon-review`**, если каноны уже есть, но подозреваешь дыры. Скилл проходит по матрице измерений.
3. **Прочитай [`using-canon-dev`](skills/using-canon-dev/SKILL.md)** — top-level навигатор, который решает какой sub-скилл вызвать в любой ситуации.

---

## Каталог скиллов

### Main pipeline (6 стадий)

| # | Скилл | Назначение |
|---|---|---|
| 1 | **braindump-to-canon** | Превращает свободный braindump о продукте в каноны, ТЗ модулей, выбор стека через структурированное интервью. |
| 2 | **canon-review** | Аудит существующих канонов на архитектурные дыры, упущенные edge cases, противоречия. |
| 3 | **design-canon** | Создание дизайн-системы — токены, типографика, иконки, motion — с HTML-предпросмотром до кодинга. |
| 4 | **canon-roadmap** | Декомпозиция канонов на план по фазам. Генерирует ROADMAP и самодостаточный промпт следующей фазы. |
| 5 | **canon-execute** | Дисциплина реализации одной фазы: pre-flight, branch hygiene, transactions, pre-merge gates. |
| 6 | **canon-verify** | Read-only auditor. Сверяет реализацию с каноном и ТЗ, классифицирует находки по severity. |

### Cross-cutting discipline (5)

| Скилл | Когда применять |
|---|---|
| **canon-tdd** | Внутри `canon-execute` для testable логики. RED-GREEN-REFACTOR. Каждый bugfix начинается с failing test. |
| **canon-debug** | Когда что-то сломалось — до любого fix-attempt'а. 4-фазное расследование с defense-in-depth. |
| **canon-receive-review** | Когда пришли comments на PR. Three response modes: Implement / Push back / Document tradeoff. |
| **canon-parallel** | Когда coordinator рассматривает 2+ independent tasks. Independence audit, partition, branch isolation. |
| **canon-worktree** | Утилита — изолированные git checkouts для parallel work или hotfix-во-время-feature. |

---

## Философия в семи строках

1. **Канон выше кода.** Когда код противоречит канону — правится код, не канон. Канон обновляется только через осознанное решение с записью в журнал.
2. **Документация — закон, код — реализация.** Любое архитектурное решение существует сначала как текст, потом как код. Reverse — анти-паттерн.
3. **Никаких хардкодов.** Цвета — токены. Размеры — токены. Пороги — версионируемые реестры. Формулы — нормативные модели.
4. **Самодостаточные промпты.** Агент-реализатор в чистой сессии должен выполнить промпт без внешнего контекста.
5. **Acknowledged debt вместо скрытого долга.** Когда не можешь сделать каноничный — записываешь как debt с явными условиями закрытия, а не молча пропускаешь.
6. **Antidubli как первоклассный инвариант.** Idempotency keys, partial unique indexes, флаги участия — там где действие может быть вызвано дважды, схема или сервис предотвращают дубль.
7. **Append-only audit logs, atomic transactions, honest unavailable.** Критическое состояние — либо консистентно, либо не существует.

Полная версия с обоснованиями и исключениями: [`docs/PHILOSOPHY.md`](docs/PHILOSOPHY.md).

---

## Workflow — пример

Новый продукт с нуля:

```
1. braindump-to-canon   →  docs/КАНОНЫ/, docs/ТЗ/, docs/STACK.md
2. canon-review         →  findings применены, каноны помечены enterprise-ready
3. design-canon         →  docs/DESIGN-SYSTEM/ с HTML-предпросмотром
4. canon-roadmap        →  ROADMAP.md + промпт фазы 1
5. canon-execute        →  свежая сессия реализует фазу 1, открывает PR
6. canon-verify         →  свежая сессия проводит audit PR против канона
   ↺ повтор 4-6 для каждой следующей фазы
```

Три вещи отличают это от «давайте сначала напишем какие-то доки»:

- Каждая фаза идёт в **свежей сессии** с самодостаточным промптом — нет context bleed между фазами.
- Auditor (`canon-verify`) **отделён от implementer'а** — нет blind-spot bias.
- `ROADMAP` — **живой документ** с журналом решений и секцией acknowledged-debt, а не артефакт «создали один раз и забыли».

---

## Документация

| Документ | Что покрывает |
|---|---|
| [`skills/using-canon-dev/SKILL.md`](skills/using-canon-dev/SKILL.md) | Top-level навигатор — какой sub-скилл вызвать для текущей ситуации |
| [`docs/PHILOSOPHY.md`](docs/PHILOSOPHY.md) | Десять принципов с обоснованиями и исключениями |
| [`docs/WORKFLOW.md`](docs/WORKFLOW.md) | Полный pipeline, диаграммы артефактов, сессионная модель, анти-паттерны |
| [`docs/BRAND-VOICE.md`](docs/BRAND-VOICE.md) | Голос документации — правила копирайта для канонов и промптов |

У каждого скилла в `skills/` свой `SKILL.md` плюс опциональные `references/` и `templates/`.

---

## Совместимость

Все 50+ агентов из [skills.sh](https://skills.sh) поддерживаются из коробки, включая:

| Платформа | Статус |
|---|---|
| **Claude Code** | Основная цель |
| **OpenAI Codex CLI** | Поддерживается |
| **Cursor** | Поддерживается |
| **Continue.dev** | Поддерживается |
| **OpenCode, Cline, Amp, Gemini CLI, Roo, Kilo, Goose, Droid, Copilot CLI, …** | Поддерживаются через `npx skills add` |

Сами скиллы не содержат platform-specific кода — это markdown-инструкции плюс reference-материалы. Имена инструментов внутри скиллов (Read, Write, Edit, Bash, Glob, Grep) — конвенция Claude Code; на других платформах твой loader мапит их на локальные эквиваленты.

---

## Вдохновение и благодарности

- [`obra/superpowers`](https://github.com/obra/superpowers) — концептуальное вдохновение для skill-pipeline паттерна
- [`alchaincyf/huashu-design`](https://github.com/alchaincyf/huashu-design) — паттерны design-дисциплины, легли в основу `design-canon`

Контент здесь оригинальный, не производный. Где паттерн взят из другого проекта — он переформулирован, а не скопирован.

---

## Contributing

Контрибьюции приветствуются. Баги и feature requests — через GitHub Issues. Новые скиллы и улучшения — через pull request. См. [`CONTRIBUTING.md`](CONTRIBUTING.md) для конвенций, которым должен следовать новый скилл чтобы вписаться в философию.

Если построил что-то интересное на canon-dev — открой issue со ссылкой. Библиотека выигрывает от обратной связи людей, использующих её на реальных проектах.

---

## Лицензия

MIT — см. [`LICENSE`](LICENSE). Используй, форкай, делись.
