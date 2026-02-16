# Deployment Reference

Step-by-step deployment for backend (Render) and mobile (EAS).

---

## Backend: Render

### Prerequisites

- GitHub repo with backend code
- Render account (https://render.com)
- `render.yaml` at repo root
- `Dockerfile` in `backend/`

### render.yaml Template

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
    healthCheckPath: /health
    envVars:
      - key: DATABASE_URL
        fromDatabase: app-db.connectionString
      - key: SECRET_KEY
        generateValue: true
      - key: EXTERNAL_API_KEY
        sync: false
```

### Deployment Steps

1. **Push to GitHub**: Ensure `render.yaml` and `Dockerfile` are committed
2. **Connect Render**: Dashboard → New → Blueprint → Connect your repo
3. **Render reads `render.yaml`**: Auto-creates database + web service
4. **Set manual env vars**: Dashboard → Service → Environment → Add `EXTERNAL_API_KEY`
5. **First deploy**: Render builds Docker image, starts service
6. **Verify**: `curl https://your-app.onrender.com/health`
7. **Run seed**: SSH into service or trigger seed via admin endpoint

### Dockerfile Template

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Free Tier Notes

| Constraint | Detail |
|-----------|--------|
| Sleep after idle | 15 minutes → service sleeps, ~30s cold start on next request |
| Database | 256 MB storage, expires after 90 days |
| Bandwidth | 100 GB/month |
| CPU/RAM | Shared, limited |

### Keep-Alive Ping

Prevent free-tier sleep during active hours:

1. Go to https://cron-job.org
2. Create a job: `GET https://your-app.onrender.com/health`
3. Schedule: Every 14 minutes
4. Optional: Only during business hours (save free minutes)

### Upgrading

When cold starts matter:
- **Starter plan** ($7/month): Always-on, no sleep
- **Database**: Upgrade before 90-day expiration or when you need >256 MB
- **Custom domain**: Add in Render dashboard → Settings → Custom Domains

---

## Database: PostgreSQL on Render

### Connection String

Render auto-generates and injects `DATABASE_URL`. Format:

```
postgres://user:password@host:5432/database_name
```

The backend config converts this to async:

```
postgresql+asyncpg://user:password@host:5432/database_name
```

### Migrations in Production

Migrations run automatically at startup via the lifespan hook:

```python
async with engine.begin() as conn:
    await conn.run_sync(Base.metadata.create_all)
```

For Alembic-managed migrations, add to startup:

```python
from alembic.config import Config
from alembic import command

alembic_cfg = Config("alembic.ini")
command.upgrade(alembic_cfg, "head")
```

### Backup

Render free tier has no automatic backups. For critical data:

```bash
# Export from Render
pg_dump $DATABASE_URL > backup.sql

# Or use Render dashboard: Database → Backups → Create Backup
```

---

## Mobile: EAS (Expo Application Services)

### Prerequisites

- Expo account (https://expo.dev)
- EAS CLI: `npm install -g eas-cli`
- `app.json` with bundle identifier and EAS project ID
- Apple Developer account (for iOS) / Google Play account (for Android)

### First-Time Setup

```bash
# Login to Expo
eas login

# Link project (creates EAS project ID)
eas build:configure

# This updates app.json with your EAS project ID
```

### Build Profiles

Configure in `eas.json` (auto-created by `eas build:configure`):

```json
{
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal"
    },
    "preview": {
      "distribution": "internal",
      "ios": {
        "simulator": false
      }
    },
    "production": {
      "autoIncrement": true
    }
  }
}
```

### Build Commands

```bash
# Development build (includes dev tools)
eas build --platform ios --profile development

# Preview build (for testers via TestFlight / internal distribution)
eas build --platform ios --profile preview
eas build --platform android --profile preview

# Production build (for App Store / Play Store)
eas build --platform ios --profile production
eas build --platform android --profile production
```

### Submit to Stores

```bash
# iOS → App Store Connect → TestFlight
eas submit --platform ios

# Android → Google Play Console
eas submit --platform android
```

### OTA Updates (Over-the-Air)

For JS-only changes (no native module changes):

```bash
eas update --branch production --message "Fix typo in welcome screen"
```

Users get the update on next app launch without re-downloading from the store.

---

## Environment Strategy

| Environment | Backend | Mobile | Database |
|------------|---------|--------|----------|
| **Local** | `uvicorn --reload` on localhost:8000 | Expo Go on phone (QR scan) | Local PostgreSQL |
| **Preview** | Render free tier | EAS preview build (TestFlight) | Render free PostgreSQL |
| **Production** | Render paid tier | EAS production build (App Store) | Render paid PostgreSQL |

### Switching API URLs

The mobile app reads `apiUrl` from `app.json`:

```json
{
  "expo": {
    "extra": {
      "apiUrl": "https://your-app.onrender.com"
    }
  }
}
```

For local development, update to your machine's local IP:

```json
{
  "expo": {
    "extra": {
      "apiUrl": "http://192.168.1.42:8000"
    }
  }
}
```

**Do not commit local IP changes.** Use environment-specific `app.config.js` for dynamic config if needed:

```javascript
// app.config.js
export default {
  expo: {
    extra: {
      apiUrl: process.env.API_URL ?? "https://your-app.onrender.com",
    },
  },
};
```

Then: `API_URL=http://192.168.1.42:8000 npx expo start`

---

## CI/CD Considerations

### Auto-Deploy Backend

Render auto-deploys on push to the connected branch (usually `main`). No CI config needed.

### Mobile Builds

EAS builds run in Expo's cloud. Trigger manually or via GitHub Actions:

```yaml
# .github/workflows/eas-build.yml
name: EAS Build
on:
  push:
    branches: [main]
    paths: [mobile/**]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: expo/expo-github-action@v8
        with:
          eas-version: latest
          token: ${{ secrets.EXPO_TOKEN }}
      - run: cd mobile && npm install
      - run: cd mobile && eas build --platform all --profile preview --non-interactive
```
