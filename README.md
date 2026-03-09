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

Cross-repo features live under `features/`. Each directory contains context docs, Notion exports, engineering notes, open questions, and an implementation tracker for work that touches multiple repos.

| Feature | Jira | Repos |
|---|---|---|
| [`dynamic-smart-offer`](features/dynamic-smart-offer/context.md) | RDESQ-1900 | expert-workspace, expert-workspace-api, expert-workspace-config |

## Workflows

Reusable process docs live under `workflows/`. Each is a markdown file describing a repeatable process — starting a new cross-repo feature, cutting a release, syncing context from Notion, etc.

## Usage with Claude Code

This repo is designed to be used as a working context hub with [Claude Code](https://claude.ai/claude-code). Open it as your working directory when doing cross-repo work — `CLAUDE.md` and the `features/` docs are loaded automatically to give Claude full context on conventions, architecture, and current state.

Each `repos/*/CLAUDE.md` symlink points to the live file in its source repo, so Claude always has current, repo-specific conventions without duplication.

## Maintaining This Repo

This repo is a living document. Keep it updated as work progresses:

- Update `features/*/context.md` trackers as items are completed
- Add new feature directories when starting cross-repo work
- Add workflow docs to `workflows/` as processes are established
- Update repo `CLAUDE.md` files (via the source repos) when conventions change
