# Dev Commands — Authaver

Alle wichtigen Befehle für Entwicklung, Testing und Deployment.

---

## Entwicklung starten

```bash
# Abhängigkeiten installieren
npm install

# Datenbank starten (PostgreSQL + Redis via Docker)
docker compose up -d db redis

# Datenbank migrieren
npx prisma migrate dev

# Demo-Daten laden
npx prisma db seed

# Dev-Server starten
npm run dev
# → http://localhost:3000

# BullMQ Worker starten (separates Terminal)
npm run workers
```

## Vollständiges Docker-Setup (Produktion / Demo)

```bash
cp .env.example .env
# .env mit Werten befüllen (AI_PROVIDER, IMAP_*, SMTP_*, DATABASE_URL)

docker compose up -d
# → http://localhost:3000

# Logs ansehen
docker compose logs -f app

# Stoppen
docker compose down
```

## Datenbank

```bash
# Neue Migration erstellen
npx prisma migrate dev --name <migration-name>

# Migration in Produktion anwenden
npx prisma migrate deploy

# Prisma Studio (DB-Browser)
npx prisma studio
# → http://localhost:5555

# Schema neu generieren (nach schema.prisma Änderung)
npx prisma generate

# Seed-Daten laden
npx prisma db seed

# Datenbank zurücksetzen (ACHTUNG: löscht alle Daten)
npx prisma migrate reset
```

## Code-Qualität

```bash
# TypeScript prüfen (MUSS vor jedem Commit grün sein)
npm run typecheck

# Linting
npm run lint

# Linting mit Auto-Fix
npm run lint:fix

# Beides zusammen (CI-Check)
npm run check
```

## Testing

```bash
# Unit Tests
npm run test

# Tests mit Watch-Mode
npm run test:watch

# Test-Coverage
npm run test:coverage

# E2E Tests (Playwright)
npm run test:e2e

# Einzelnen Test ausführen
npm run test -- --grep "Paula"
```

## Nützliche Hilfsbefehle

```bash
# BullMQ Queue-Status ansehen (Development only)
open http://localhost:3000/admin/queues

# Agent manuell triggern (Development)
npm run agent:trigger paula
npm run agent:trigger rex
npm run agent:trigger zara

# Alle Agents einmal ausführen (Debug)
npm run agents:run-all

# E-Mail-Polling manuell triggern
npm run email:fetch

# NK-Berechnungen testen
npm run nk:preview <property-id> <year>
```

## Environment Variables (.env.example)

```bash
# Datenbank
DATABASE_URL="postgresql://authaver:password@localhost:5432/authaver"

# Redis (für BullMQ)
REDIS_URL="redis://localhost:6379"

# Auth
AUTH_SECRET="your-random-secret-min-32-chars"

# KI-Provider (anthropic | openai | ollama)
AI_PROVIDER="anthropic"
ANTHROPIC_API_KEY=""
OPENAI_API_KEY=""
OLLAMA_BASE_URL="http://localhost:11434"
OLLAMA_MODEL="llama3.2:3b"

# E-Mail IMAP (Posteingang)
IMAP_HOST=""
IMAP_PORT="993"
IMAP_USER=""
IMAP_PASS=""
IMAP_TLS="true"

# E-Mail SMTP (Versand)
SMTP_HOST=""
SMTP_PORT="587"
SMTP_USER=""
SMTP_PASS=""
SMTP_FROM="noreply@example.com"

# Uploads
UPLOAD_PATH="./uploads"
MAX_UPLOAD_MB="20"

# App
NEXT_PUBLIC_APP_URL="http://localhost:3000"
NODE_ENV="development"
```

## package.json Scripts Referenz

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "workers": "tsx watch workers/index.ts",
    "typecheck": "tsc --noEmit",
    "lint": "next lint",
    "lint:fix": "next lint --fix",
    "check": "npm run typecheck && npm run lint",
    "test": "vitest",
    "test:watch": "vitest --watch",
    "test:coverage": "vitest --coverage",
    "test:e2e": "playwright test",
    "agent:trigger": "tsx scripts/trigger-agent.ts",
    "agents:run-all": "tsx scripts/run-all-agents.ts",
    "email:fetch": "tsx scripts/fetch-emails.ts",
    "nk:preview": "tsx scripts/nk-preview.ts"
  }
}
```
