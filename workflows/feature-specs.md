# Feature Spec Format

This repo uses a convention adapted from [OpenSpec](https://github.com/Fission-AI/OpenSpec) for writing delta specifications in `features/<name>/specs/`. The format structures requirements as explicit, testable scenarios so that AI agents and engineers alike can understand exactly what behavior is expected.

See also: `openspec/` for the source-of-truth specs that accumulate as features ship.

---

## File structure

Each `specs/` file covers one domain area. Name files by domain:

```
features/<name>/specs/
  ui.md       # Frontend component and rendering behavior
  api.md      # Backend endpoints, Lambda handlers, data transforms
  infra.md    # CDK constructs, IAM, Kafka connectors, feature flags
```

---

## Delta format

Each file describes what changed relative to the previous state of the system. Use these section headers:

```markdown
## ADDED Requirements
## MODIFIED Requirements
## REMOVED Requirements
## RENAMED Requirements
```

Only include sections that apply. Requirements without changes don't appear.

---

## Requirement format

Each requirement gets a short title and a requirement statement using RFC 2119 keywords, followed by one or more scenarios:

```markdown
### Requirement: <title>
<One-line statement using MUST/SHALL/SHOULD/MAY>

#### Scenario: <description>

- GIVEN <precondition or system state>
- [AND <additional precondition>]
- WHEN <trigger or action>
- [AND <additional trigger>]
- THEN <expected observable outcome>
- [AND <additional outcome>]
```

**RFC 2119 keywords:**

| Keyword | Meaning |
|---------|---------|
| MUST / SHALL | Absolute requirement — no exceptions |
| SHOULD | Recommended — exceptions exist but must be justified |
| MAY | Optional |

Rules:
- Every requirement MUST have at least one `#### Scenario:`
- `GIVEN` describes the precondition or system state
- `WHEN` describes the trigger or action
- `THEN` describes the observable outcome
- Use `AND` to chain conditions or outcomes
- Keep scenarios atomic — one behavior per scenario
- Write from the system's perspective, not implementation details
- No internal class/function names, library choices, or CSS specifics in specs (those belong in `design.md`)

---

## Example

```markdown
## ADDED Requirements

### Requirement: Personalized greeting
The system MUST display a personalized greeting when the user has a display name configured.

#### Scenario: User has a display name set

- GIVEN the user's profile has a `displayName`
- WHEN the user loads the dashboard
- THEN the header renders "Welcome back, <displayName>"

#### Scenario: User has no display name

- GIVEN the user's profile has no `displayName`
- WHEN the user loads the dashboard
- THEN the header renders "Welcome back"

## REMOVED Requirements

### Requirement: Generic static greeting

#### Scenario: Any page load

- GIVEN any authenticated user
- WHEN the user loads the dashboard
- THEN the static "Hello, Expert" string is not rendered
```

---

## Header boilerplate

Start every spec file with:

```markdown
# <Feature> — <Domain> Specs

Delta specifications for `<repo>` changes.
Format: [OpenSpec](https://github.com/Fission-AI/OpenSpec)
```

---

## Lifecycle

When a feature ships, its delta specs merge into `openspec/specs/`:
1. Run `/opsx:archive` — this merges `features/<name>/specs/` deltas into `openspec/specs/<domain>/spec.md`
2. Move feature artifacts to `features/<name>/archive/`

The `openspec/specs/` accumulates the full source-of-truth picture of the platform over time.
