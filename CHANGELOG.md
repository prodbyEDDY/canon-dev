# Changelog

All notable changes to canon-dev are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [Unreleased]

## [0.1.1] - 2026-04-29

### Changed
- One-line install via `npx skills add prodbyEDDY/canon-dev` ([skills.sh](https://skills.sh) ecosystem) replaces the manual git+symlink instructions. Manual install kept as a fallback in collapsed `<details>`.
- Top-level `SKILL.md` (the navigator) moved to `skills/using-canon-dev/SKILL.md` so the `skills` CLI discovers all 12 skills (was finding only the root one). Total skill count is now 12 (1 navigator + 6 main + 5 cross-cutting).
- Pixel-art cover image added to README hero (`assets/cover.jpg`); the small SVG logo is no longer rendered in the README but remains in `assets/logo.svg` for other uses.

### Fixed
- `canon-tdd` and `canon-receive-review` SKILL.md descriptions used unquoted YAML colon-space sequences (`discipline: write`, `Enforces: technical`) that caused YAML parsers to mis-detect them and skip them during `npx skills add` enumeration. Replaced with em-dash and removed the colon respectively.

### Removed
- Empty `references/` directories under `canon-parallel/` and `canon-receive-review/` (these skills had no reference files to put there yet).

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

[Unreleased]: https://github.com/prodbyEDDY/canon-dev/compare/v0.1.1...HEAD
[0.1.1]: https://github.com/prodbyEDDY/canon-dev/compare/v0.1.0...v0.1.1
[0.1.0]: https://github.com/prodbyEDDY/canon-dev/releases/tag/v0.1.0
