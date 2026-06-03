# Contributing to VibeCoder Essentials

Thanks for helping make vibe-coded software safer and saner. This project is a
documentation-only Agent Skill, so contributing is mostly about clear writing,
accurate engineering advice, and good judgment.

## Ways to contribute

- **Improve the advice** — sharper questions in Planning Mode, better checks in
  Review Mode, clearer rationale.
- **Fix inaccuracies** — outdated practices, wrong OWASP/CWE mappings, broken
  reasoning.
- **Add coverage** — a class of bug, an architecture pattern, or a platform
  concern that vibe-coders routinely miss.
- **Improve compatibility** — better install instructions for Claude.ai, Claude
  Code, Codex, or Copilot/VS Code.
- **Add examples** — realistic, anonymized sample Build Plans or Reviews under
  `examples/`.

## Principles

1. **Never trust the frontend.** Every piece of advice should be consistent with
   the skill's rule number one: validation and authorization live on the server.
2. **Explain the *why*.** We favor reasoning over rigid `MUST`/`NEVER` walls.
   Tell the model (and the reader) why something matters so it generalizes.
3. **Keep `SKILL.md` lean.** It should stay focused and ideally under ~500 lines.
   Push exhaustive checklists and long material into `references/`.
4. **Stay safe and honest.** No exploit code, no malware, no real secrets, no
   misleading instructions. Defensive, illustrative snippets only. Don't make the
   skill hallucinate findings — preserve the calibration guidance.
5. **Cite standards.** When you add a security check, map it to OWASP Top 10,
   OWASP API Security Top 10, or a CWE where one applies.
6. **Don't invent tools.** Recommend reputable, widely-used tools; don't fabricate
   obscure ones.

## Style

- Imperative voice in instructions ("Validate input server-side"), not passive.
- Concrete over abstract. A finding format of *what / where / why / fix* beats a
  vague warning.
- Bilingual triggering: the skill's frontmatter `description` includes English
  and Spanish so it activates in both. If you touch the description, keep both.

## Making a change

1. Fork and branch: `git checkout -b improve-review-checklist`.
2. Edit the relevant file(s). Keep the frontmatter valid (`name`, `description`,
   `version`, `license`, and a Claude-Code-compatible `allowed-tools`).
3. If you change behavior, update `CHANGELOG.md` under an *Unreleased* heading.
4. Proofread for clarity and accuracy. Read it back with fresh eyes.
5. Open a PR describing **what** you changed and **why**, and how you verified the
   advice is correct.

## Versioning

We follow [Semantic Versioning](https://semver.org/):

- **Patch** — typos, clarifications, small accuracy fixes.
- **Minor** — new checks, new sections, expanded coverage (backward-compatible).
- **Major** — changes to the output templates or mode behavior that downstream
  users depend on.

Bump `version` in `SKILL.md` frontmatter and note it in `CHANGELOG.md`.

## Code of conduct

Be direct about technical risks, kind to people. Critique the work, not the
contributor. That's the same posture the skill itself takes.
