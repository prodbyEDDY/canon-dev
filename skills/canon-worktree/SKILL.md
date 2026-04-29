---
name: canon-worktree
description: Use when starting feature work that needs physical isolation from current workspace, or before dispatching parallel agents. Creates isolated git worktrees so agent works on independent checkout — no risk of conflicting with concurrent work, no need to stash, branch switching cheap. Includes safety checks (clean state verification) and cleanup discipline.
---

# canon-worktree

🚀 Утилитарный скилл — git worktrees для физической изоляции работы

---

## Когда использовать

- Начинаешь фазу реализации, текущая checkout грязная (uncommitted changes от другой работы)
- Coordinator dispatches parallel agents — каждому нужен изолированный checkout
- Нужно «попробовать» что-то рискованное без disturbing main checkout
- Bug investigation — хочешь reproduce на старом commit без losing current work

## Когда НЕ использовать

- Маленькая правка в текущей ветке — просто branch + commit
- Solo work, всегда working on main checkout — overhead не нужен
- Storage-constrained (worktrees займут ~size repo каждое)

---

## Что такое git worktree

`git worktree` — feature git'а который позволяет иметь **несколько checkout'ов одного repo одновременно**, в разных директориях, на разных branch'ах.

В отличие от clone — все worktrees делятся одной `.git`-историей. Дёшево storage-wise (только working files дублируются, не history).

В отличие от branch switching — не нужно stash'ить uncommitted changes. Branch остаётся в своём worktree.

---

## Setup

### Initial worktree

```bash
# Repo в C:\projects\my-app
# Hauptcheckout — на main
cd C:\projects\my-app
git status  # clean

# Создать worktree для новой ветки:
git worktree add ../my-app-feat-foo -b feat/foo

# Теперь:
# C:\projects\my-app          ← main
# C:\projects\my-app-feat-foo ← feat/foo branch, fresh checkout

# Перейти в worktree:
cd ../my-app-feat-foo

# Работать как обычно
git status
git add .
git commit -m "..."
git push -u origin feat/foo
```

### Naming convention

Worktree directory имена — descriptive, паттерн `{repo}-{feature-slug}`:

```
C:\projects\
├── my-app/                    ← main
├── my-app-feat-taxes/         ← Phase 4D работа
├── my-app-feat-payroll/       ← Phase 4E работа
└── my-app-debug-issue-42/     ← bug investigation
```

### Branch naming

Standard: `feat/{module}-{slug}`. Или `bugfix/{issue}-{slug}`. Или `audit/{phase}-{date}`.

---

## Discipline

### Pre-create checks

Перед `git worktree add`:

```bash
# 1. Verify main checkout — clean
cd C:\projects\my-app
git status  # should be empty
git log --oneline -1  # знаешь last commit

# 2. Pull latest
git fetch --all --prune
git checkout main
git pull --ff-only

# 3. Только теперь create worktree
git worktree add ../my-app-feat-foo -b feat/foo
```

### One worktree = one task

Не пытайся делать several phase в одном worktree. Цель — изоляция. Каждый task — свой worktree.

### Don't pollute neighbour worktrees

Worktrees делят `.git` каталог. Команды затрагивающие сам repo (например, `git config`, `git remote`) — affect все worktrees. Будь careful.

Worktree-specific operations (commit, push, branch switch внутри worktree) — изолированы. Это ОК.

### Status awareness

```bash
git worktree list
# Покажет все active worktrees:
# C:/projects/my-app                  abc123 [main]
# C:/projects/my-app-feat-foo         def456 [feat/foo]
# C:/projects/my-app-feat-bar         ghi789 [feat/bar]
```

Регулярно проверяй чтобы не накопилось забытых.

---

## Cleanup

После merge'а ветки — удалить worktree.

```bash
# Из main checkout:
cd C:\projects\my-app

# Removed worktree directory (и automatic git worktree prune):
git worktree remove ../my-app-feat-foo

# Если worktree содержит uncommitted changes — будет error.
# Force-remove (терять changes):
git worktree remove --force ../my-app-feat-foo

# Cleanup orphan worktree references (если directory удалена manually):
git worktree prune
```

### Branch cleanup отдельно

`git worktree remove` НЕ удаляет branch. Если branch merged:

```bash
git branch -d feat/foo  # delete local
git push origin --delete feat/foo  # delete remote
```

---

## Common scenarios

### Scenario 1 — Parallel agents

Coordinator dispatches 3 agents:

```bash
# Setup:
git worktree add ../my-app-agent-1 -b feat/refactor-foo
git worktree add ../my-app-agent-2 -b feat/refactor-bar
git worktree add ../my-app-agent-3 -b feat/refactor-baz

# Agent 1's prompt: «работай в C:\projects\my-app-agent-1, не выходи за пределы»
# Agent 2's prompt: «работай в C:\projects\my-app-agent-2, не выходи за пределы»
# Agent 3's prompt: «работай в C:\projects\my-app-agent-3, не выходи за пределы»

# После merge всех PRs:
git worktree remove ../my-app-agent-1
git worktree remove ../my-app-agent-2
git worktree remove ../my-app-agent-3
git branch -d feat/refactor-foo feat/refactor-bar feat/refactor-baz
```

### Scenario 2 — Bug investigation на старом commit

```bash
# Воспроизвести bug на конкретном commit:
git worktree add ../my-app-bug-investigation abc123  # detached at commit abc123

cd ../my-app-bug-investigation
# Делаешь debug, никак не трогая main checkout
# Когда понял — return обратно

cd ../my-app
git worktree remove ../my-app-bug-investigation
```

### Scenario 3 — Hotfix во время large feature

```bash
# Я на feat/big-feature, не хочу stash'ить, нужен hotfix:
git worktree add ../my-app-hotfix -b hotfix/critical-bug origin/main

cd ../my-app-hotfix
# Implement hotfix
git push -u origin hotfix/critical-bug
# Merge через PR

cd ../my-app
git worktree remove ../my-app-hotfix

# Возвращаюсь к big feature, всё where left it.
```

---

## Anti-patterns

❌ **Forgotten worktrees** — accumulate. Регулярно `git worktree list`, cleanup unused.

❌ **Worktree без соответствующей branch** — confusing. Always `-b {name}` при создании.

❌ **Шарить worktree между agents** — defeats purpose isolation. Один agent = один worktree.

❌ **Modify .git config из worktree** — affects все worktrees. Делай из main checkout если применимо.

❌ **Force-remove с uncommitted changes** — терь данные. Сначала commit или push, потом remove.

---

## Critical principles

1. **One worktree = one task.**
2. **Pre-create checks** — main clean, latest, fetch.
3. **Self-contained briefs для agent'ов** в worktrees — не cross-reference.
4. **Cleanup after merge** — worktree + branch (local + remote).
5. **`git worktree list` регулярно** — track что активно.

---

## Финальная заметка

Worktrees — небольшой overhead в storage и organization, big win в mental clarity. Особенно для parallel work и hotfix workflow.

Если работаешь solo на одном feature за раз — overhead не нужен. Используй когда multitasking или dispatching agents.
