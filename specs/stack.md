# Tech Stack — Authaver

Diese Datei definiert den verbindlichen Tech-Stack. Keine Abweichungen ohne explizite Änderung dieser Datei.

---

## Runtime & Framework

| Schicht | Technologie | Version | Hinweis |
|---|---|---|---|
| Runtime | Node.js | 20 LTS | |
| Framework | Next.js | 15.x | App Router, Server Components, Server Actions |
| Sprache | TypeScript | 5.x | Strict Mode aktiviert |
| Styling | Tailwind CSS | 3.x | shadcn/ui Komponenten |

## Datenbank

| Technologie | Version | Hinweis |
|---|---|---|
| PostgreSQL | 16 | Primäre Datenbank |
| Prisma ORM | 5.x | Kein Raw SQL außer für FK-Indizes |
| Redis | 7 | Job Queue State (BullMQ) |

## Authentifizierung

- **Lucia Auth** v3 mit `@lucia-auth/adapter-prisma`
- Kein NextAuth, kein Clerk, kein Auth0
- Sessions in PostgreSQL-Tabelle (`session`)

## KI / LLM

- **Interface:** `lib/ai/index.ts` — `AIProvider` Interface
- **Provider A:** `@anthropic-ai/sdk` (claude-3-5-sonnet-20241022) — Default
- **Provider B:** `openai` (gpt-4o-mini) — Alternative
- **Provider C:** `ollama` (lokal, kein Cloud) — Privacy-Option
- **Auswahl:** via `AI_PROVIDER` Environment-Variable (`anthropic` | `openai` | `ollama`)
- **Strukturierte Outputs:** Zod-Schemas — kein freies JSON-Parsing

## Job Queue & Scheduling

- **BullMQ** v5 mit Redis
- Worker-Prozesse in `workers/` (separate Node-Prozesse)
- Cron-Scheduling via BullMQ Repeatable Jobs

## E-Mail

- **IMAP-Empfang:** `imapflow` v1
- **SMTP-Versand:** `nodemailer` (konfigurierbar: SMTP-Host via env)

## Dokumentenverarbeitung

- **PDF-Textextraktion:** `pdf-parse` v1
- **OCR (Bilder):** `tesseract.js` v5
- **PDF-Generierung:** `@react-pdf/renderer` v3

## Deployment

- **Container:** Docker + Docker Compose
- **Reverse Proxy:** Traefik (in docker-compose.prod.yml)
- **Datei-Storage:** Lokales Dateisystem via Docker Volume (kein S3 in v1)

## Wichtige Library-Entscheidungen

- Alle monetären Werte: **Integer (Cent)** in DB, Formatierung nur in UI
- Datumslogik: **date-fns** — kein moment.js, kein dayjs
- Charts: **recharts** — kein Chart.js
- Icons: **lucide-react** — kein heroicons
- Formulare: **react-hook-form** + **zod** für Validierung
- HTTP-Requests (Server → extern): **native fetch** — kein axios
