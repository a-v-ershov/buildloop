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
  skills/_shared/spec-pipeline/*.md      # shared methodology for the project-spec phases (no SKILL.md)
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

**Bump policy:** the `version` field is **owned by the user**. The agent MUST NOT edit it on its
own initiative under any circumstance — only when the user explicitly asks for a bump. The user
decides when a bump is due and will request it; the agent does not drive it. The agent may, at
most, mention in passing that skills changed, but MUST NOT change the version, propose a specific
number as an action, or treat a bump as pending work. Never auto-bump.

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
its own focused skill and adopts a persona. Internally, every phase runs the same machine —
**elicit → research (verify world-claims) → draft → adversarial review (a separate problems doc)
→ conflict gate → merge → dual output** — and keeps **two** files: a detailed source-cited
**research** doc and a short human summary. The reviewer's problems doc is **intermediate** —
applied at the merge stage, then deleted. The `create-project-spec` skill is a thin
**orchestrator** that sequences the sub-skills — it conducts, it does not duplicate phase logic.

| # | Skill | Persona | Research doc (+ `.summary.md`; review intermediate) |
|---|-------|---------|-----------------------------------------------------|
| — | `create-project-spec` | Orchestrator (conductor) | — (sequences the steps below; builds the final `summary.md`) |
| 1 | `validate-idea` | Founder-turned-investor | `docs/project-spec/idea-validation.research.md` |
| 2 | `define-product-requirements` | Product manager | `docs/project-spec/product-requirements.research.md` |
| 3 | `create-user-flows` | Product designer | `docs/project-spec/user-flows.research.md` |
| 4 | `design-architecture` | Software architect (requirements-first) | `docs/project-spec/architecture.research.md` (+ `docs/project-spec/adr/*`) |
| 5 | `design-dev-architecture` | DX / platform engineer | `docs/project-spec/dev-architecture.research.md` (+ `docs/project-spec/adr/*`) |

The adversarial review is **built into each phase** (a spawned reviewer subagent), not a separate
skill, and its file does not survive the merge. The shared machinery lives in
`skills/_shared/spec-pipeline/` (research method, review method, dual-output format, pipeline
config/modes, and the summary + review templates).

**Artifact location & naming: all project-spec artifacts live in `docs/project-spec/` and are
named as descriptive nouns** (skills are verbs; their outputs are nouns) — `validate-idea` →
`idea-validation`, `define-product-requirements` → `product-requirements`, `create-user-flows` →
`user-flows`, `design-architecture` → `architecture` (+ `adr/`), `design-dev-architecture` →
`dev-architecture` (+ `adr/`), all under `docs/project-spec/`. **Each phase keeps a pair**:
`<noun>.research.md` (detailed, source-cited, for the AI/next phase) and `<noun>.summary.md`
(short, for the human — essence + forks to answer). A transient `<noun>.review.md` (the
reviewer's inconsistencies + gaps) is applied at the merge stage and then **deleted** — its
findings live on in the research doc and its Forks / Decisions log. The orchestrator additionally
builds a combined human-facing `docs/project-spec/summary.md` at the end of a run.

Conventions for these skills:

- **Artifacts live in the repo under `docs/project-spec/`.** Phase N reads phase N−1's research
  doc from there. Each skill creates the directory if it does not exist.
- **Every phase researches and reviews itself.** It verifies its world-claims against real
  sources (adaptive depth: light web search by default, `/deep-research` only when warranted or
  requested), cites them inline + in a `## Sources` section, then spawns a separate reviewer
  subagent that writes `<noun>.review.md`, merges the corrections back, and **deletes the review
  file** after merging (only the research doc + summary survive). Modeled on the `vibecoding_course`
  `video-research` skill (research → проверка → synthesis).
- **Two run modes, set once.** `create-project-spec` (or the first standalone phase) asks two
  settings and writes `docs/project-spec/.spec-config.md`: `mode` (`interactive` | `autopilot`)
  and `final_summary` (`true` | `false`). **interactive** pauses at each fork and each phase's
  hard gate. **autopilot** lets the AI resolve every fork itself and run phases back-to-back —
  but it still researches, reviews, and writes both kept files, and **logs every fork** (choice +
  rationale + confidence) in the doc's `## Forks / Decisions log`; low/medium-confidence forks
  surface in the human summary as "must answer". Autopilot changes *who answers*, never *whether
  it's recorded*. Each phase reads the config and is independently runnable (asks the two settings
  itself when the config is absent).
- **The `create-project-spec` orchestrator conducts, never duplicates.** It invokes each sub-skill via the Skill
  tool, lets it run its internal pipeline, then advances per the mode (approval gate in
  interactive, straight through in autopilot). Each sub-skill responds in the user's language on
  its own, so the whole pipeline stays consistent. Each sub-skill remains independently runnable.
- **Idea validation is adversarial.** A cheap KILL / SKIP / SHRINK pre-filter, then forcing
  questions (demand, audience specificity, problem validation, status-quo competitor, wedge,
  business model). Its outputs are the validation research doc + its human summary — no solutioning.
- **Two layers: product, then technical (two steps).** `define-product-requirements` +
  `create-user-flows` form the product layer (WHAT and for WHOM — features, audience, user flows).
  The technical layer is **two** steps: `design-architecture` (the system/production architecture —
  components + the concrete technologies that realize them, co-designed because the toolbox shapes
  the decomposition) then `design-dev-architecture` (the inner loop / developer experience — how to
  run the product locally with prod-parity stand-ins, how to test it in an AI-drivable way, and how
  to configure the AI tooling for the stack). `design-dev-architecture` never re-opens the stack or
  redraws the architecture; product-layer skills never make technical decisions.
- **The product definition captures the full committed feature set — no prioritization.** No
  must/should/could tiers, no MVP cut line, no deferred-feature backlog. Everything in the
  feature list ships; a feature that doesn't belong is removed, not parked. Every feature traces
  to a validated need.
- **Architecture is requirements-first, then co-designed with the toolbox.** Elicit measurable
  quality-attribute scenarios (cost, performance, security, reliability, scale) **first — before
  naming any tool**; this ordering is the guard against tool-first design. Then, for each
  significant component, propose 2–3 *integrated* options that bundle structure + concrete tools,
  weigh them against the scenarios and the team/budget constraints, and recommend one — scenario-
  driven, not hype-driven. Default to proven tech and the fewest moving parts (prefer an option
  that collapses components unless a scenario forbids it). Keep each component labelled with its
  logical role and record significant decisions as ADRs (options + ruled-out alternatives +
  trade-offs + status) so a later tool swap stays cheap.
- **Dev architecture is the inner loop, AI-first.** `design-dev-architecture` takes the chosen
  stack and designs three things together: a **prod-parity local run** (local stand-ins whose APIs
  mirror the production services, one-command bring-up, seed data — every divergence named as a
  risk), **AI-drivable testing** (test levels + an e2e harness an agent can run and verify with no
  manual step), and **AI tooling tuned to the stack** (Claude Code config, MCP servers, plugins/
  skills, other agents). Minimal, proven infra — never reproduce production scale/HA locally. It
  continues the `adr/` numbering from `design-architecture` and never re-opens stack choices.

## Authoring conventions

- **Write all skill content (instructions, prompts, descriptions) in English**, regardless of the
  language a skill responds in at runtime. Responding in the user's language is runtime behavior,
  not the language the skills themselves are written in.
- **Write this CLAUDE.md and all repository documentation in English.**
- **Always write git commit messages in English.**

## Git workflow

- **Never create a separate feature branch unless explicitly asked to.** Work on the current
  branch by default; only branch when the user explicitly requests it.
