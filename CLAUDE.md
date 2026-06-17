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

## Core design requirement: respond in the user's language

Every skill in this repository MUST **respond in, and reason in, the language the user addressed
it in**. If the user wrote in Russian, the skill answers and thinks in Russian; in English, it
uses English; and so on. There is nothing to configure — detect the user's language from their
message and match it.

Guidelines:

- This applies only to natural-language text (questions, summaries, reports, explanations). It
  MUST NOT translate code, identifiers, file paths, commands, or API names.
- When a skill spawns subagents, instruct them to follow the same rule, so the whole flow speaks
  the user's language consistently.
- Commit messages are the one fixed exception: they are ALWAYS written in English (see the
  `commit` skill), regardless of the user's language.

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
/plugin marketplace add a-v-ershov/builder-skills
/plugin install builder-skills@builder-skills
```

Skills then appear namespaced as `builder-skills:<skill>` (e.g. `builder-skills:validate-idea`).

### Versioning

The plugin carries an explicit `version` in `plugins/builder-skills/.claude-plugin/plugin.json`
(semver). Consumers only receive changes via `/plugin update` when this version is **bumped** —
pushing skill changes without bumping is ignored downstream.

**Bump policy:** whenever skills change, the version MUST be bumped before push. The agent must
**remind the user** that a bump is due and **propose** the new version, but MUST NOT bump it
without the user's explicit confirmation. Never auto-bump.

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
  tool, lets it run to its hard gate, gets approval, then advances. Each sub-skill responds in the
  user's language on its own, so the whole pipeline stays consistent. Each sub-skill remains
  independently runnable on its own.
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

- **Write all skill content (instructions, prompts, descriptions) in English**, regardless of the
  language a skill responds in at runtime. Responding in the user's language is runtime behavior,
  not the language the skills themselves are written in.
- **Write this CLAUDE.md and all repository documentation in English.**
- **Always write git commit messages in English.**

## Git workflow

- **Never create a separate feature branch unless explicitly asked to.** Work on the current
  branch by default; only branch when the user explicitly requests it.
