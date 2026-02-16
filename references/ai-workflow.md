# AI-Assisted Development Workflow

How to effectively use Claude Code as a development partner across projects.

---

## CLAUDE.md — Project Context File

Every project gets a `CLAUDE.md` at the root. Claude Code reads this automatically when starting a session. It's the AI's first impression of your project.

### Template

```markdown
# Project Name

One-sentence description of what this app does.

## Quick Start

- Backend: `cd backend && source venv/bin/activate && uvicorn app.main:app --reload`
- Mobile: `cd mobile && npx expo start`
- Seed DB: `cd backend && python -m app.seed`
- Test API: `curl http://localhost:8000/health`

## Current State

- [x] Backend API with auth, CRUD for core entities
- [x] Mobile app with auth flow, list/detail screens
- [ ] Background jobs for daily processing
- [ ] Push notifications

## Architecture

- Backend: FastAPI + PostgreSQL (async SQLAlchemy)
- Mobile: Expo (React Native) + React Query
- Deployment: Render (Docker)
- See PLAYBOOK.md in dev-playbook repo for full patterns

## Key Files

| File | Purpose |
|------|---------|
| backend/app/main.py | FastAPI app, router registration |
| backend/app/models/*.py | Database models |
| backend/app/services/*.py | Business logic |
| backend/app/routers/*.py | API endpoints |
| mobile/App.tsx | React Native root |
| mobile/src/api/client.ts | Typed API client |
| mobile/src/hooks/useApi.ts | React Query hooks |

## Design Decisions

- Token auth (not JWT) — simplicity for invite-only beta
- UUID primary keys — prevents enumeration
- Transactions as source of truth — audit trail for all state changes
- React Query for server state, Context for app state

## Commands

| Command | Purpose |
|---------|---------|
| `alembic revision --autogenerate -m "msg"` | Generate migration |
| `alembic upgrade head` | Apply migrations |
| `python -m app.seed` | Seed initial data |
| `npx expo start --clear` | Start mobile with cache clear |
```

### Rules

- Keep it under 100 lines — Claude reads the whole thing every session
- Focus on what's unique to this project, not general patterns
- Update it as the project evolves (especially "Current State")
- Point to `dev-playbook` for general patterns instead of duplicating

---

## Permission Guardrails

### .claude/settings.local.json

Controls which commands Claude Code can run without asking for confirmation.

```json
{
  "permissions": {
    "allow": [
      "Bash(curl:*)",
      "Bash(node -e:*)",
      "Bash(npm install:*)",
      "Bash(npx expo start:*)",
      "Bash(uvicorn:*)",
      "Bash(python3:*)",
      "Bash(python -m app.seed:*)",
      "Bash(pip install:*)",
      "Bash(pip freeze:*)",
      "Bash(git add:*)",
      "Bash(git commit:*)",
      "Bash(git status:*)",
      "Bash(git diff:*)",
      "Bash(git log:*)",
      "Bash(alembic:*)",
      "Bash(psql:*)",
      "WebFetch(domain:your-api.onrender.com)"
    ]
  }
}
```

### What to Allow

- **Read-only operations**: git status, git log, git diff, curl, psql queries
- **Development tools**: uvicorn, expo start, npm install, pip install
- **Safe write operations**: git add, git commit (not push — confirm manually)
- **Project-specific**: seed scripts, migration commands

### What NOT to Allow

- `git push` — always confirm before pushing
- `git reset --hard` — destructive
- `rm -rf` — destructive
- `docker` commands that affect production
- Any command that modifies shared infrastructure

---

## Effective Prompting Patterns

### Starting a New Feature

```
I want to add [feature]. Here's what it should do:
- [requirement 1]
- [requirement 2]

Follow the patterns in dev-playbook for the backend service/router/model structure.
Start with the backend, then mobile.
```

### Bug Investigation

```
[Describe the bug behavior]

Check the relevant service and router. The issue is likely in [area].
Don't change anything yet — just investigate and tell me what you find.
```

### Referencing the Playbook

```
Add a new model for [entity] following the patterns in dev-playbook
(UUID primary key, timestamps, soft delete, unique constraints).
```

```
Set up the API client functions for the new endpoints
following the client.ts pattern from dev-playbook.
```

### Code Review

```
Review the changes I've made. Check for:
- Missing error handling
- Security issues
- Patterns that deviate from our playbook
- Missing React Query cache invalidation
```

---

## Session Workflow

### Starting a Session

1. Claude Code reads `CLAUDE.md` automatically
2. State where you left off: "Last session we finished X. Today I want to work on Y."
3. If starting a new feature, ask Claude to plan before coding

### During Development

- **Let Claude explore first**: "Read the existing model/service before making changes"
- **Work in layers**: Backend model → service → router → mobile client → hook → screen
- **Test incrementally**: "Hit the endpoint with curl before building the mobile screen"
- **Commit frequently**: After each working feature, commit

### Ending a Session

- Update `CLAUDE.md` "Current State" section
- Commit all changes
- Note any TODO items for next session

---

## Multi-Project Consistency

When working across multiple apps that follow this playbook:

1. **Same structure**: Every project has the same directory layout — no relearning
2. **Same patterns**: Services, routers, hooks, client all work the same way
3. **Same CLAUDE.md format**: Claude Code gets oriented quickly on any project
4. **Reference the playbook**: "Follow dev-playbook patterns" is a valid instruction

This consistency means:
- Context switching between projects is fast
- Claude Code produces consistent output across projects
- New collaborators (human or AI) onboard quickly
- Bugs follow predictable patterns → faster debugging
