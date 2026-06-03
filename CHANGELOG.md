# Changelog

All notable changes to VibeCoder Essentials are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

No unreleased changes yet.

## [1.0.2] — 2026-06-03

### Fixed

- Restored a richer bilingual, trigger-focused Skill description.
- Confirmed `allowed-tools` is valid YAML frontmatter.
- Confirmed Markdown files use LF line endings.
- Kept `.gitattributes` configured for LF normalization.

## [1.0.1] — 2026-06-03

### Fixed

- Fixed `allowed-tools` YAML frontmatter in `SKILL.md`.
- Added `.gitattributes` to enforce LF line endings.
- Normalized Markdown formatting for README and examples.

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
- Example output files:
  - `examples/sample-review.md` — full illustrative Review Mode output.
  - `examples/sample-build-plan.md` — full illustrative Planning Mode output.
- **Claude Code Plugin Marketplace support** via `.claude-plugin/marketplace.json`.
  Claude Code users can install with
  `/plugin marketplace add axelandreyrv-dotcom/vibecoder-essentials` followed by
  `/plugin install vibecoder-essentials@vibecoder-essentials`, and invoke the
  skill with `/vibecoder-essentials:vibecoder-essentials`.
- **Codex native Agent Skill** install paths:
  `.agents/skills/vibecoder-essentials/SKILL.md` (repo-level) and
  `~/.agents/skills/vibecoder-essentials/SKILL.md` (user-level), with `AGENTS.md`
  as optional project-level guidance.
- `.github/copilot-instructions.md` example for GitHub Copilot / VS Code users.
- Install instructions for Claude.ai, Claude Code, Codex, and GitHub Copilot /
  VS Code.

[Unreleased]: https://github.com/axelandreyrv-dotcom/vibecoder-essentials/compare/v1.0.2...HEAD
[1.0.2]: https://github.com/axelandreyrv-dotcom/vibecoder-essentials/compare/v1.0.1...v1.0.2
[1.0.1]: https://github.com/axelandreyrv-dotcom/vibecoder-essentials/compare/v1.0.0...v1.0.1
[1.0.0]: https://github.com/axelandreyrv-dotcom/vibecoder-essentials/releases/tag/v1.0.0
