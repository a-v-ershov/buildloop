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

The plugin is collapsed into the repo root: the marketplace catalog and the plugin manifest
both live in `.claude-plugin/`, and the plugin's components sit at the repo root.

```
.claude-plugin/marketplace.json          # marketplace catalog (lists the plugin; source: "./")
.claude-plugin/plugin.json               # the plugin manifest
skills/<name>/SKILL.md                   # one directory per skill
skills/<name>/references/*.md            # bundled templates/rubrics (progressive disclosure)
skills/_shared/spec-pipeline/*.md        # shared methodology for the project-spec phases (no SKILL.md)
skills/_shared/build-pipeline/*.md       # shared methodology for the build phase (no SKILL.md)
skills/_shared/agent-guide.md            # shared: the project CLAUDE.md "project map" block (cross-cutting, no SKILL.md)
CLAUDE.md
README.md
```

- Skills live under `skills/` at the repo root. Component dirs (`skills/`, and later
  `commands/`, `agents/`) sit at the **plugin root**, not inside `.claude-plugin/`.
- `SKILL.md` = YAML frontmatter (`name` + `description`) + a thin procedure with a copyable
  checklist. Long templates go in `references/` and load on demand.
- Marketplace plugin `source` must be a relative path starting with `./`. Because the plugin is
  the repo root, the source is `"./"` (the marketplace-root source). A bare `"."` is NOT a valid
  source — relative paths must start with `./`.
- Validate any change with `claude plugin validate .`.

### Installing the skills into another project

```
/plugin marketplace add a-v-ershov/builder-skills
/plugin install builder-skills@builder-skills
```

Skills then appear namespaced as `builder-skills:<skill>` (e.g. `builder-skills:validate-idea`).

### Versioning

The plugin carries an explicit `version` in `.claude-plugin/plugin.json`
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
→ conflict gate → merge → dual output** (where **elicit** is the shared, grill-style interview
technique — `_shared/spec-pipeline/elicitation-method.md`) — and keeps **two** files: a detailed
source-cited **research** doc and a short human summary. The reviewer's problems doc is
**intermediate** — applied at the merge stage, then deleted. The `create-project-spec` skill is a
thin **orchestrator** that sequences the sub-skills — it conducts, it does not duplicate phase
logic. The pipeline **opens with `gather-context`** (step 1): a discovery interview that turns the
user's short brief into a rich shared understanding (`project-brief.research.md`) every later phase
reads as settled intent. `gather-context` runs a lighter variant of the machine (interview-led,
light research, a coverage-critic review) and is **also a reusable grill** — any phase can invoke
it scoped to a fork blocked on context only the user holds, and the user can invoke it directly at
any time to be interviewed on any topic.

| # | Skill | Persona | Research doc (+ `.summary.md`; review intermediate) |
|---|-------|---------|-----------------------------------------------------|
| — | `create-project-spec` | Orchestrator (conductor) | — (sequences the steps below; builds the final `summary.md`) |
| 1 | `gather-context` | Discovery interviewer (the grill) | `docs/project-spec/project-brief.research.md` |
| 2 | `validate-idea` | Founder-turned-investor | `docs/project-spec/idea-validation.research.md` |
| 3 | `define-product-requirements` | Product manager | `docs/project-spec/product-requirements.research.md` |
| 4 | `create-user-flows` | Product designer | `docs/project-spec/user-flows.research.md` |
| 5 | `define-design-decisions` | Design-system lead | `docs/project-spec/design-decisions.research.md` |
| 6 | `design-architecture` | Software architect (requirements-first) | `docs/project-spec/architecture.research.md` (+ `docs/project-spec/adr/*`) |
| 7 | `design-dev-architecture` | DX / platform engineer | `docs/project-spec/dev-architecture.research.md` (+ `docs/project-spec/adr/*`) |

The adversarial review is **built into each phase** (a spawned reviewer subagent), not a separate
skill, and its file does not survive the merge. The shared machinery lives in
`skills/_shared/spec-pipeline/` (the elicitation/interview method, research method, review method,
dual-output format, pipeline config/modes, and the summary + review templates).

**Artifact location & naming: all project-spec artifacts live in `docs/project-spec/` and are
named as descriptive nouns** (skills are verbs; their outputs are nouns) — `gather-context` →
`project-brief`, `validate-idea` →
`idea-validation`, `define-product-requirements` → `product-requirements`, `create-user-flows` →
`user-flows`, `define-design-decisions` → `design-decisions`, `design-architecture` →
`architecture` (+ `adr/`), `design-dev-architecture` → `dev-architecture` (+ `adr/`), all under
`docs/project-spec/`. **Each phase keeps a pair**:
`<noun>.research.md` (detailed, source-cited, for the AI/next phase) and `<noun>.summary.md`
(the human report — maximally compressed and decisions-first: what you must answer, then risks, then
a few key facts; the only artifact written for the human). A transient `<noun>.review.md` (the
reviewer's inconsistencies + gaps) is applied at the merge stage and then **deleted** — its
findings live on in the research doc and its Forks / Decisions log. The orchestrator additionally
builds a combined human-facing `docs/project-spec/summary.md` at the end of a run.

**Commit policy: everything kept under `docs/project-spec/` is committed project documentation** —
the `<noun>.research.md` (its Forks / Decisions log is the audit trail of *why*), the
`<noun>.summary.md`, the combined `summary.md`, the `adr/*`, and `.spec-config.md`. The only
exception is the transient `<noun>.review.md`, which is **deleted after merge** and additionally
gitignored (belt-and-suspenders against an aborted run) via a local `docs/project-spec/.gitignore`
containing `*.review.md`. The artifacts are never hidden in a dot-dir — they are documentation,
and `docs/` is their visible, conventional home.

Conventions for these skills:

- **Artifacts live in the repo under `docs/project-spec/`.** Phase N reads phase N−1's research
  doc from there. Each skill creates the directory if it does not exist — and, when creating it,
  drops a `docs/project-spec/.gitignore` containing `*.review.md` if absent.
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
- **Context-gathering is a grill, and it's reusable.** `gather-context` opens the pipeline: after
  the user's short brief it runs an iterative interview — one thread at a time, a recommended answer
  on every question, pushing past the first answer, mirroring back to confirm — until shared
  understanding is reached, then writes the discovery brief (`project-brief.research.md` + summary).
  It captures the user's *intent* (goal, audience, scope, constraints, taste), never validating the
  idea or defining features (those are later phases) and never solutioning. The same skill is
  callable on demand: any phase can invoke it scoped to a fork blocked on context only the user
  holds (returning the gathered answers), and the user can run it directly to be interviewed on any
  topic. The interview technique is shared (`_shared/spec-pipeline/elicitation-method.md`) and every
  phase's elicit step uses it.
- **Idea validation is adversarial.** A cheap KILL / SKIP / SHRINK pre-filter, then forcing
  questions (demand, audience specificity, problem validation, status-quo competitor, wedge,
  business model). Its outputs are the validation research doc + its human summary — no solutioning.
- **Two layers, bridged by design decisions: product → (design) → technical.**
  `define-product-requirements` + `create-user-flows` form the product layer (WHAT and for WHOM —
  features, audience, user flows). `define-design-decisions` is the **bridge**: it sets the design
  direction (design system, key screens, viewports, target platforms, media-heaviness, offline,
  accessibility) — design decisions only, never mockups or code (the cheapest mockup is real
  rendered code at implementation time) — and hands the technically-weighty ones to the next step
  as quality-attribute scenario inputs. The technical layer is **two** steps: `design-architecture`
  (the system/production architecture — components + the concrete technologies that realize them,
  co-designed because the toolbox shapes the decomposition) then `design-dev-architecture` (the
  inner loop / developer experience — how to run the product locally with prod-parity stand-ins,
  how to test it in an AI-drivable way, and how to configure the AI tooling for the stack).
  `design-dev-architecture` never re-opens the stack or redraws the architecture;
  `define-design-decisions` and the product-layer skills never make technical decisions.
- **The product definition captures the full committed feature set — no prioritization.** No
  must/should/could tiers, no MVP cut line, no deferred-feature backlog. Everything in the
  feature list ships; a feature that doesn't belong is removed, not parked. Every feature traces
  to a validated need **and carries at least one behavioral, testable acceptance criterion**
  (Given/When/Then or EARS) — the verifiable definition of done that the `design-dev-architecture`
  verification loop later proves against. `define-product-requirements` also keeps the **conceptual
  domain model + glossary** (entities + one canonical vocabulary, reused by every later phase, NOT
  a database schema) — this lives in the research doc only; the human summary stays non-technical
  (key concepts at most). User flows then carry their own acceptance criteria on each flow's
  success outcome and significant states.
- **Architecture is requirements-first, then co-designed with the toolbox.** Elicit measurable
  quality-attribute scenarios (cost, performance, security, reliability, scale) **first — before
  naming any tool**; this ordering is the guard against tool-first design. Then, for each
  significant component, propose 2–3 *integrated* options that bundle structure + concrete tools,
  weigh them against the scenarios and the team/budget constraints, and recommend one — scenario-
  driven, not hype-driven. Default to proven tech and the fewest moving parts (prefer an option
  that collapses components unless a scenario forbids it). Keep each component labelled with its
  logical role and record significant decisions as ADRs (options + ruled-out alternatives +
  trade-offs + status) so a later tool swap stays cheap. **For security-sensitive products** (money,
  PII/credentials, shared/multi-tenant access) it also runs a **STRIDE-lite threat model** over the
  component map + trust boundaries (assets, attack surfaces, threats, mitigations) and folds the
  mitigations back into the design + ADRs; for a product with no sensitive assets it records that
  it skipped it and why.
- **Dev architecture is the inner loop, AI-first.** `design-dev-architecture` takes the chosen
  stack and designs three things together: a **prod-parity local run** (local stand-ins whose APIs
  mirror the production services, one-command bring-up, seed data — every divergence named as a risk —
  plus an **environment-access model**, an advisory lock and/or per-run isolation, so concurrent actors
  never clobber the single shared env), **AI-drivable testing** (test levels + **purpose-built
  developer/test scripts** that deliberately diverge from prod for fast iteration — distinct from the
  parity stand-ins, the divergence an intentional speed tradeoff — + an e2e harness an agent can run and
  verify with no manual step), and **AI tooling tuned to the stack** (Claude Code config, MCP servers,
  plugins/skills, other agents). Minimal, proven infra — never reproduce production scale/HA locally. It
  continues the `adr/` numbering from `design-architecture` and never re-opens stack choices.

## Development-process skill pipeline (build phase)

After the spec is complete, a second pipeline turns it into working software. Unlike the spec phase
(which only writes documents), the build phase **mutates the real repository** — it scaffolds, runs
commands, and writes code — so it is a categorically different, side-effecting pipeline. It is
**sequential**: one task at a time, on a single working tree, no worktrees and no parallelism (a
deliberate choice — parallel worktrees were rejected for their runtime-isolation, merge, and
orchestration cost). Its shared machinery lives in `skills/_shared/build-pipeline/`.

| # | Skill | Role | Reads / Writes |
|---|-------|------|----------------|
| — | `build-product` | Orchestrator (conductor) | the backlog → the build loop |
| 1 | `setup-dev-environment` | Platform / release engineer | dev-architecture.research.md → scaffolded repo + `docs/project-setup/` |
| 2 | `plan-development` | Delivery tech lead | the spec → `docs/build-plan/` kanban backlog |
| 3 | `implement-feature` | Implementer | one task → code |
| 4 | `verify-feature` | Independent verifier (separate agent) | a task → adversarial tests + pass/fail verdict |
| — | `propagate-changes` | Cross-cutting conductor | a changed spec doc → reconciled downstream docs + backlog |

Conventions for these skills:

- **The build phase mutates the repo; the spec phase did not.** `setup-dev-environment` is the boundary
  `design-dev-architecture` stopped at — it executes the documented inner loop (installs, compose, seed,
  one-command bring-up). It plans everything but auto-executes only repo-local scaffolding; global
  installs, API keys, and plugin installs run only with explicit confirmation (the `careful` pattern),
  and it is idempotent (detect-state-first, back up before overwrite). It also stands up the **enforced
  quality gate** (linter + type-checker + tests behind one `make check`, zero-tolerance, a pre-commit
  hook that blocks the commit on red, a Stop hook that feeds failures back —
  `skills/_shared/build-pipeline/quality-gate.md`). It also bakes in the **environment-access mechanism**
  (lock and/or per-run isolation — `skills/_shared/build-pipeline/env-access.md`) and scaffolds the
  **developer/test-script skeletons** the spec named (built out later as backlog tasks). It ends with a
  **smoke-test** (the stack comes up *and* the gate has teeth) rather than the spec phase's adversarial reviewer.
- **The backlog is a kanban board with blockers — own format, not a library.** One markdown file per
  task under `docs/build-plan/tasks/` (status in frontmatter; each agent edits only its own file). A
  task's `blocked_by` list *is* the dependency graph, implicitly; `ready` = `todo` with all blockers
  `done`. `board.md` is a derived view, regenerated, never hand-edited. Format + lifecycle:
  `skills/_shared/build-pipeline/backlog-format.md`.
- **`build-product` conducts; it does not duplicate.** It picks one `ready` task at a time (lowest id),
  **spawns `implement-feature` as a fresh subagent** (fresh per task, kept across that task's rounds so
  it remembers what it tried), then spawns `verify-feature` as a separate agent, loops them (bounded by
  `max_verify_iterations`, default 4 — at the cap the task goes `needs_human`), and on pass — **only once
  the full quality gate is green** — sets the task `done` and makes a checkpoint commit (via the `commit`
  skill, carrying the task id). Resumable — the backlog is the source of truth.
- **Verification runs in a separate, unbiased agent that authors the adversarial tests.** `verify-feature`
  is generic — it reads the project-specific run/drive/prove commands from `docs/project-setup/verification.md`
  (written by `setup-dev-environment`) and the task's own acceptance criteria, **authors adversarial
  automated tests for those criteria** (committed — the implementer may also write its own tests for a fast
  self-check, but the adversarial layer comes from the side that didn't write the code, and the verifier
  never touches the implementation), and proves observable outcomes (a screenshot, a DB row, a log line, an
  asserted response) — never "it ran", and never "tests are green" alone. A fresh agent (not the implementer)
  is what actually moves a task to `done`; the accumulated tests are the regression net the quality gate runs.
- **Two run modes, like the spec phase.** `docs/build-plan/.build-config.md` holds `mode`
  (`interactive` | `autopilot`) and `max_verify_iterations`. Two things always stop regardless of mode:
  a `needs_human` escalation, and a critical/destructive change-propagation step.
- **Change propagation reconciles downstream after a spec edit.** When a stage document changes,
  `propagate-changes` walks the chain **forward** — each downstream spec skill has an **amend mode** that
  surgically updates its own doc (preserving its Forks/Decisions log) or self-skips if unaffected — then
  continues into the backlog via `plan-development`'s amend mode (task deltas: add / modify / cancel /
  reopen-as-rework). It runs automatically, asking only on critical or destructive questions, and never
  writes code — rebuilding affected features is a separate `build-product` run. Detection is by reading
  the files directly (no git/checksum staleness check). Method:
  `skills/_shared/build-pipeline/propagation-method.md`.
- **Artifacts live under `docs/build-plan/` (backlog, board, plan summary) and `docs/project-setup/`
  (setup log, verification contract) — both committed project documentation.** Skills are verbs; their
  outputs are nouns.

## Project documentation map (the target project's `CLAUDE.md`)

Both pipelines write `docs/`; an agent later working in the project needs to find its way around them.
So the target project's **root `CLAUDE.md`** carries a small, marker-delimited **project documentation
map** — a navigational index of `docs/project-spec/`, `docs/build-plan/`, and `docs/project-setup/`
plus the order to read them in before changing code. It is a **map, not a copy**: it points at the
artifacts, never restates them. This is a deliberately separate concern from the *built-in* `/init`
(which writes a `CLAUDE.md` from analysing existing code) — the map is spec/backlog-aware, not
code-derived.

The behavior is defined once, in the shared spec **`skills/_shared/agent-guide.md`**, and three skills
render the **same** marked block at natural moments (so nothing is duplicated): `create-project-spec`
**seeds** it at run start (artifacts shown as *planned*) and **finalizes** it at the end;
`setup-dev-environment` writes it inside the project `CLAUDE.md` it scaffolds (next to its stack notes
+ commands, which live outside the markers); `plan-development` **refreshes** it once the backlog
exists. The operation is **idempotent and non-destructive** — writers touch only the content between
`<!-- builder-skills:project-map:start -->` and `<!-- builder-skills:project-map:end -->`, never the
user's own `CLAUDE.md` content. There is no separate skill for this — it is shared methodology, like
the research/review/output-format docs.

## Authoring conventions

- **Write all skill content (instructions, prompts, descriptions) in English**, regardless of the
  language a skill responds in at runtime. Responding in the user's language is runtime behavior,
  not the language the skills themselves are written in.
- **Write this CLAUDE.md and all repository documentation in English.**
- **Always write git commit messages in English.**

## Git workflow

- **Never create a separate feature branch unless explicitly asked to.** Work on the current
  branch by default; only branch when the user explicitly requests it.
