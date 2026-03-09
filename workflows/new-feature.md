# Starting a New Cross-Repo Feature

This workflow describes how to set up and maintain a feature directory in `features/` for work that spans multiple repos. It adapts the [OpenSpec](https://dev.to/webdeveloperhyper/how-to-make-ai-follow-your-instructions-more-for-free-openspec-2c85) proposal → apply → archive lifecycle for multi-repo orchestration.

See [`workflows/feature-specs.md`](feature-specs.md) for the spec delta format used in `specs/`.

---

## Directory structure

```
features/<name>/
  proposal.md     # Why this feature, what it does, cross-repo impact, success metrics
  tasks.md        # Implementation checklist, grouped by repo
  design.md       # Technical decisions, architecture, open questions
  specs/          # Delta specs per domain area (ADDED/MODIFIED/REMOVED + WHEN/THEN)
    ui.md
    api.md
    infra.md
  archive/        # Completed specs, Notion exports, historical research
```

---

## Lifecycle

### 1. Propose

Create `features/<name>/` with at minimum:
- `proposal.md` — fill in why, what, target, repos involved, success metrics
- `tasks.md` — stub with repo sections; items can be empty to start
- `design.md` — can be a stub; fill in as design firms up

Add the feature to the tables in both `CLAUDE.md` and `README.md`.

### 2. Spec

As design firms up, populate `specs/`:
- Create one file per domain area (ui, api, infra)
- Write delta requirements (ADDED / MODIFIED / REMOVED) with WHEN/THEN scenarios
- Update specs as decisions change — they are living docs until implementation is complete

### 3. Implement

Work through `tasks.md`:
- Check off items as PRs land (include PR numbers / branch names in the item if helpful)
- Keep `design.md` open questions current — resolve them and mark ✅ when answered
- Update `specs/` if behavior changes during implementation

### 4. Archive

When the feature ships:
- Move completed specs and historical docs (Notion exports, research) to `archive/`
- Mark `tasks.md` fully done or trim it
- Leave `proposal.md` and `design.md` in place as a permanent record

---

## Adding to CLAUDE.md and README.md

When you create a new feature, add it to the features table in both root files:

**CLAUDE.md:**
```markdown
| `features/<name>` | <JIRA-ID> | repo1, repo2, repo3 |
```

**README.md:**
```markdown
| [`<name>`](features/<name>/proposal.md) | <JIRA-ID> | repo1, repo2, repo3 |
```
