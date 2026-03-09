# Agent Orchestrator

Homebase for cross-repo engineering work across the Expert Workspace platform. This is where context lives for features that span multiple repos, reusable workflows are documented, and integrations with tools like Notion, Jira, Slack, and GitHub will be built out over time.

## Structure

```
repos/        # Symlinked CLAUDE.md files for each repo — context when working cross-repo
features/     # Cross-repo feature context: Notion exports, engineering notes, trackers
workflows/    # Repeatable process docs (markdown)
```

## Repos

Each subdirectory under `repos/` has a `CLAUDE.md` symlink pointing to the live file in the actual repo. Read these for repo-specific conventions before working in that codebase.

| Dir | Repo path | CLAUDE.md |
|---|---|---|
| `repos/expert-workspace` | `~/code/expert-workspace` | symlinked |
| `repos/expert-workspace-api` | `~/code/expert-workspace-api` | symlinked |
| `repos/expert-workspace-config` | `~/code/expert-workspace-config` | none yet |

### expert-workspace

React 19 SPA monorepo (pnpm + Turborepo). Expert-facing UI platform. Primary app is `expert-ui` — Vite, Mantine 8, Zustand 5, react-router 7, TanStack Query 5.

### expert-workspace-api

Node.js monorepo for Expert Workspace and Expert Profile APIs. AWS Lambda + Fastify behind API Gateway, DynamoDB, PostgreSQL (Drizzle), Redis. Also builds Docker images for K8s.

### expert-workspace-config

AWS CDK infrastructure config repo. No CLAUDE.md yet — add one when conventions are established.

## Features

Cross-repo features live under `features/`. Each directory contains context docs, Notion exports, engineering notes, open questions, and implementation trackers for a feature that touches multiple repos.

| Feature | Jira | Repos involved |
|---|---|---|
| `features/dynamic-smart-offer` | RDESQ-1900 | expert-workspace, expert-workspace-api, expert-workspace-config |

## Workflows

Reusable workflows live under `workflows/`. Each is a markdown file describing a repeatable process — e.g. starting a new cross-repo feature, cutting a release, syncing context from Notion, triaging Jira, etc.

## Integrations (planned)

- **Notion** — sync context/PRDs into features/
- **Jira** — link features to tickets, track status
- **Slack** — workflow triggers, notifications
- **GitHub** — PR/issue automation across repos
