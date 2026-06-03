# Changelog

All notable changes to VibeCoder Essentials are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Added

- **Claude Code Plugin Marketplace support** via `.claude-plugin/marketplace.json`.
  Claude Code users can now install with
  `/plugin marketplace add axelandreyrv-dotcom/vibecoder-essentials` followed by
  `/plugin install vibecoder-essentials@vibecoder-essentials`, and invoke the
  skill with `/vibecoder-essentials:vibecoder-essentials`.
- Codex **native Agent Skill** install paths documented
  (`.agents/skills/vibecoder-essentials/SKILL.md` repo-level and
  `~/.agents/skills/vibecoder-essentials/SKILL.md` user-level), with `AGENTS.md`
  kept as optional project guidance.

### Changed

- Reworked the `README.md` installation flow to put the Claude Code plugin
  marketplace **Quick start** first, clarified that **Claude.ai** custom skills
  use **ZIP upload**, and moved manual install lower as a fallback.

## [1.0.0] — 2026-06-02

### Added

- Initial release of the **VibeCoder Essentials** Agent Skill.
- **Planning Mode** — pre-build question bank and a `Build Plan` deliverable with
  a Build Readiness Verdict.
- **Review Mode** — post-build audit checklist and a `Review` deliverable with a
  Final Verdict.
- **Whole-project mode** — survey-then-drill workflow for auditing entire repos
  or deployments, with optional subagent fan-out.
- Bilingual (English + Spanish), trigger-focused frontmatter `description` for
  reliable activation in both languages.
- Reference files:
  - `references/planning-mode.md` — Planning Mode question bank.
  - `references/review-mode.md` — Review Mode audit checklist.
  - `references/security-baseline.md` — engineering/security baseline mapped to
    OWASP Top 10, OWASP API Security Top 10, and CWE.
  - `references/output-templates.md` — exact Build Plan and Review templates.
- Repository documentation: `README.md`, `SECURITY.md`, `CONTRIBUTING.md`,
  `AGENTS.md`, and `examples/sample-review.md`.
- Install instructions for Claude.ai, Claude Code, Codex, and GitHub Copilot /
  VS Code.

[Unreleased]: https://github.com/your-org/vibecoder-essentials/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/your-org/vibecoder-essentials/releases/tag/v1.0.0
