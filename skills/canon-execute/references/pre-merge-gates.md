# Pre-merge gates — детальный checklist

> Gates которые **обязательно** проходятся перед открытием PR. Если хоть один не пройден — fix перед PR.

---

## Gate 1 — TypeScript compilation

```bash
pnpm exec tsc --noEmit
```

**Expected:** 0 errors, 0 warnings.

**Если упал:**
- Read every error. Не игнорируй.
- Не используй `// @ts-ignore` или `// @ts-expect-error` без обоснования. Эти comments означают «я знаю что есть проблема, но решил не fixить» — фиксят почти всегда.
- `any` исключение — сначала пытайся с `unknown` + narrowing. `any` — last resort с inline-комментарием почему.

**Common fixes:**
- Missing imports → добавь
- Type mismatch на API response → правильно типизируй с zod schema (одно schema = source-of-truth для API response и frontend type)
- Untyped JSON parsing → используй Zod's `parse` или `safeParse`

---

## Gate 2 — ESLint

```bash
pnpm lint
```

**Expected:** 0 errors, 0 warnings.

**Если упал:**
- Unused imports — remove
- Unused variables — remove или префиксуй `_` если по контракту нужны (например, `_req` в API handler где request не используется)
- React-hooks-rules violations — это обычно реальный bug, исследуй
- `console.log` warnings — replace на `console.warn` если действительно нужен log в prod, иначе — remove

**ESLint config tweaks:**
Если правило кажется чрезмерным для нашего проекта — обсуди с координатором, не отключай молча. Lint-rules — это team-level decision.

---

## Gate 3 — Database schema sync

```bash
pnpm db:generate
```

**Expected:** одно из:
- `No schema changes detected` — если schema файлы не трогал
- Migration файл создан в `src/db/migrations/` — если schema трогал

**Critical:**
- Если schema трогал — **обязательно** закоммить и schema, и migration вместе в одном commit'е
- Migration файл нельзя редактировать вручную (особенно ALTER TYPE ADD VALUE statements которые non-transactional)
- `pnpm db:push` — только для local dev, никогда для production

**Common issues:**
- Schema-file имеет drift от applied migrations → `pnpm db:generate` создаст extra migration. Просмотри SQL — это то что ожидал?
- Forgot to import new schema in `src/db/schema/index.ts` → migration не сгенерируется. Re-run.

---

## Gate 4 — Production build

```bash
pnpm build
```

**Expected:** success, no errors.

**Если упал:**
- Server-only code в client component → переместить или wrap'нуть
- Missing env vars — добавь в `.env.example` если новые нужны
- Bundle size warnings — обычно safe ignore, но если bundle > 500KB на route — investigate

---

## Gate 5 — Brand compliance grep

(Из промпта phase, sectional N+1)

```bash
# No bold weights
grep -rn "font-bold\|font-semibold\|fontWeight: *[6789]\|font-[678]00" \
  src/{paths-changed}/ 2>/dev/null

# Expected: 0 matches
```

```bash
# No off-brand colors
grep -rn "purple\|violet\|teal\|emerald\|orange\|amber\|fuchsia\|cyan" \
  src/{paths-changed}/ 2>/dev/null

# Expected: 0 matches
# Exception: legitimate use в lucide-react icon imports (e.g. `Cyan`-icon) — но обычно нет
```

```bash
# No hardcoded colors
grep -rn "text-black\|bg-black\|text-gray-[0-9]\|bg-gray-[0-9]" \
  src/{paths-changed}/ 2>/dev/null

# Expected: 0 matches (use --fg-* / --bg-* tokens)
```

```bash
# No emojis в production code (hard to grep, но manual scan)
# Особенно проверь в UI-component files
```

```bash
# No dev-time console.log left over
grep -rn "console\.log" src/{paths-changed}/ 2>/dev/null

# Expected: 0 matches (или только intentional с comment'ом)
```

---

## Gate 6 — T-scenarios manual run

(Из промпта phase, секция T-сценарии)

Каждый T-сценарий выполни локально:

```
T-01. {Сценарий описание}
- Open: http://localhost:3000/{path}
- Action: {что делаешь}
- Expected: {ожидание}
- Actual: ✅ matches / ⚠️ partial: {detail} / ❌ fail: {detail}
```

**Note about auth-gated UI:**

Если local dev не имеет working auth (например, BetterAuth настроен на production callback URL), некоторые T-сценарии не verifiable локально. В этом случае:
- Помечай как `⚠️ verifiable on staging only`
- В PR description явно укажи которые
- Координатор проверит на staging

Не пропускай T-сценарии которые **могут** быть verified локально.

---

## Gate 7 — Git state

```bash
git status --short
```

**Expected:** только intended changes. Не должно быть:
- Random untracked files (особенно `.env.local`, `*.log`, `node_modules` — это `.gitignore`-bug)
- Случайно committed скриншоты / dump-файлы
- IDE-specific files (`.idea/`, `.vscode/`)

```bash
git log --oneline -10
```

**Expected:** последние коммиты — твои, по теме фазы. Если есть случайные коммиты от других веток — что-то с branch state, разбираться.

```bash
git diff main..HEAD --stat
```

**Expected:** только файлы которые промпт обещал затронуть. Если есть unrelated файлы — out-of-scope нарушение.

---

## Gate 8 — Documentation alignment

- [ ] Если schema изменилась — закомичен migration файл?
- [ ] Если новый module — есть README в module-folder (для нетривиальных)?
- [ ] Если architectural deviation — есть inline-comment с TODO + reference на canon?
- [ ] Если новые tokens — обновлены или нет (обычно не должны меняться в обычной phase)?

---

## Gate 9 — PR title and description ready

- [ ] Title по convention: `feat({module}): {short} (Phase {N})`
- [ ] Description filled per phase prompt §10 template
- [ ] Acknowledged deviations явно перечислены
- [ ] Pre-merge gates прошедшие — отмечены
- [ ] T-сценарии results — честно отмечены

---

## Gate-failure log

Если хоть один gate упал — **stop**. Не открывай PR. Fix → re-run all gates → only then PR.

Это золотое правило. PR с failing gates стоят больше чем delay'и от gate-fix'ов.

---

## Quick reference

```bash
# Полный pre-merge sequence:

pnpm exec tsc --noEmit       # Gate 1
pnpm lint                     # Gate 2
pnpm db:generate              # Gate 3
pnpm build                    # Gate 4

# Gate 5 — adapt grep paths to changed dirs
grep -rn "font-bold\|font-semibold" src/{paths}/ 2>/dev/null
grep -rn "purple\|violet\|teal" src/{paths}/ 2>/dev/null
# ...

# Gate 6 — manual run в browser

# Gate 7 — final git check
git status --short
git diff main..HEAD --stat | head -20

# Gate 8-9 — manual checklist
```

Когда все gates ✅ — открывай PR.
