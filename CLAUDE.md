# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This repository is a collection of **skills and agents for Claude Code**, focused on
**coding workflows**. Each skill packages a reusable, opinionated procedure (e.g. planning,
review, debugging, shipping) that Claude Code can invoke on demand.

The design and structure are informed by [gstack](https://github.com/garrytan/gstack) — a
mature, production-grade collection of Claude Code coding skills. We study gstack's conventions
(skill anatomy, progressive disclosure, decision flowcharts, safety guardrails) as a
reference while building our own set.

## Core design requirement: two independent language settings

Every skill in this repository MUST support **two independent, configurable languages**:

1. **Output language** — the language a skill uses for its *results*: the text returned to
   whoever invoked the skill (summaries, reports, explanations, commit messages shown to the
   user, etc.).
2. **Thinking language** — the language a skill uses for its *internal reasoning*: planning,
   analysis, intermediate notes, and chain-of-thought.

These two settings are **fully independent**. For example, a skill may *think* in English
(for maximal model capability and consistency with code/identifiers) while *returning results*
in Russian to the caller — or any other combination.

Guidelines:

- Default both settings to **English** when unset.
- Resolve the setting at skill invocation time; do not hardcode language into skill logic.
- The output language affects only user-facing text. It MUST NOT translate code, identifiers,
  file paths, commands, or API names.
- When a skill spawns subagents, it must propagate both language settings to them.

**Resolution mechanism** (every skill follows this order, first match wins):

1. Per-invocation flags `--output-lang <lang>` and `--thinking-lang <lang>`.
2. Repo config file `.claude/skill-config.json` with keys `outputLanguage` and
   `thinkingLanguage`.
3. Default: **English**.

## Repository layout

- `skills/<name>/SKILL.md` — one directory per skill; `SKILL.md` holds the skill definition
  (YAML frontmatter `name` + `description`, then the procedure). This `skills/` directory is
  the canonical source for the collection.
- `skills/<name>/references/` — bundled templates, checklists, and rubrics loaded on demand.
  Keep `SKILL.md` thin (procedure + a copyable checklist); move long templates here
  (progressive disclosure).

## Skill authoring conventions

Grounded in Anthropic's Agent Skills best practices and patterns observed across mature
collections (gstack, BMAD-METHOD, Spec Kit, Pimzino spec-workflow):

- **Description = discoverability.** Write the `description` in the third person and state both
  WHAT the skill does and WHEN to use it. Claude selects skills from this field. Do NOT rely on
  literal "trigger phrases".
- **Progressive disclosure.** Thin orchestrating body (~1,500–2,000 words) + a copyable
  sequential checklist; long templates/rubrics live in `references/`.
- **Persona + anti-sycophancy.** Validation and review skills adopt an explicit critical
  persona, take a position, and name failure patterns instead of hedging.

## Development-process skill pipeline (PRD phase)

The "raw idea → initial project documentation" flow is a fixed, phased pipeline. Each phase is
its own skill, adopts a persona, and writes one persistent artifact that the next phase reads.

| # | Skill | Persona | Artifact |
|---|-------|---------|----------|
| 1 | `idea-validation` | Founder-turned-investor | `docs/idea-validation.md` |
| 2 | `prd` | Product manager | `docs/prd.md` |
| 3 | `ux-journey` | Product designer | `docs/ux-journey.md` |
| 4 | `architecture` | Software architect (requirements-first) | `docs/architecture.md` + `docs/adr/*` |
| 5 | `review-doc` | Critic (rubric gate, reusable) | annotates the target doc |

Conventions for these skills:

- **Artifacts live in the repo under `docs/`.** Phase N reads phase N−1's artifact from there.
- **Hard gate between phases.** A phase skill must finish its artifact and get explicit user
  approval before the next phase starts. Never jump ahead to a later phase's concern.
- **Idea validation is adversarial.** A cheap KILL / SKIP / SHRINK pre-filter, then forcing
  questions (demand, audience specificity, problem validation, status-quo competitor, wedge,
  business model). Its only output is the validation doc — no solutioning.
- **Architecture is requirements-first.** Elicit quality-attribute scenarios (cost, performance,
  security, reliability, scale) first; then propose 2–3 component options with trade-offs.
  Separate WHAT from HOW — do NOT commit to specific frameworks here. Record significant
  decisions as ADRs (options + ruled-out alternatives + trade-offs + status).
- A router/orchestrator over these skills is deferred until the manual flow stabilizes.

## Authoring conventions

- **Write all skill content (instructions, prompts, descriptions) in English**, regardless of
  the configured output/thinking languages. The languages above are runtime behavior, not the
  language the skills themselves are written in.
- **Write this CLAUDE.md and all repository documentation in English.**
- **Always write git commit messages in English.**

## Git workflow

- **Never create a separate feature branch unless explicitly asked to.** Work on the current
  branch by default; only branch when the user explicitly requests it.
