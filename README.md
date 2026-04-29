<p align="center">
  <img src="assets/cover.jpg" alt="canon-dev — pixel rocket carrying Claude through the canon pipeline" width="100%" />
</p>

<div align="center">

  # canon-dev

  **Canon-driven development pipeline for Claude Code, Codex, and compatible platforms.**

  [![English](https://img.shields.io/badge/lang-English-1E3A8A?style=for-the-badge)](README.md)
  [![Русский](https://img.shields.io/badge/lang-Русский-grey?style=for-the-badge)](README.ru.md)

  ![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)
  ![Status: alpha](https://img.shields.io/badge/status-alpha-orange.svg)
  ![Skills: 11](https://img.shields.io/badge/skills-11-1E3A8A.svg)
</div>

---

## What is this

`canon-dev` is a library of 11 skills that turn a raw product idea into production-ready code through a chain of versioned, auditable documents called **canons**. Instead of spec living in chat and getting lost, every architectural decision exists first as text — then as code, with verification along the way.

Useful for product engineers and architects who want documentation to lead implementation rather than chase it. Works in Claude Code (primary target), OpenAI Codex CLI, Cursor, Continue.dev, and any platform that loads skill manifests.

**Key benefit:** the next agent that touches your project — human or AI — does not have to reverse-engineer your intent. The canon already says what the system is and what it is not.

> Core documentation (philosophy, workflow, brand voice) is currently in Russian. English translations are welcome — see [`CONTRIBUTING.md`](CONTRIBUTING.md).

---

## The pyramid

```
                 ┌─────────────┐
                 │   CANONS    │  ← top-level principles and invariants
                 └──────┬──────┘
                        │
                ┌───────▼────────┐
                │     SPECS      │  ← per-module technical specifications
                └───────┬────────┘
                        │
                ┌───────▼────────┐
                │ ROADMAP / PLAN │  ← phases, dependencies, critical path
                └───────┬────────┘
                        │
                ┌───────▼────────┐
                │ PHASE PROMPTS  │  ← self-contained briefs for agents
                └───────┬────────┘
                        │
                ┌───────▼────────┐
                │     CODE       │  ← implementation
                └───────┬────────┘
                        │
                ┌───────▼────────┐
                │     AUDIT      │  ← verify code against canon
                └────────────────┘
```

Higher levels override lower ones. Canon beats spec. Spec beats plan. Plan beats prompt. Prompt beats code. Audit checks every level against the one above.

---

## Installation

One command, all platforms:

```bash
npx skills add prodbyEDDY/canon-dev
```

This uses the [`skills`](https://skills.sh) CLI from Vercel Labs. It clones the repo, asks which agents to target (Claude Code, Codex, Cursor, Continue, OpenCode, Cline, Amp, Gemini CLI — 50+ supported), and symlinks all 12 skills into the right directory.

### Common variants

```bash
# Install all skills to all detected agents, no prompts
npx skills add prodbyEDDY/canon-dev --all

# Install to a specific agent
npx skills add prodbyEDDY/canon-dev -a claude-code
npx skills add prodbyEDDY/canon-dev -a codex
npx skills add prodbyEDDY/canon-dev -a cursor

# Install globally (user home) instead of current project
npx skills add prodbyEDDY/canon-dev -g

# Install one specific skill only
npx skills add prodbyEDDY/canon-dev --skill braindump-to-canon

# Preview without installing
npx skills add prodbyEDDY/canon-dev --list

# Update to latest version later
npx skills update
```

### Manual install (no Node.js)

<details>
<summary>Expand for git-based install</summary>

```bash
# Claude Code (mac/Linux)
git clone https://github.com/prodbyEDDY/canon-dev.git ~/canon-dev
mkdir -p ~/.claude/skills
ln -s ~/canon-dev/skills/* ~/.claude/skills/

# Codex CLI
mkdir -p ~/.codex/skills
ln -s ~/canon-dev/skills/* ~/.codex/skills/
```

**Windows (PowerShell as Administrator):**

```powershell
git clone https://github.com/prodbyEDDY/canon-dev.git $HOME\canon-dev
New-Item -ItemType Directory -Force -Path $HOME\.claude\skills | Out-Null
Get-ChildItem $HOME\canon-dev\skills | ForEach-Object {
  New-Item -ItemType SymbolicLink -Path "$HOME\.claude\skills\$($_.Name)" -Target $_.FullName
}
```

Update with `cd ~/canon-dev && git pull` — symlinks pick up changes automatically.

</details>

---

## Quick start

After installation:

1. **Try `braindump-to-canon` first** if you have a product idea but no formal documentation yet. It runs a structured interview and produces canons, specs, and a stack decision.
2. **Try `canon-review` first** if you already have canons but suspect gaps. It runs an audit against a dimensional matrix.
3. **Read [`using-canon-dev`](skills/using-canon-dev/SKILL.md)** — top-level navigator that decides which skill to invoke for any situation.

---

## Skill catalog

### Main pipeline (6 stages)

| # | Skill | Purpose |
|---|-------|---------|
| 1 | **braindump-to-canon** | Turn raw product braindump into canons, module specs, and stack choice via structured interview. |
| 2 | **canon-review** | Audit existing canons for architectural gaps, missing edge cases, contradictions. |
| 3 | **design-canon** | Build a design system — tokens, typography, icons, motion — with HTML preview before any code. |
| 4 | **canon-roadmap** | Decompose canons into phased plan. Generates ROADMAP and a self-contained prompt for the next phase. |
| 5 | **canon-execute** | Implementation discipline for a single phase: pre-flight, branch hygiene, transactions, pre-merge gates. |
| 6 | **canon-verify** | Read-only auditor. Compares implementation against canon and spec, classifies findings by severity. |

### Cross-cutting discipline (5)

| Skill | When to apply |
|-------|---------------|
| **canon-tdd** | Inside `canon-execute` for testable logic. RED-GREEN-REFACTOR. Every bugfix starts with a failing test. |
| **canon-debug** | When something breaks, before any fix attempt. Four-phase investigation with defense-in-depth. |
| **canon-receive-review** | When PR review comments arrive. Three response modes: implement, push back, document tradeoff. |
| **canon-parallel** | When the coordinator considers 2+ independent tasks. Independence audit, partition, branch isolation. |
| **canon-worktree** | Utility — isolated git checkouts for parallel work or hotfix-during-feature scenarios. |

---

## Philosophy in seven lines

1. **Canon beats code.** When code conflicts with canon, the code changes — not the canon. Canon updates only via deliberate decision logged in the journal.
2. **Documentation is law, code is implementation.** Every architectural decision exists as text first, then as code. The reverse is an anti-pattern.
3. **No hardcoded values.** Colors are tokens. Sizes are tokens. Thresholds live in versioned registries. Formulas live in normative models.
4. **Self-contained prompts.** The implementing agent in a fresh session must be able to execute the prompt with zero outside context.
5. **Acknowledged debt over hidden debt.** When you cannot do it canonically right now, log it as debt with explicit close conditions — never silently skip.
6. **Antidubli as a first-class invariant.** Idempotency keys, partial unique indexes, participation flags — wherever an action could fire twice, the schema or service prevents the duplicate.
7. **Append-only audit logs, atomic transactions, honest unavailability.** Critical state is either consistent or it does not exist.

Full version with reasoning and exceptions: [`docs/PHILOSOPHY.md`](docs/PHILOSOPHY.md).

---

## Workflow example

A new product, end to end:

```
1. braindump-to-canon   →  docs/CANONS/, docs/SPECS/, docs/STACK.md
2. canon-review         →  findings applied, canons promoted to enterprise-ready
3. design-canon         →  docs/DESIGN-SYSTEM/ with HTML preview
4. canon-roadmap        →  ROADMAP.md + phase-1 prompt
5. canon-execute        →  fresh session implements phase 1, opens PR
6. canon-verify         →  fresh session audits the PR against canon
   ↺ repeat 4-6 for each subsequent phase
```

Three things make this different from "write some docs first":

- Each phase runs in a **fresh session** with a self-contained prompt — no context bleed between phases.
- The auditor (`canon-verify`) is **separate from the implementer** — no blind-spot bias.
- The `ROADMAP` is a **living document** with a decision journal and acknowledged-debt section — not an artifact created once and forgotten.

---

## Documentation

| Document | What it covers |
|----------|----------------|
| [`skills/using-canon-dev/SKILL.md`](skills/using-canon-dev/SKILL.md) | Top-level navigator — which sub-skill to invoke for your current situation |
| [`docs/PHILOSOPHY.md`](docs/PHILOSOPHY.md) | The ten principles, with reasoning and exceptions |
| [`docs/WORKFLOW.md`](docs/WORKFLOW.md) | Full pipeline, artifact diagrams, session model, anti-patterns |
| [`docs/BRAND-VOICE.md`](docs/BRAND-VOICE.md) | Voice guidelines for canon and prompt copy |

Each skill in `skills/` has its own `SKILL.md` plus optional `references/` and `templates/`.

---

## Compatibility

All 50+ agents supported by [skills.sh](https://skills.sh) work out of the box, including:

| Platform | Status |
|----------|--------|
| **Claude Code** | Primary target |
| **OpenAI Codex CLI** | Supported |
| **Cursor** | Supported |
| **Continue.dev** | Supported |
| **OpenCode, Cline, Amp, Gemini CLI, Roo, Kilo, Goose, Droid, Copilot CLI, …** | Supported via `npx skills add` |

The skills themselves contain no platform-specific code — they are markdown instructions plus reference material. Tool names referenced inside skills (Read, Write, Edit, Bash, Glob, Grep) are Claude Code conventions; on other platforms, your loader maps them to local equivalents.

---

## Inspiration and acknowledgements

- [`obra/superpowers`](https://github.com/obra/superpowers) — concept inspiration for the skill-pipeline pattern
- [`alchaincyf/huashu-design`](https://github.com/alchaincyf/huashu-design) — design discipline patterns that informed `design-canon`

Content here is original, not derived. Where a pattern was learned from another project, it is reformulated rather than copied.

---

## Contributing

Contributions are welcome. Bug reports and feature requests via GitHub Issues. New skills and improvements via pull request — see [`CONTRIBUTING.md`](CONTRIBUTING.md) for the conventions a skill must follow to fit the philosophy.

If you build something interesting on top of canon-dev, open an issue and link it. The library benefits from feedback by people running real projects on it.

---

## License

MIT — see [`LICENSE`](LICENSE). Use it, fork it, ship it.
