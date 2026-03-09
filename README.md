# Agent Orchestrator

Homebase for cross-repo engineering work across the Expert Workspace platform. This is where context lives for features that span multiple repos, reusable workflows are documented, and integrations with tools like Notion, Jira, Slack, and GitHub are built out over time.

## Structure

```
repos/        # Symlinked CLAUDE.md files for each repo — context when working cross-repo
features/     # Cross-repo feature context: Notion exports, engineering notes, trackers
workflows/    # Repeatable process docs (markdown)
```

## Repos

Each subdirectory under `repos/` contains a `CLAUDE.md` symlink pointing to the live file in the actual repo, providing up-to-date conventions when working cross-repo.

| Dir | Repo | Description |
|---|---|---|
| `repos/expert-workspace` | [expert-workspace](https://github.com/Soluto-Private/expert-workspace) | React 19 SPA monorepo — expert-facing UI platform |
| `repos/expert-workspace-api` | [expert-workspace-api](https://github.com/Soluto-Private/expert-workspace-api) | Node.js API monorepo — Lambda + Fastify, DynamoDB, PostgreSQL, Redis |
| `repos/expert-workspace-config` | [expert-workspace-config](https://github.com/Soluto-Private/expert-workspace-config) | AWS CDK infrastructure config |

## Features

Cross-repo features live under `features/`. Each directory follows an [OpenSpec](https://dev.to/webdeveloperhyper/how-to-make-ai-follow-your-instructions-more-for-free-openspec-2c85)-inspired structure:

```
features/<name>/
  proposal.md   # Why, what, cross-repo impact, success metrics
  tasks.md      # Implementation checklist grouped by repo
  design.md     # Architecture, technical decisions, open questions
  specs/        # Delta specs (ADDED/MODIFIED/REMOVED + WHEN/THEN scenarios)
  archive/      # Historical docs, Notion exports, completed specs
```

| Feature | Jira | Repos |
|---|---|---|
| [`dynamic-smart-offer`](features/dynamic-smart-offer/proposal.md) | RDESQ-1900 | expert-workspace, expert-workspace-api, expert-workspace-config |

## Workflows

Reusable process docs live under `workflows/`.

| Workflow | Description |
|---|---|
| [`new-feature.md`](workflows/new-feature.md) | Start a new cross-repo feature — directory structure and lifecycle |
| [`feature-specs.md`](workflows/feature-specs.md) | OpenSpec-style delta spec format for `specs/` files |

## Usage with Claude Code

This repo is designed to be used as a working context hub with [Claude Code](https://claude.ai/claude-code). Open it as your working directory when doing cross-repo work — `CLAUDE.md` and the `features/` docs are loaded automatically to give Claude full context on conventions, architecture, and current state.

Each `repos/*/CLAUDE.md` symlink points to the live file in its source repo, so Claude always has current, repo-specific conventions without duplication.

## Maintaining This Repo

This repo is a living document. Keep it updated as work progresses:

- Update `features/*/tasks.md` as items are completed
- Add new feature directories (following `workflows/new-feature.md`) when starting cross-repo work
- Add workflow docs to `workflows/` as processes are established
- Update repo `CLAUDE.md` files (via the source repos) when conventions change
