# User Flows — <Product name>

> Status: Draft | Date: <YYYY-MM-DD>
> Source: docs/project-spec/product-requirements.md

## Customer journey map — <primary persona>

| Stage | User goal | What they do | Touchpoint | Friction / emotion |
|-------|-----------|--------------|------------|--------------------|
| Discover | <goal> | <action> | <where> | <friction> |
| Onboard | | | | |
| First value ("aha") | | | | |
| Habitual use | | | | |
| Return / expand | | | | |

## User flows

### Flow: <name>
- **Serves (features):** <feature(s) from product-requirements.md>
- **Entry point:** <where the user comes from / trigger>

| # | User action / what they see | Branch → goes to |
|---|-----------------------------|------------------|
| 1 | <step> | <branch, if any> |
| 2 | <step> | |

- **Success outcome:** <what "done" looks like>
- **Alternate / error paths:** <important non-happy paths and where they lead>

### Flow: <name>
- **Serves (features):** <...>
- **Entry point:** <...>

| # | User action / what they see | Branch → goes to |
|---|-----------------------------|------------------|
| 1 | <step> | |

- **Success outcome:** <...>
- **Alternate / error paths:** <...>

## States & edge cases

| Flow / step | Empty / first-time | Loading | Error / retry | Success | Auth (signed-out / no access) |
|-------------|--------------------|---------|---------------|---------|-------------------------------|
| <flow> | <state> | <state> | <state> | <state> | <state> |

## Coverage check

- **Features without a flow:** <none, or list — must be resolved>
- **Flows needing a missing feature:** <none, or list → update product-requirements.md>

## Open questions

- <Unresolved question carried into the design-architecture step.>
