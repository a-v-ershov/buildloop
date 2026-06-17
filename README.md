# builder-skills

A collection of [Claude Code](https://claude.com/claude-code) skills for coding workflows,
distributed as a plugin you can install into any project from GitHub.

The centerpiece is **`create-project-spec`** — a thin orchestrator that takes a raw product idea
all the way to a buildable spec, one reviewed step at a time:

| Step | Skill | Writes |
|------|-------|--------|
| — | `create-project-spec` | (orchestrates the steps below) |
| 1 | `validate-idea` | `docs/project-spec/idea-validation.md` |
| 2 | `define-product-requirements` | `docs/project-spec/product-requirements.md` |
| 3 | `create-user-flows` | `docs/project-spec/user-flows.md` |
| 4 | `design-architecture` | `docs/project-spec/architecture.md` (+ `adr/`) |

Plus `commit` (intelligent, split-aware git commits). Each step stops at a hard gate for your
approval before the next runs, and each sub-skill is also runnable on its own.

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
`plugins/builder-skills/.claude-plugin/plugin.json` (and `metadata.version` in
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
(`plugins/builder-skills/`). After editing, validate the manifests:

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
