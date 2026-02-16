# Dev Playbook

A standardized set of patterns, infrastructure decisions, and processes for building mobile-first applications. Follow this document to bootstrap new apps with a proven architecture and get to feature development faster.

**Reference implementation:** [fantasy-trading](../fantasy-trading/) — a full-stack competitive paper trading app built with this exact stack.

---

## Table of Contents

1. [Stack Overview](#stack-overview)
2. [Project Structure](#project-structure)
3. [Backend Architecture](#backend-architecture)
4. [Mobile Architecture](#mobile-architecture)
5. [Authentication](#authentication)
6. [Database Patterns](#database-patterns)
7. [API Design](#api-design)
8. [Background Jobs](#background-jobs)
9. [Deployment](#deployment)
10. [Development Environment](#development-environment)
11. [AI-Assisted Development](#ai-assisted-development)
12. [Checklist: New Project Bootstrap](#checklist-new-project-bootstrap)

For deeper dives, see the [references/](references/) directory.

---

## Stack Overview

| Layer | Technology | Why |
|-------|-----------|-----|
| **Backend** | Python + FastAPI | Async-first, auto-generated OpenAPI docs, dependency injection, type-safe with Pydantic |
| **Database** | PostgreSQL + SQLAlchemy (async) | Robust relational DB, async driver (asyncpg), mature migration tooling (Alembic) |
| **Mobile** | React Native (Expo managed) | Cross-platform from single codebase, managed cloud builds (EAS), OTA updates, large ecosystem |
| **Server State** | React Query (@tanstack/react-query) | Caching, auto-refetch, mutation invalidation, deduplication — keeps mobile responsive |
| **App State** | React Context API | Lightweight, no extra dependencies, sufficient for auth/mode/preferences |
| **Token Storage** | expo-secure-store | Encrypted persistent storage on iOS (Keychain) and Android (Keystore) |
| **Deployment** | Render (Docker) | Infrastructure-as-code via render.yaml, free tier for prototyping, auto-deploy on push |
| **Mobile Builds** | EAS (Expo Application Services) | Cloud builds for iOS/Android, TestFlight/Play Store submission, no local Xcode/Gradle |
| **Migrations** | Alembic | Version-controlled schema changes, autogenerate from models, rollback support |
| **Background Jobs** | APScheduler | In-process async scheduler, cron triggers, no external service needed |
| **AI Dev Tool** | Claude Code CLI | AI pair programming with permission guardrails, project context via CLAUDE.md |

---

## Project Structure

Every project follows this layout:

```
project-name/
├── backend/
│   ├── app/
│   │   ├── main.py              # FastAPI entrypoint, lifespan, CORS, router registration
│   │   ├── config.py            # Settings via pydantic-settings (.env loading)
│   │   ├── database.py          # Async engine, session factory, get_db() dependency
│   │   ├── models/              # SQLAlchemy ORM models (one file per entity)
│   │   │   └── __init__.py      # Re-export all models for Alembic discovery
│   │   ├── routers/             # API endpoint handlers (one file per feature)
│   │   │   └── __init__.py
│   │   ├── services/            # Business logic layer (one file per domain)
│   │   │   └── __init__.py
│   │   ├── schemas/             # Pydantic request/response DTOs
│   │   │   └── __init__.py
│   │   ├── jobs/                # Background job definitions
│   │   │   ├── __init__.py
│   │   │   └── scheduler.py     # APScheduler setup + job registration
│   │   └── seed.py              # Initial data seeder (run once)
│   ├── alembic/
│   │   ├── env.py               # Async-aware Alembic environment
│   │   ├── script.py.mako       # Migration template
│   │   └── versions/            # Generated migration files
│   ├── alembic.ini              # Alembic config (points to database URL)
│   ├── requirements.txt         # Pinned Python dependencies
│   ├── Dockerfile               # Production container image
│   └── .env.example             # Template for required environment variables
├── mobile/
│   ├── App.tsx                  # Root component (providers, auth gate, navigation)
│   ├── src/
│   │   ├── api/
│   │   │   └── client.ts        # Typed API client, auth headers, 401 handling
│   │   ├── hooks/
│   │   │   └── useApi.ts        # React Query hooks wrapping API client
│   │   ├── screens/             # Screen components (one file per screen)
│   │   ├── components/          # Reusable UI components
│   │   ├── contexts/            # React Context providers (auth, preferences)
│   │   ├── navigation/          # Navigator configs (if extracted from App.tsx)
│   │   └── utils/
│   │       └── theme.ts         # Design tokens (colors, spacing, fonts, radii)
│   ├── package.json
│   ├── app.json                 # Expo config (API URL, bundle IDs, EAS project)
│   ├── tsconfig.json            # Strict TypeScript + path aliases
│   └── babel.config.js
├── docs/                        # Feature specs, design docs, decisions log
├── .claude/
│   └── settings.local.json      # Claude Code permission guardrails
├── CLAUDE.md                    # Project context for Claude Code (quick start, key files)
├── README.md                    # Setup instructions, API reference, deployment guide
├── render.yaml                  # Infrastructure-as-code for Render
├── .gitignore
└── .nvmrc                       # Pinned Node.js version
```

### Conventions

- **One file per entity** in models/, routers/, services/ — named after the domain concept
- **`__init__.py`** in models/ re-exports all models so Alembic can discover them
- **Routers** handle HTTP concerns only (parsing requests, returning responses)
- **Services** contain all business logic (testable without HTTP layer)
- **Schemas** define Pydantic models for request validation and response serialization
- **Theme** is the single source of truth for all visual design tokens

---

## Backend Architecture

### FastAPI Application Setup

```python
# app/main.py
from contextlib import asynccontextmanager
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

@asynccontextmanager
async def lifespan(app: FastAPI):
    # STARTUP: create tables, run migrations, start scheduler
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    scheduler.start()
    yield
    # SHUTDOWN: stop scheduler
    scheduler.shutdown()

app = FastAPI(title="App Name", lifespan=lifespan)

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],        # Permissive for mobile clients
    allow_methods=["*"],
    allow_headers=["*"],
)

# Register routers
app.include_router(auth.router, prefix="/auth", tags=["auth"])
app.include_router(items.router, prefix="/items", tags=["items"])
# ...

@app.get("/health")
async def health():
    return {"status": "ok"}
```

**Key decisions:**
- Lifespan context manager for startup/shutdown (not deprecated `on_event`)
- Health endpoint at `/health` for load balancers and keep-alive pings
- CORS allows all origins — mobile apps make cross-origin requests by nature
- Each router gets a prefix and tag (auto-groups in OpenAPI docs)

### Configuration

```python
# app/config.py
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    database_url: str
    secret_key: str
    # Add app-specific settings here

    class Config:
        env_file = ".env"

settings = Settings()

# Fix Render's postgres:// → postgresql+asyncpg://
if settings.database_url.startswith("postgres://"):
    settings.database_url = settings.database_url.replace(
        "postgres://", "postgresql+asyncpg://", 1
    )
```

**Rule:** All secrets and environment-specific values come from `.env` (local) or environment variables (production). Never hardcode secrets.

### Database Setup

```python
# app/database.py
from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker, AsyncSession
from sqlalchemy.orm import DeclarativeBase

engine = create_async_engine(
    settings.database_url,
    pool_size=10,
    max_overflow=20,
)

async_session = async_sessionmaker(engine, expire_on_commit=False)

class Base(DeclarativeBase):
    pass

async def get_db() -> AsyncSession:
    async with async_session() as session:
        try:
            yield session
        finally:
            await session.close()
```

**Key decisions:**
- `expire_on_commit=False` prevents lazy-loading issues in async context
- Connection pooling (10 persistent + 20 overflow) handles concurrent requests
- `get_db()` is a FastAPI dependency — inject into any endpoint

### Service Layer Pattern

```
Router (HTTP) → Service (Logic) → Model (ORM) → Database
```

- **Router**: Parses request, calls service, returns response
- **Service**: Validates business rules, orchestrates operations, wraps in DB transaction
- **Model**: Defines schema, relationships, constraints

```python
# app/routers/items.py
@router.post("/")
async def create_item(req: CreateItemRequest, db=Depends(get_db), user=Depends(get_current_user)):
    return await item_service.create_item(db, user.id, req)

# app/services/item_service.py
async def create_item(db: AsyncSession, user_id: UUID, req: CreateItemRequest) -> Item:
    item = Item(name=req.name, owner_id=user_id)
    db.add(item)
    await db.commit()
    return item
```

See [references/backend-patterns.md](references/backend-patterns.md) for detailed examples.

---

## Mobile Architecture

### App Root Structure

```tsx
// App.tsx
export default function App() {
  const [isAuthenticated, setIsAuthenticated] = useState(false);

  return (
    <QueryClientProvider client={queryClient}>
      <SafeAreaProvider>
        <NavigationContainer>
          {isAuthenticated ? <MainTabs /> : <AuthStack />}
        </NavigationContainer>
      </SafeAreaProvider>
    </QueryClientProvider>
  );
}
```

**Provider hierarchy (outside → inside):**
1. `QueryClientProvider` — React Query cache
2. `SafeAreaProvider` — safe area insets
3. `NavigationContainer` — React Navigation
4. Auth gate — conditional rendering based on auth state
5. Feature-specific contexts (inside authenticated area only)

### API Client

A single `client.ts` file that:
1. Stores auth token in memory (fast) and SecureStore (persistent)
2. Attaches `Authorization: Bearer <token>` to every request
3. Auto-signs out on 401 responses
4. Exports typed functions grouped by feature domain

```typescript
// src/api/client.ts
const API_BASE = Constants.expoConfig?.extra?.apiUrl ?? "http://localhost:8000";

let authToken: string | null = null;

async function request<T>(path: string, options?: RequestInit): Promise<T> {
  const headers: Record<string, string> = { "Content-Type": "application/json" };
  if (authToken) headers.Authorization = `Bearer ${authToken}`;

  const response = await fetch(`${API_BASE}${path}`, { ...headers, ...options });

  if (response.status === 401) {
    await signOut();
    throw new Error("Session expired");
  }

  if (!response.ok) throw new Error(await response.text());
  return response.json();
}

// Group by domain
export const auth = {
  register: (data) => request("/auth/register", { method: "POST", body: JSON.stringify(data) }),
  login: (data) => request("/auth/login", { method: "POST", body: JSON.stringify(data) }),
  me: () => request("/auth/me"),
};

export const items = {
  list: () => request("/items"),
  create: (data) => request("/items", { method: "POST", body: JSON.stringify(data) }),
};
```

### React Query Hooks

```typescript
// src/hooks/useApi.ts
export function useItems() {
  return useQuery({ queryKey: ["items"], queryFn: () => items.list() });
}

export function useCreateItem() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (data) => items.create(data),
    onSuccess: () => queryClient.invalidateQueries({ queryKey: ["items"] }),
  });
}
```

**Rules:**
- Every API endpoint gets a hook
- Query keys are descriptive arrays: `["items"]`, `["items", itemId]`, `["items", { filter }]`
- Mutations invalidate related queries on success
- Use `refetchInterval` for live-updating data (leaderboards, prices)

### Design Tokens

```typescript
// src/utils/theme.ts
export const colors = {
  background: "#000000",
  card: "#1A1A1A",
  surface: "#333333",
  text: "#FFFFFF",
  textSecondary: "#A0A0A0",
  primary: "#6172C5",
  accent: "#ED2EA5",
  success: "#38A169",
  error: "#E53E3E",
  warning: "#FA8057",
};

export const spacing = { xs: 4, sm: 8, md: 12, lg: 16, xl: 20, xxl: 24, xxxl: 32 };
export const fontSize = { xs: 11, sm: 13, md: 15, lg: 17, xl: 20, xxl: 24, hero: 40 };
export const radius = { sm: 6, md: 10, lg: 14, xl: 20, full: 999 };
```

**Rule:** Never use magic numbers for colors, spacing, or sizes. Always reference theme tokens. This makes rebranding a single-file change.

See [references/mobile-patterns.md](references/mobile-patterns.md) for navigation, font loading, and context patterns.

---

## Authentication

### Token-Based Auth (No Passwords)

This stack uses a simplified token auth model suitable for invite-only or early-stage apps:

1. **Registration**: User provides alias + invite code → server generates token → returned once
2. **Storage**: Token saved to `expo-secure-store` (encrypted on device)
3. **Requests**: Token sent as `Authorization: Bearer <token>` on every API call
4. **Backend**: Token is SHA-256 hashed before DB lookup (never store raw tokens)

```python
# Generate
raw_token = secrets.token_urlsafe(32)  # ~256 bits entropy

# Store (DB)
token_hash = hashlib.sha256(raw_token.encode()).hexdigest()

# Verify (on request)
incoming_hash = hashlib.sha256(incoming_token.encode()).hexdigest()
user = await db.execute(select(User).where(User.token_hash == incoming_hash))
```

### Invite Code System

Controls access during beta/early stages:

- Codes stored in `InviteCode` table with `max_uses` and `expires_at`
- Validated and consumed during registration
- Generate in bulk via admin endpoint or seed script
- Never commit codes to git

### Auth Dependency (FastAPI)

```python
async def get_current_user(
    authorization: str = Header(...),
    db: AsyncSession = Depends(get_db),
) -> User:
    token = authorization.replace("Bearer ", "")
    token_hash = hashlib.sha256(token.encode()).hexdigest()
    result = await db.execute(select(User).where(User.token_hash == token_hash))
    user = result.scalar_one_or_none()
    if not user:
        raise HTTPException(status_code=401, detail="Invalid token")
    return user
```

Use as a dependency: `user = Depends(get_current_user)`

### Upgrading to JWT

When you need refresh tokens, expiration, or multi-device management, upgrade to JWT:
- `python-jose[cryptography]` for encoding/decoding
- Access token (short-lived, 15 min) + Refresh token (long-lived, 90 days)
- Store refresh token in SecureStore, access token in memory
- Add `/auth/refresh` endpoint

---

## Database Patterns

### UUID Primary Keys

All user-facing tables use UUID primary keys:

```python
from uuid import uuid4
from sqlalchemy import UUID
from sqlalchemy.orm import mapped_column

class User(Base):
    __tablename__ = "users"
    id = mapped_column(UUID, primary_key=True, default=uuid4)
```

**Why:** Prevents enumeration attacks, safe for client-side generation, no sequence conflicts across environments.

### Transactions as Source of Truth

For any operation involving a sequence of changes (financial transactions, state transitions):

1. **Log the event** as an immutable record first
2. **Derive current state** from the event log
3. **Cache derived state** in denormalized tables for performance

```
Event happens → Create Transaction record → Update derived state (Holdings, Balances) → COMMIT atomically
```

If derived state ever corrupts, rebuild from the transaction log.

### Unique Constraints

Prevent duplicates at the database level, not just application level:

```python
class Holding(Base):
    __tablename__ = "holdings"
    __table_args__ = (
        UniqueConstraint("player_season_id", "stock_symbol", name="uq_holding_player_stock"),
    )
```

### Snapshot Pattern

For historical data that would be expensive to calculate on-the-fly:

- Capture snapshots on a schedule (e.g., daily at close of business)
- Store pre-computed values (totals, percentages, rankings)
- Use for charts, historical leaderboards, analytics
- Snapshots are immutable once written

### Relationship Conventions

```python
# Parent
class User(Base):
    items = relationship("Item", back_populates="owner")

# Child
class Item(Base):
    owner_id = mapped_column(ForeignKey("users.id"))
    owner = relationship("User", back_populates="items")
```

Always use `back_populates` (explicit) over `backref` (implicit).

---

## API Design

### URL Conventions

```
GET    /items              → List items
POST   /items              → Create item
GET    /items/{id}         → Get single item
PUT    /items/{id}         → Update item
DELETE /items/{id}         → Delete item
GET    /items/{id}/related → Get related resources
POST   /items/{id}/action  → Perform action on item
```

### Response Patterns

- **Success**: Return the resource directly (not wrapped in `{ data: ... }`)
- **List**: Return an array `[...]`
- **Error**: FastAPI default `{ "detail": "message" }` with appropriate HTTP status
- **Validation**: Use Pydantic schemas — FastAPI auto-returns 422 with field-level errors

### Rate Limiting

Apply to auth endpoints at minimum:

```python
from collections import defaultdict
from time import time

_rate_limit: dict[str, list[float]] = defaultdict(list)

def check_rate_limit(request: Request, max_per_minute: int = 10):
    ip = request.client.host
    now = time()
    _rate_limit[ip] = [t for t in _rate_limit[ip] if now - t < 60]
    if len(_rate_limit[ip]) >= max_per_minute:
        raise HTTPException(status_code=429, detail="Too many requests")
    _rate_limit[ip].append(now)
```

---

## Background Jobs

### APScheduler Setup

```python
# app/jobs/scheduler.py
from apscheduler.schedulers.asyncio import AsyncIOScheduler
from apscheduler.triggers.cron import CronTrigger

scheduler = AsyncIOScheduler()

async def job_example():
    async with async_session() as db:
        # Do work
        await db.commit()

def register_jobs():
    scheduler.add_job(job_example, CronTrigger(hour=16, minute=30, timezone="America/New_York"))
```

**Rules:**
- Each job creates its own DB session (not shared with request sessions)
- Jobs are idempotent — safe to re-run if they fail
- Log job execution (count of records processed, errors)
- Use timezone-aware triggers for business-hour operations

### Common Job Patterns

| Job | Schedule | Purpose |
|-----|----------|---------|
| Health/keep-alive ping | Every 14 min | Prevent free-tier sleep |
| Cache refresh | Every 15-60 min | Update cached external data |
| Daily snapshots | Once daily at close | Capture historical state |
| Cleanup/maintenance | Daily off-peak | Remove expired records, compact data |

---

## Deployment

### Render (render.yaml)

```yaml
databases:
  - name: app-db
    plan: free
    databaseName: app_name
    user: app_user

services:
  - type: web
    name: app-api
    runtime: docker
    plan: free
    dockerfilePath: ./backend/Dockerfile
    dockerContext: ./backend
    envVars:
      - key: DATABASE_URL
        fromDatabase: app-db.connectionString
      - key: SECRET_KEY
        generateValue: true
      - key: EXTERNAL_API_KEY
        sync: false  # Set manually in dashboard
```

### Dockerfile

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**Rules:**
- Use `python:3.12-slim` (not full image — saves 800+ MB)
- Copy requirements first (layer caching — only rebuild on dependency changes)
- `--no-cache-dir` reduces image size
- `--host 0.0.0.0` required for container networking

### Mobile Builds (EAS)

```bash
# Install EAS CLI
npm install -g eas-cli

# Configure (first time)
eas build:configure

# Build for testing
eas build --platform ios --profile preview    # → TestFlight
eas build --platform android --profile preview # → APK

# Build for production
eas build --platform all --profile production

# Submit to stores
eas submit --platform ios
eas submit --platform android
```

### Free Tier Considerations

Render free tier sleeps after 15 minutes of inactivity. Mitigate with:
1. External ping service (cron-job.org) hitting `/health` every 14 minutes
2. Accept ~30 second cold starts during off-peak hours
3. Upgrade to paid plan ($7/mo) when cold starts matter

See [references/deployment.md](references/deployment.md) for detailed deployment walkthrough.

---

## Development Environment

### Prerequisites

| Tool | Version | Install |
|------|---------|---------|
| Python | 3.11+ | `apt install python3` or pyenv |
| Node.js | 20 (pinned in .nvmrc) | `nvm install 20` |
| PostgreSQL | 14+ | `apt install postgresql` |
| Claude Code | Latest | `npm install -g @anthropic-ai/claude-code` |

### Local Development Workflow

**Three terminal tabs:**

```bash
# Tab 1: Backend
cd backend
source venv/bin/activate
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# Tab 2: Mobile
cd mobile
npx expo start

# Tab 3: Claude Code
claude
```

### First-Time Setup

```bash
# Backend
cd backend
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env  # Fill in values
python -m app.seed    # Seed initial data

# Mobile
cd mobile
npm install
# Update app.json extra.apiUrl to local IP if needed
```

### PostgreSQL Quick Start

```bash
# Start service (required after each boot on Chromebook/WSL)
sudo service postgresql start

# Create database
sudo -u postgres createdb app_name
sudo -u postgres createuser app_user --superuser

# Connection string for .env
DATABASE_URL=postgresql+asyncpg://app_user:@localhost/app_name
```

See [references/dev-environment.md](references/dev-environment.md) for Chromebook-specific setup.

---

## AI-Assisted Development

### CLAUDE.md

Every project gets a `CLAUDE.md` at the root. This is loaded automatically by Claude Code and provides project context:

```markdown
# Project Name

## Quick Start
- Backend: cd backend && source venv/bin/activate && uvicorn app.main:app --reload
- Mobile: cd mobile && npx expo start
- Seed DB: cd backend && python -m app.seed

## Current State
- [What's built, what's in progress]

## Key Files
| File | Purpose |
|------|---------|
| backend/app/main.py | FastAPI entrypoint |
| mobile/App.tsx | React Native root |
| ... | ... |

## Design Decisions
- [Important architectural choices and why]

## Commands Reference
- [Common dev commands]
```

### Permission Guardrails

Configure `.claude/settings.local.json` to whitelist safe commands:

```json
{
  "permissions": {
    "allow": [
      "Bash(curl:*)",
      "Bash(npm install:*)",
      "Bash(npx expo start:*)",
      "Bash(uvicorn:*)",
      "Bash(python3:*)",
      "Bash(git add:*)",
      "Bash(git commit:*)"
    ]
  }
}
```

**Rule:** Only whitelist commands the AI agent needs. Prevents accidental destructive operations.

See [references/ai-workflow.md](references/ai-workflow.md) for tips on effective AI-assisted development.

---

## Checklist: New Project Bootstrap

Use this checklist when starting a new app. Each step references the relevant section above.

### 1. Repository Setup
- [ ] Create repo with [project structure](#project-structure)
- [ ] Add `.gitignore` (Python, Node, IDE, secrets)
- [ ] Add `.nvmrc` with Node version `20`
- [ ] Write `README.md` with setup instructions
- [ ] Write `CLAUDE.md` with project context

### 2. Backend Scaffold
- [ ] Create `venv` and install base dependencies (see [requirements below](#base-requirements))
- [ ] Set up `config.py` with pydantic-settings
- [ ] Set up `database.py` with async engine + `get_db()`
- [ ] Set up `main.py` with lifespan, CORS, health endpoint
- [ ] Create first model (usually `User`)
- [ ] Create `auth_service.py` with token generation + hashing
- [ ] Create `auth` router with register/login/me
- [ ] Set up Alembic with async env.py
- [ ] Create `.env.example`
- [ ] Write `seed.py` for initial data

### 3. Mobile Scaffold
- [ ] `npx create-expo-app` with TypeScript template
- [ ] Install core dependencies (React Query, React Navigation, SecureStore)
- [ ] Set up `theme.ts` with design tokens
- [ ] Set up `client.ts` with typed API client + auth headers + 401 handling
- [ ] Set up `useApi.ts` with React Query hooks
- [ ] Create `App.tsx` with provider hierarchy + auth gate
- [ ] Create `AuthScreen` (register + login)
- [ ] Create first authenticated screen
- [ ] Configure `app.json` with API URL, bundle IDs

### 4. Deployment
- [ ] Write `Dockerfile` for backend
- [ ] Write `render.yaml` with database + web service
- [ ] Deploy to Render
- [ ] Set environment variables in Render dashboard
- [ ] Set up keep-alive ping (cron-job.org → `/health`)
- [ ] Update mobile `app.json` with production API URL

### 5. Development Workflow
- [ ] Set up `.claude/settings.local.json` with permissions
- [ ] Test full flow: register → login → create item → view item
- [ ] Set up background jobs if needed

### Base Requirements (backend/requirements.txt)

```
fastapi==0.115.0
uvicorn[standard]==0.30.6
sqlalchemy[asyncio]==2.0.35
asyncpg==0.29.0
alembic==1.13.2
pydantic==2.9.2
pydantic-settings==2.5.2
httpx==0.27.2
apscheduler==3.10.4
python-jose[cryptography]==3.3.0
```

### Base Mobile Dependencies

```bash
npx expo install expo-secure-store expo-font @expo-google-fonts/space-grotesk
npm install @tanstack/react-query @react-navigation/native @react-navigation/bottom-tabs @react-navigation/native-stack react-native-screens react-native-safe-area-context
```

---

## Principles

1. **Async everywhere.** All database I/O, HTTP calls, and background jobs use `async/await`. No blocking calls in the request path.

2. **Services own logic, routers own HTTP.** Never put business logic in a router. Never put HTTP concerns in a service.

3. **Transactions as source of truth.** Log events immutably, derive state from logs, cache for performance.

4. **Type safety end-to-end.** Pydantic on the backend, TypeScript (strict mode) on mobile. Catch errors at build time.

5. **Theme tokens, not magic numbers.** All visual values come from `theme.ts`. Rebranding is a single-file change.

6. **Secrets never in code.** All secrets in `.env` (local) or environment variables (production). `.env` is gitignored.

7. **Idempotent operations.** Seed scripts, migrations, and background jobs are safe to re-run.

8. **Mobile-first mindset.** The API serves a mobile client. Design for intermittent connectivity, small screens, and touch interactions.
