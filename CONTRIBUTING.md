# Contributing to canon-dev

Thanks for your interest. This library grows when people running real projects feed back what worked and what did not.

---

## How to contribute

| Kind of contribution | Where it goes |
|----------------------|---------------|
| Bug report (typo, broken link, skill misbehavior) | [GitHub Issues](https://github.com/prodbyEDDY/canon-dev/issues) |
| Feature request or new skill idea | [GitHub Issues](https://github.com/prodbyEDDY/canon-dev/issues) — with rationale for fit with the philosophy |
| New skill | Pull request — see "Skill conventions" below |
| Improvement to existing skill | Pull request — small, focused, with a clear changelog line |
| English translation of `docs/` | Pull request — keeps tone from `docs/BRAND-VOICE.md` |

Use Issues for discussion before opening a PR for anything non-trivial. Saves rework on both sides.

---

## Skill conventions

A new skill must follow the structure established by existing ones. Mismatches make the library inconsistent and harder to use.

### Required structure

```
skills/your-skill-name/
├── SKILL.md             ← required
├── references/          ← optional, for deep-dive material
│   └── *.md
└── templates/           ← optional, for boilerplate the skill produces
    └── *.md
```

### `SKILL.md` frontmatter

YAML frontmatter with two fields, both required:

```yaml
---
name: your-skill-name
description: One sentence stating when to invoke this skill — used by the agent's skill router. Be specific about triggers.
---
```

The `description` is the most important field. It should let an agent decide in one read whether the skill applies. Include trigger phrases ("Use when…"), not abstract definitions.

### `SKILL.md` body

Body conventions, derived from existing skills:

- **Decision flow first.** A short section that helps the agent decide whether to proceed with this skill or hand off to another.
- **Numbered phases.** Most canon-dev skills are sequential procedures. Number them.
- **Concrete examples.** Show one good output, one bad output. Not abstractions.
- **References last.** Link to `references/` files for deep material that does not fit inline.

### Brand voice

All skills, references, and templates follow [`docs/BRAND-VOICE.md`](docs/BRAND-VOICE.md). Calm, structural, declarative. No exclamations, no emoji in body text (graphic markers in headers are allowed). No marketing language.

### Philosophy fit

A new skill must clearly serve one or more of the ten principles in [`docs/PHILOSOPHY.md`](docs/PHILOSOPHY.md). The PR description should name which principle(s) the skill supports and why the existing skills do not already cover that ground.

Skills that overlap heavily with existing ones will be asked to merge instead of stand alone.

### Originality

Do not paste skill content verbatim from other libraries. Where you learn a pattern from another project (`obra/superpowers`, `alchaincyf/huashu-design`, etc.), reformulate it in canon-dev voice and credit the source in the PR description.

---

## Pull request process

1. **Branch naming:** `feat/<slug>` for new skills or features, `fix/<slug>` for bugfixes, `docs/<slug>` for documentation-only changes.
2. **Description:** clear summary of what changed and why. For new skills, include the philosophy-fit rationale.
3. **Changelog:** add a line under `## [Unreleased]` in `CHANGELOG.md`.
4. **Self-review:** before requesting review, read your own diff. Catch typos, broken links, voice drift.
5. **Reviews:** maintainers will respond within a few days. Reviews may push back on scope or voice — that is part of keeping the library coherent.

---

## Code of conduct

This project follows the spirit of the [Contributor Covenant](https://www.contributor-covenant.org/). In short: be respectful, assume good faith, focus on the work. Maintainers reserve the right to remove contributions or contributors that consistently violate this.

---

## Questions

Open an issue. Do not email maintainers directly — keeping discussion in the open lets future contributors learn from the same answer.
