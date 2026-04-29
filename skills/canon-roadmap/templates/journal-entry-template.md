# Journal entry template

> Шаблон записи в ROADMAP §11 «Журнал решений». Используется координатором при обновлении после каждой фазы.

---

## Type 1 — Phase merge entry

```markdown
### {YYYY-MM-DD} — Phase {N}.{X} замёржен (PR #{num})

**Контекст:** {1-2 предложения про что делалось — отсылка к промпту}

**Что замёржено:**
- Schema: {list of new tables/enums/changes}
- Logic: {list of new modules в src/lib/}
- API: {list of new endpoints}
- UI: {list of new pages/components}
- Total: {X} files changed, {+Y/-Z} lines

**Канонические тесты пройдены:**
- §{section} тест-кейсы — {N}/{N} ✓
- Учебные примеры расчётов §{section} — корректны

**Расхождения с планом:**
- {Spec §X.Y} обещал {expected}, реализовано как {actual}. Причина: {reason}. Это:
  - 🟢 acceptable (improvement)
  - 🟡 acknowledged debt (см. §10.4 Ф{code})
  - 🔴 needs follow-up (см. open question Q-{N})

**Open questions resolved:**
- Q-{N}: {description} → {решение}

**Acknowledged debt added:**
- Ф{code}: {описание + when to close}

**Verification:**
- Local: `pnpm exec tsc --noEmit`, `pnpm lint`, `pnpm build` ✓
- Brand grep: clean ✓
- T-сценарии: {N}/{N} pass
- Coordinator post-merge на staging: {когда / pending}

**Git cleanup после merge:**
- Удалена ветка `feat/{name}` локально и remote
- main fast-forwarded
```

---

## Type 2 — Major decision entry

```markdown
### {YYYY-MM-DD} — {Decision title}

**Контекст:** {что подтолкнуло к решению — баг? новое требование? technical exploration?}

**Решение:** {принятое решение в одно предложение}

**Альтернативы рассмотрены:**
- A: {option A} — trade-off
- B: {option B} — trade-off
- C: {chosen} — почему лучше

**Что меняется:**
- В коде: {что нужно изменить}
- В канонах: {какие каноны обновятся}
- В roadmap: {как влияет на фазы}

**Дата effective:** {with which phase / immediately}

**Кто решал:** {user / coordinator / consult с экспертом}
```

---

## Type 3 — Hotfix entry

```markdown
### {YYYY-MM-DD} — Hotfix Ф{code} замёржен (PR #{num})

**Производственный инцидент:** {описание симптома в проде}

**Root cause:** {что именно сломано}

**Fix:** {что изменено}

**Affected:** {какие пользователи / org / периоды затронуты}

**Verification:**
- Reproduce: {как воспроизводилось до fix'а}
- Confirmation: {как подтверждено что fix работает}

**Backfill для production:** {делается / не делается / частично}

**Post-mortem:** {1-2 предложения про что упустили в pre-merge → tighten как процесс}
```

---

## Type 4 — Architectural deviation entry

```markdown
### {YYYY-MM-DD} — Architectural deviation в Phase {N}

**Plan §{X.Y} говорил:** {что было запланировано}

**Реализовано как:** {фактический подход}

**Почему отклонились:**
- {Constraint 1}: например, schema требует NOT NULL для field X, а план предполагал nullable
- {Constraint 2}

**Что сохраняется из канонической логики:**
- {Aspect 1}: например, ОПиУ-classification сохранена через operation_date

**Что отличается:**
- {Aspect 1}: например, accrual создаётся в pay-time вместо approve-time

**Trade-offs:**
- ✅ {pro}: {почему это OK}
- ⚠️ {con}: {что компенсируем}

**Условия пересмотра:**
{When/if this deviation should be revisited — например, «when schema-PR introduces nullable financeAccountId, restore approve-time accrual»}

**Acknowledged debt code:** Ф{N}
```

---

## Заметки по стилю

- Краткость: одна запись = ≤ 500 слов. Если длиннее — обычно стоит вынести в отдельный документ (post-mortem).
- Конкретность: имена файлов, PR-номера, даты. Не «недавно сделали» — «PR #42 merged 2026-04-29».
- Honest: если что-то отклонено от плана — фиксировать, не маскировать. Это и есть ценность журнала.
- Cross-references: всегда ссылка на ROADMAP §10/§10.4 коды Ф{N} и open questions Q-{N}.
- Reverse chronology: новейшие записи **сверху**.

## Когда писать

- После **каждого** merge'а phase-PR (Type 1)
- После принятия любого крупного архитектурного решения (Type 2)
- После каждого hotfix'а (Type 3)
- При обнаружении archaeologically-важного отклонения (Type 4)

## Когда НЕ писать

- Минорные коммиты (typo fixes, dependency updates)
- Routine refactoring без архитектурных последствий
- Внутренние discussion'ы (это в чате, не в журнале)

## Критическое правило

Журнал — **append-only**. Не удалять и не редактировать прошлые записи. Если решение оказалось неверным — добавить новую запись с reversal, ссылка на старую. История остаётся.
