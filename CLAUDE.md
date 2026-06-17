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

This repo is **both a Claude Code plugin marketplace and the plugin it ships** — so the skills
can be installed into any project from GitHub.

```
.claude-plugin/marketplace.json          # marketplace catalog (lists the plugin)
plugins/builder-skills/
  .claude-plugin/plugin.json             # the plugin manifest
  skills/<name>/SKILL.md                 # one directory per skill
  skills/<name>/references/*.md          # bundled templates/rubrics (progressive disclosure)
CLAUDE.md
README.md
```

- Skills live under `plugins/builder-skills/skills/`. Component dirs (`skills/`, and later
  `commands/`, `agents/`) sit at the **plugin root**, not inside `.claude-plugin/`.
- `SKILL.md` = YAML frontmatter (`name` + `description`) + a thin procedure with a copyable
  checklist. Long templates go in `references/` and load on demand.
- Marketplace plugin `source` must be a relative path starting with `./` (here
  `./plugins/builder-skills`); a repo-root/`"."` source is NOT supported.
- Validate any change with `claude plugin validate .`.

### Installing the skills into another project

```
/plugin marketplace add eershoov/builder-skills
/plugin install builder-skills@builder-skills
```

Skills then appear namespaced as `builder-skills:<skill>` (e.g. `builder-skills:validate-idea`).

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

## Development-process skill pipeline (project-spec phase)

The "raw idea → initial project documentation" flow is a fixed, phased pipeline. Each phase is
its own focused skill, adopts a persona, and writes one persistent artifact that the next phase
reads. The `create-project-spec` skill is a thin **orchestrator** that sequences the sub-skills,
pausing at each hard gate — it conducts, it does not duplicate phase logic.

| # | Skill | Persona | Artifact |
|---|-------|---------|----------|
| — | `create-project-spec` | Orchestrator (conductor) | — (sequences the steps below) |
| 1 | `validate-idea` | Founder-turned-investor | `docs/project-spec/idea-validation.md` |
| 2 | `define-product-requirements` | Product manager | `docs/project-spec/product-requirements.md` |
| 3 | `create-user-flows` | Product designer | `docs/project-spec/user-flows.md` |
| 4 | `design-architecture` | Software architect (requirements-first) | `docs/project-spec/architecture.md` (+ `docs/project-spec/adr/*`) |
| — | `review-doc` | Critic (rubric gate, reusable) | annotates the target doc |

**Artifact location & naming: all project-spec artifacts live in `docs/project-spec/` and are
named as descriptive nouns** (skills are verbs; their outputs are nouns) — `validate-idea` →
`docs/project-spec/idea-validation.md`, `define-product-requirements` →
`docs/project-spec/product-requirements.md`, `create-user-flows` →
`docs/project-spec/user-flows.md`, `design-architecture` → `docs/project-spec/architecture.md`
(+ `docs/project-spec/adr/`).

Conventions for these skills:

- **Artifacts live in the repo under `docs/project-spec/`.** Phase N reads phase N−1's artifact
  from there. Each skill creates the directory if it does not exist.
- **Hard gate between phases.** A phase skill must finish its artifact and get explicit user
  approval before the next phase starts. Never jump ahead to a later phase's concern.
- **The `create-project-spec` orchestrator conducts, never duplicates.** It invokes each sub-skill via the Skill
  tool, lets it run to its hard gate, gets approval, then advances. It resolves the language
  settings once and propagates them to every sub-skill. Each sub-skill remains independently
  runnable on its own.
- **Idea validation is adversarial.** A cheap KILL / SKIP / SHRINK pre-filter, then forcing
  questions (demand, audience specificity, problem validation, status-quo competitor, wedge,
  business model). Its only output is the validation doc — no solutioning.
- **Two layers: product, then technical.** `define-product-requirements` + `create-user-flows` form the
  product layer (WHAT and for WHOM — features, audience, user flows). `design-architecture` is
  the separate technical layer (HOW). Product-layer skills never make technical/stack decisions.
- **The product definition captures the full committed feature set — no prioritization.** No
  must/should/could tiers, no MVP cut line, no deferred-feature backlog. Everything in the
  feature list ships; a feature that doesn't belong is removed, not parked. Every feature traces
  to a validated need.
- **Architecture is requirements-first.** Elicit quality-attribute scenarios (cost, performance,
  security, reliability, scale) first; then propose 2–3 component options with trade-offs.
  Separate WHAT from HOW — do NOT commit to specific frameworks here. Record significant
  decisions as ADRs (options + ruled-out alternatives + trade-offs + status).

## Authoring conventions

- **Write all skill content (instructions, prompts, descriptions) in English**, regardless of
  the configured output/thinking languages. The languages above are runtime behavior, not the
  language the skills themselves are written in.
- **Write this CLAUDE.md and all repository documentation in English.**
- **Always write git commit messages in English.**

## Git workflow

- **Never create a separate feature branch unless explicitly asked to.** Work on the current
  branch by default; only branch when the user explicitly requests it.
