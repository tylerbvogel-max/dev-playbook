# dev-playbook

Standardized patterns, infrastructure, and processes for building mobile-first applications. Use this as the single source of truth when bootstrapping new apps.

## How to Use

1. **Starting a new project?** Follow the [Bootstrap Checklist](PLAYBOOK.md#checklist-new-project-bootstrap) in the playbook.
2. **Working with Claude Code?** Point it to this repo: _"Follow the rules and protocols in dev-playbook so we can get to the specific development aspects."_
3. **Need a specific pattern?** Check the [references/](references/) directory.

## Documents

| File | Purpose |
|------|---------|
| [PLAYBOOK.md](PLAYBOOK.md) | The main guide — stack, structure, architecture, patterns, bootstrap checklist |
| [references/backend-patterns.md](references/backend-patterns.md) | FastAPI models, routers, services, schemas, migrations, error handling |
| [references/mobile-patterns.md](references/mobile-patterns.md) | Expo setup, navigation, API client, React Query hooks, contexts, screens |
| [references/deployment.md](references/deployment.md) | Render deployment, EAS mobile builds, environment strategy |
| [references/dev-environment.md](references/dev-environment.md) | Local setup, Chromebook-specific config, daily workflow |
| [references/ai-workflow.md](references/ai-workflow.md) | CLAUDE.md template, permission guardrails, prompting patterns |

## Stack

- **Backend**: Python + FastAPI + PostgreSQL (async SQLAlchemy) + Alembic + APScheduler
- **Mobile**: React Native (Expo) + TypeScript + React Query + React Navigation
- **Deployment**: Render (Docker) + EAS (mobile builds)
- **AI Tooling**: Claude Code with permission guardrails

## Reference Implementation

[fantasy-trading](../fantasy-trading/) — a full-stack competitive paper trading app built with this exact stack and these exact patterns.
