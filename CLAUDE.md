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

Cross-repo features live under `features/`. Each directory follows the [OpenSpec](https://dev.to/webdeveloperhyper/how-to-make-ai-follow-your-instructions-more-for-free-openspec-2c85)-inspired structure: `proposal.md`, `tasks.md`, `design.md`, `specs/` (delta specs with WHEN/THEN scenarios), and `archive/` (historical docs). See `workflows/new-feature.md` for the full lifecycle.

| Feature | Jira | Repos involved |
|---|---|---|
| `features/dynamic-smart-offer` | RDESQ-1900 | expert-workspace, expert-workspace-api, expert-workspace-config, eds-platform-connectors, enterprise-eventing |

## Workflows

Reusable workflows live under `workflows/`. Each is a markdown file describing a repeatable process.

| Workflow | Description |
|---|---|
| `workflows/new-feature.md` | Start a new cross-repo feature — directory structure, lifecycle (propose → spec → implement → archive) |
| `workflows/feature-specs.md` | OpenSpec-style delta spec format — ADDED/MODIFIED/REMOVED requirements with WHEN/THEN scenarios |

## Maintaining This Repo

This repo is a living document. As we work, keep it updated:
- Document new patterns, conventions, or lessons learned
- Add new workflow files to `workflows/` as processes are established
- Add feature docs to `features/` when starting feature work
- Update repo CLAUDE.md files (via the source repos) when conventions change
- If something was hard to figure out, write it down so we don't repeat the effort

**Keep both `CLAUDE.md` and `README.md` in sync.** They describe the same repo — `CLAUDE.md` is Claude's working reference, `README.md` is the human-facing entry point on GitHub. When structure, features, workflows, or integrations change, update both. If you add a feature to the features table, add it to both. If you add a workflow, add it to both.

## Integrations (planned)

- **Notion** — sync context/PRDs into features/
- **Jira** — link features to tickets, track status
- **Slack** — workflow triggers, notifications
- **GitHub** — PR/issue automation across repos
