# Feature Spec Format

This repo uses a convention adapted from [OpenSpec](https://dev.to/webdeveloperhyper/how-to-make-ai-follow-your-instructions-more-for-free-openspec-2c85) for writing delta specifications in `features/<name>/specs/`. The format structures requirements as explicit, testable scenarios so that AI agents and engineers alike can understand exactly what behavior is expected.

---

## File structure

Each `specs/` file covers one domain area. Name files by domain:

```
features/<name>/specs/
  ui.md       # Frontend component and rendering behavior
  api.md      # Backend endpoints, Lambda handlers, data transforms
  infra.md    # CDK constructs, IAM, config, feature flags
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

Each requirement within a section gets a short title, followed by one or more scenarios:

```markdown
### <Requirement title>

#### Scenario: <description>

WHEN <precondition or trigger>
[AND <additional condition>]
THEN <expected outcome>
[AND <additional outcome>]
```

Rules:
- Every requirement must have at least one `#### Scenario:`
- `WHEN` describes the trigger or context
- `THEN` describes the observable outcome
- Use `AND` to chain conditions or outcomes
- Keep scenarios atomic — one behavior per scenario
- Write from the system's perspective, not implementation details

---

## Example

```markdown
## ADDED Requirements

### Personalized greeting

#### Scenario: User has a display name set

WHEN the user loads the dashboard
AND their profile has a `displayName`
THEN the header renders "Welcome back, <displayName>"

#### Scenario: User has no display name

WHEN the user loads the dashboard
AND their profile has no `displayName`
THEN the header renders "Welcome back"

## REMOVED Requirements

### Generic static greeting

#### Scenario: Any page load

WHEN the user loads the dashboard
THEN the static "Hello, Expert" string is not rendered
```

---

## Header boilerplate

Start every spec file with:

```markdown
# <Feature> — <Domain> Specs

Delta specifications for `<repo>` changes.
Format: [OpenSpec](https://dev.to/webdeveloperhyper/how-to-make-ai-follow-your-instructions-more-for-free-openspec-2c85)
```
