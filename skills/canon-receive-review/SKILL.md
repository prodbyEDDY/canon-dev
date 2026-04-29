---
name: canon-receive-review
description: Use when receiving code review feedback on a PR (from coordinator or auditor) and need to respond technically — implement suggestions, push back on questionable feedback, document acknowledged tradeoffs. Enforces: technical rigor over performative agreement, verify before implement, document why-not when declining, batch responses to reduce churn.
---

# canon-receive-review

🚀 Дисциплина обработки feedback на PR в canon-dev pipeline

> Этот скилл — для **agent-реализатора** который открыл PR и получил review-comments. Параллелен `canon-execute` (для собственно реализации) и `canon-verify` (когда сам делаешь review).

---

## Когда использовать

- Coordinator оставил comments в PR
- Auditor (canon-verify результат) идентифицировал findings в моей реализации
- Reviewer попросил изменения

## Когда НЕ использовать

- Сам делаешь review чужого PR → `canon-verify`
- Hotfix без review (rare, и обычно неправильно)

---

## Философия

Review feedback — **диалог**, не приказ. Технический dialog.

Реализатор has knowledge об implementation context. Reviewer has knowledge об system / canon / architecture context. Оба нужны — не «реализатор молча соглашается на всё» и не «реализатор защищает каждую строку».

### Three response modes

Каждое feedback comment попадает в один из трёх:

1. **Implement** — feedback правильный, делаю изменение
2. **Push back** — feedback ошибочный или basis на misunderstanding, объясняю и не меняю
3. **Document tradeoff** — feedback указывает на real issue, но нет capacity / scope в этом PR. Acknowledge как debt, defer.

Любой из трёх — допустим. **Performative agreement** (когда implementor agrees на всё чтобы не спорить) — anti-pattern. Produceslow-quality код.

---

## Process

### Phase 1 — Read all comments first

Не отвечай по одному. Прочитай все. Сгруппируй:

- Critical (security, correctness, money)
- Major (architecture, lifecycle, antidubli)
- Minor (style, comment improvements)
- Suggestions (could-have, не-обязательные)

### Phase 2 — Classify each

Для каждого comment — какой из 3 modes?

```
Comment 1: «Add idempotency key here» 
→ Implement (Critical, real risk)

Comment 2: «I'd rename this variable»
→ Push back if not actually unclear / Implement if actually improves readability

Comment 3: «This formula doesn't handle case X»
→ Verify first: does formula actually fail for case X?
   If yes → Implement
   If no → Push back with explanation

Comment 4: «Could be more performant if cached»
→ Document tradeoff if real but not in scope / Implement if quick

Comment 5: «Inconsistent with style in module Foo»
→ Verify Foo's style is canonical → Implement
   Verify Foo is itself off-style → Push back, both should align с canon
```

### Phase 3 — Verify before implementing

Reviewer feedback не automatically correct. Перед implement — verify:

```
Reviewer: «This loop is N+1, fix to single query»

Verify: 
- Is it actually N+1? (look at code path)
- Если да — implement fix
- Если нет (e.g., uses dataloader или batch already) — push back с объяснением

Performative implement: «ok, let me restructure» → spend 30 min → 
discover it's not actually N+1 → revert → wasted time + confused reviewer.
```

### Phase 4 — Push back когда необходимо

Если feedback ошибочен — не соглашайся. Push back **technically**:

```
> Reviewer: "This formula should use compound interest"

> Response: "Compound interest applies for periods > 1 year. Per canon §X.Y, 
> we're computing simple interest for sub-year periods (line 42). Compound
> would be incorrect for period.kind='quarter'. Happy to add a comment 
> explaining the choice."
```

Хороший push-back:
- Technical (canon reference, code reference, numerical)
- Без "you're wrong" / personal
- Предлагает clarification если нужна (документация, comment)

Плохой push-back:
- Defensive («я так делал в других проектах»)
- Authority claim («я тут эксперт»)
- Без technical обоснования

### Phase 5 — Document tradeoffs

Real issue но out-of-scope этого PR? Acknowledge как debt:

```
> Reviewer: "This service has potential race condition under heavy load"

> Response: "Agreed — for high-concurrency scenarios this could fail.
> Current approach handles up to ~100 concurrent ok per our load profile.
> Adding distributed lock would require Redis infrastructure not yet 
> in stack. Filing as Ф{N} in ROADMAP §10.4 with closing condition
> 'when we add Redis to infrastructure'. Coordinator will register."
```

Это honest deferral. Reviewer удовлетворён (issue acknowledged), implementor не делает out-of-scope work, debt registered.

### Phase 6 — Batch responses

Не отвечай по одному comment'у в real time. Batch:

1. Прочитать все
2. Classify все
3. Implement all "Implement" в один (или несколько связанных) commit'ов
4. Push back на "Push back" comments одним сообщением reviewer'у
5. Document tradeoffs в одном update в PR description

Это **reduces churn**: reviewer проходит один раз, не возвращается 10 раз. Implementor concentrate'ует mental energy на implementations.

---

## Templates

### Implementing comment

Просто закоммить и упомянуть:

```
fix: address review — add idempotency key to createPayment

Per @reviewer feedback. Used format `payment-{orgId}-{paymentId}-create`.
```

В PR-comment:
> Done in {commit-sha}. ✓

### Pushing back

```markdown
**Re: {comment}**

I'd push back on this. {Technical reason}, see {file:line} or canon §X.Y.

Specifically:
- {detail 1}
- {detail 2}

Happy to {add comment / refactor naming / document the choice} if it'd help clarity.
But the implementation is correct as-is.
```

### Documenting tradeoff

```markdown
**Re: {comment}**

Real issue, but I'd defer rather than fix in this PR:

- **Reason:** {why out-of-scope — capacity / scope / dependency}
- **Closing condition:** {when this should be addressed}
- **Suggested debt code:** Ф{N}

Coordinator: please add to ROADMAP §10.4 as Ф{N} with above context.
```

### Batch update в PR description

После всех responses:

```markdown
## Review responses (round 1)

**Implemented:**
- [x] Idempotency key on createPayment ({commit-sha})
- [x] Extract pickRate helper ({commit-sha})
- [x] Add canon §X.Y reference in comment ({commit-sha})

**Push back (decided not to change):**
- Compound vs simple interest formula — see comment thread, canon §Y.Z applies
- Naming convention foo vs bar — current matches existing module style

**Acknowledged tradeoffs (deferred):**
- Race condition at heavy load → Ф{N} in ROADMAP §10.4
- Performance optimization for >10k records → Ф{M} in ROADMAP §10.4

Ready for re-review.
```

---

## Anti-patterns

❌ **Performative agreement** — agree на всё чтобы не спорить. Quality drop.

❌ **Defensive по любому feedback** — push back на everything, even valid. Wastes reviewer time.

❌ **Implement without verify** — feedback может быть основан на misunderstanding.

❌ **Mass-implement followed by mass-revert** — verify ПЕРЕД implement.

❌ **Real-time per-comment responses** — flooding reviewer notifications, churn.

❌ **Personal language** — «you misunderstand», «you're wrong». Technical only: «based on canon §X», «code at line N», «verified that ...».

❌ **Skipping verification of own claims** — push back «code is correct» without showing why.

---

## Critical principles

1. **Read all comments first.** Group, classify, plan response.
2. **Three response modes:** Implement / Push back / Document tradeoff. Pick одно для каждого.
3. **Verify before implement.** Не каждый feedback correct.
4. **Push back technically.** Canon refs, code refs, numbers.
5. **Document tradeoffs as acknowledged debt.** Defer без shame.
6. **Batch responses.** Один пройти, не 10.
7. **No performative agreement.** Defending good code is OK.

---

## Финальная заметка

Хороший review-receive process — collaborative, не submissive. Reviewer и implementor вместе делают code лучше.

Если каждый review цикл — мехническое «yes sir, fixed» — code не растёт качественно. Если каждый — «no, my way is right» — стагнирует.

Дисциплина: технически evaluating каждое feedback, implementing real improvements, pushing back на ошибочные, documenting deferrals. Это и есть dialog.
