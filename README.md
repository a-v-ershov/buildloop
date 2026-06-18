# builder-skills

A collection of [Claude Code](https://claude.com/claude-code) skills for coding workflows,
distributed as a plugin you can install into any project from GitHub.

The centerpiece is **`create-project-spec`** — a thin orchestrator that takes a raw product idea
all the way to a buildable spec, one reviewed step at a time:

| Step | Skill | Writes |
|------|-------|--------|
| — | `create-project-spec` | (orchestrates the steps below) |
| 1 | `gather-context` | `docs/project-spec/project-brief.research.md` (+ `.summary.md`) |
| 2 | `validate-idea` | `docs/project-spec/idea-validation.research.md` (+ `.summary.md`) |
| 3 | `define-product-requirements` | `docs/project-spec/product-requirements.research.md` (+ `.summary.md`) |
| 4 | `create-user-flows` | `docs/project-spec/user-flows.research.md` (+ `.summary.md`) |
| 5 | `define-design-decisions` | `docs/project-spec/design-decisions.research.md` (+ `.summary.md`) |
| 6 | `design-architecture` | `docs/project-spec/architecture.research.md` (+ `adr/`) |
| 7 | `design-dev-architecture` | `docs/project-spec/dev-architecture.research.md` (+ `adr/`) |

The pipeline opens with **`gather-context`** — a discovery grill that interviews you after your
short brief (one question at a time, always with a recommended answer) until you and it share the
same understanding of what to build, then writes a project brief every later phase reads. It's also
reusable: any phase can call it when a fork needs context only you hold, and you can invoke it
directly anytime to be interviewed on a topic.

Plus `commit` (intelligent, split-aware git commits). Each step stops at a hard gate for your
approval before the next runs, and each sub-skill is also runnable on its own.

Everything under `docs/project-spec/` is **committed project documentation** (research docs,
summaries, ADRs, and `.spec-config.md`); the only transient file is `*.review.md`, which is
deleted after merge and gitignored via a local `docs/project-spec/.gitignore`.

Once the spec exists, the **build phase** turns it into working software — sequentially, one task at
a time, on a single working tree (no parallelism):

| Step | Skill | Writes |
|------|-------|--------|
| — | `build-product` | (orchestrates the loop below) |
| 1 | `setup-dev-environment` | the scaffolded repo + enforced quality gate + `docs/project-setup/` (setup log, verification contract) |
| 2 | `plan-development` | `docs/build-plan/` — a kanban backlog, one markdown file per task |
| 3 | `implement-feature` | the feature, in the working tree |
| 4 | `verify-feature` | adversarial tests + a pass/fail verdict — a separate, unbiased agent |

`build-product` picks one ready task at a time (a task whose blockers are all `done`), builds it in a
fresh per-task agent, verifies it in a separate agent that authors adversarial tests (bounded — at the
cap a task escalates to `needs_human`), and — once the quality gate is green — makes a checkpoint commit
carrying the task id. When the spec changes later, **`propagate-changes`** walks the
edit forward through the spec docs and into the backlog, surgically, asking only on critical or
destructive changes. Everything under `docs/build-plan/` and `docs/project-setup/` is committed
project documentation.

Throughout, the project's own root **`CLAUDE.md` carries a "project documentation map"** — a small
marker-delimited block that indexes the spec, the backlog, and the setup contract and tells any
coding agent the order to read them in before touching code. `create-project-spec` seeds it at the
start (artifacts shown as *planned*) and finalizes it at the end; `setup-dev-environment` writes it
alongside its stack notes; `plan-development` refreshes it once the backlog exists. It is idempotent
and non-destructive — only the marked block is ever touched. Shared spec:
`skills/_shared/agent-guide.md`.

Every skill **responds in, and thinks in, whatever language you address it in** — write to it in
Russian and it answers in Russian, in English and it answers in English. Nothing to configure.
(Commit messages are the one exception: always written in English.)

## Install

In any project, inside Claude Code:

```
/plugin marketplace add https://github.com/a-v-ershov/builder-skills.git
/plugin install builder-skills@builder-skills
```

Skills then appear namespaced as `builder-skills:<skill>`, e.g. `builder-skills:create-project-spec`.

## Update

The plugin carries an explicit `version`, so an install only picks up changes once that version
is **bumped** — pushing skill changes without a bump is ignored downstream.

**Maintainer side** (this repo): change the skills, bump `version` in
`.claude-plugin/plugin.json` (and `metadata.version` in
`.claude-plugin/marketplace.json`), then push.

**Consumer side** (any project that has it installed): refresh the catalog, update the plugin,
then restart Claude Code (an update needs a restart to apply):

```
/plugin marketplace update builder-skills
/plugin update builder-skills@builder-skills
```

Or enable auto-update once — `/plugin` → **Marketplaces** → `builder-skills` →
**Enable auto-update** — and every startup pulls the latest version and prompts `/reload-plugins`.
(Third-party marketplaces have auto-update off by default.)

## Develop

This repo is both the marketplace (`.claude-plugin/marketplace.json`) and the plugin it ships
— the plugin is collapsed into the repo root (`.claude-plugin/plugin.json` + `skills/`), so the
marketplace entry uses `source: "./"`. After editing, validate the manifests:

```
claude plugin validate .
```

To test locally before pushing, add the marketplace from a local path:

```
/plugin marketplace add ./
/plugin install builder-skills@builder-skills
```

### Git hooks

This repo ships a `commit-msg` hook in `.githooks/` that blocks any commit whose message
contains an AI attribution trailer (`Co-Authored-By: Claude`, "Generated with Claude", etc.).
Enable it once per clone:

```
git config core.hooksPath .githooks
```

See `CLAUDE.md` for the full design conventions.
