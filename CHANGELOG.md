# Changelog

All notable changes to canon-dev are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [Unreleased]

## [0.1.0] - 2026-04-29

### Added

- Initial release of the canon-dev skill library.
- **Main pipeline (6 skills):**
  - `braindump-to-canon` — structured interview turning raw product ideas into canons, module specs, and stack decision.
  - `canon-review` — enterprise-grade audit of existing canons for gaps and contradictions.
  - `design-canon` — design-system construction with HTML preview before code, fact-verification step, and brand-spec protocol.
  - `canon-roadmap` — decomposition of canons into phased plan with self-contained per-phase prompts.
  - `canon-execute` — implementation discipline for a single phase: pre-flight, branch hygiene, transactions, pre-merge gates.
  - `canon-verify` — read-only auditor that compares implementation against canon and classifies findings by severity.
- **Cross-cutting discipline (5 skills):**
  - `canon-tdd` — RED-GREEN-REFACTOR discipline for testable logic.
  - `canon-debug` — four-phase systematic debugging with defense-in-depth.
  - `canon-receive-review` — handling PR review comments via three response modes.
  - `canon-parallel` — independence audit and partition for parallel agent dispatch.
  - `canon-worktree` — git worktree utility for isolated parallel checkouts.
- **Top-level documentation:**
  - `SKILL.md` — top-level skill router.
  - `docs/PHILOSOPHY.md` — ten principles with reasoning.
  - `docs/WORKFLOW.md` — full pipeline, artifact diagrams, session model.
  - `docs/BRAND-VOICE.md` — voice guidelines for canon and prompt copy.
- **Public-facing files:**
  - `README.md` — English landing page with installation for Claude Code, Codex, Cursor, Continue.dev.
  - `README.ru.md` — Russian mirror.
  - `LICENSE` — MIT.
  - `CONTRIBUTING.md` — skill conventions and PR process.
- `assets/logo.svg` — rocket-on-foundation logo.

[Unreleased]: https://github.com/prodbyEDDY/canon-dev/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/prodbyEDDY/canon-dev/releases/tag/v0.1.0
