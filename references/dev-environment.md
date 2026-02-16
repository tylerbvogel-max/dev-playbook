# Development Environment Reference

Setup instructions for local development, with Chromebook/Linux-specific guidance.

---

## System Requirements

| Tool | Version | Purpose |
|------|---------|---------|
| Python | 3.11+ | Backend runtime |
| Node.js | 20 (pinned in .nvmrc) | Mobile tooling, Expo CLI |
| PostgreSQL | 14+ | Database |
| Git | 2.x | Version control |
| Claude Code | Latest | AI-assisted development |

---

## Quick Setup (Any Linux / macOS / WSL)

### 1. Python

```bash
# Ubuntu/Debian
sudo apt update && sudo apt install python3 python3-venv python3-pip

# macOS
brew install python3

# Verify
python3 --version  # 3.11+
```

### 2. Node.js (via nvm)

```bash
# Install nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc

# Install and use pinned version
nvm install 20
nvm use 20

# Verify
node --version  # v20.x.x
```

### 3. PostgreSQL

```bash
# Ubuntu/Debian
sudo apt install postgresql postgresql-contrib

# Start service
sudo service postgresql start

# Create database and user
sudo -u postgres createdb app_name
sudo -u postgres createuser app_user --superuser

# Verify
psql -U app_user -d app_name -c "SELECT 1;"
```

### 4. Claude Code

```bash
npm install -g @anthropic-ai/claude-code
claude  # First run prompts for API key
```

---

## Project Setup

### Backend

```bash
cd backend

# Create virtual environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Configure environment
cp .env.example .env
# Edit .env with your values:
#   DATABASE_URL=postgresql+asyncpg://app_user:@localhost/app_name
#   SECRET_KEY=any-random-string-for-local-dev
#   EXTERNAL_API_KEY=your-key-here

# Seed initial data
python -m app.seed

# Start server
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# Verify
curl http://localhost:8000/health
```

### Mobile

```bash
cd mobile

# Install dependencies
npm install

# Start Expo dev server
npx expo start

# Scan QR code with Expo Go app on your phone
# Or press 'i' for iOS simulator / 'a' for Android emulator
```

### Connecting Mobile to Local Backend

The mobile app needs to reach your local backend. Options:

**Option A: Local IP (recommended)**
1. Find your local IP: `hostname -I | awk '{print $1}'`
2. Update `app.json`: `"apiUrl": "http://192.168.1.42:8000"`
3. Restart Expo

**Option B: Expo tunnel mode**
```bash
npx expo start --tunnel
```
This creates a public URL but is slower. Use when local IP doesn't work (corporate networks, VPNs).

---

## Chromebook-Specific Setup

### Enable Linux (Crostini)

1. Settings → Advanced → Developers → Linux development environment → Turn on
2. Wait for Linux container to install
3. Open Terminal app

### PostgreSQL Auto-Start

PostgreSQL doesn't auto-start on Chromebook Linux. Add to `.bashrc`:

```bash
echo 'sudo service postgresql start 2>/dev/null' >> ~/.bashrc
```

Or start manually each session:

```bash
sudo service postgresql start
```

### File Access

- Linux files are accessible from Chrome OS Files app under "Linux files"
- Drag and drop files between Chrome OS and Linux
- Linux home directory: `/home/your-username/`

### Port Forwarding

Chrome OS automatically forwards ports from the Linux container. `localhost:8000` in Chrome browser reaches the Linux backend.

### Performance Tips

- Use `--host 0.0.0.0` when running uvicorn (required for cross-container access)
- Expo tunnel mode may be needed if QR scanning doesn't connect
- Close unused Chrome tabs — Crostini shares RAM with Chrome OS

---

## Daily Workflow

### Terminal Layout (3 tabs)

```
Tab 1: Backend
  cd ~/Projects/app-name/backend
  source venv/bin/activate
  uvicorn app.main:app --reload --host 0.0.0.0

Tab 2: Mobile
  cd ~/Projects/app-name/mobile
  npx expo start

Tab 3: Claude Code / Git
  cd ~/Projects/app-name
  claude
```

### Common Commands

```bash
# Backend
source venv/bin/activate          # Activate Python env
uvicorn app.main:app --reload     # Start with auto-reload
python -m app.seed                # Re-seed database
pip install package-name          # Add dependency
pip freeze > requirements.txt     # Update requirements

# Mobile
npx expo start                    # Start dev server
npx expo start --clear            # Clear cache and start
npm install package-name          # Add dependency
npx expo install expo-package     # Add Expo-compatible dependency

# Database
sudo service postgresql start     # Start PostgreSQL
psql -U app_user -d app_name      # Connect to database
alembic revision --autogenerate -m "description"  # Generate migration
alembic upgrade head              # Apply migrations

# Git
git status
git add -p                        # Interactive staging
git commit -m "message"
git push
```

### Troubleshooting

| Problem | Solution |
|---------|----------|
| `psql: could not connect to server` | Run `sudo service postgresql start` |
| Expo QR code doesn't connect | Try `npx expo start --tunnel` |
| `ModuleNotFoundError` in Python | Activate venv: `source venv/bin/activate` |
| `ENOSPC: no space left on device` | Clear Expo cache: `npx expo start --clear` |
| Mobile can't reach backend | Check API URL matches your local IP + port |
| Permission denied on port 8000 | Kill existing process: `lsof -ti :8000 \| xargs kill` |

---

## Version Pinning

### .nvmrc

```
20
```

Ensures all developers use the same Node version. Run `nvm use` when entering the project.

### requirements.txt

Pin exact versions:

```
fastapi==0.115.0
sqlalchemy[asyncio]==2.0.35
```

Not ranges (`>=0.115.0`). Prevents surprise breaking changes.

### package.json

Use caret ranges (npm default):

```json
"@tanstack/react-query": "^5.59.0"
```

Allows minor/patch updates. Expo manages its own dependency versions via `npx expo install`.
