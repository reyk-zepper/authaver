# Authaver — Open-Source AI Property Management

> **Self-hosted. AI-powered. Zero monthly fees.**
>
> Authaver is an open-source property management platform with a team of AI agents working in the background. They process your inbox, verify invoices, monitor rent payments, and archive documents — automatically. You only decide on exceptions.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Status: Early Alpha](https://img.shields.io/badge/Status-Early%20Alpha-orange.svg)]()
[![Stack: Next.js + TypeScript](https://img.shields.io/badge/Stack-Next.js%20%2B%20TypeScript-black.svg)]()
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)]()

[🇩🇪 Deutsche Version](README.de.md) · [Live Demo](mockup.html) · [Architecture Docs](docs/architecture/)

---

## What is Authaver?

Managing rental properties is full of repetitive, time-consuming work: sorting mail, checking invoices, chasing rent, preparing utility billing. Most landlords either spend hours every week on administration or pay professional property managers €30–50 per unit per month.

Authaver takes a different approach: **a team of specialized AI agents handles the routine, you stay in control of decisions.**

The system is inspired by the [Paperclip agent architecture](https://paperclip.ing) — each agent has a defined role, a monthly token budget, a heartbeat schedule, and writes every action to an immutable audit log. You're the board. You can pause, override, or adjust any agent at any time.

---

## Core Features

### Dashboard
Know what needs attention in under 2 minutes. The AI prepares a prioritized list every morning — urgent issues at the top, low-priority items at the bottom.

### AI Inbox (Paula)
Every incoming email is automatically classified: invoice, repair request, rent query, legal notice, confirmation. Paula routes it to the right place and prepares a suggested response.

### Invoice Verification (Rex)
Every invoice is matched against open tickets. Rex flags mismatches, explains deviations from comparable invoices, and prepares a booking suggestion. You approve or reject in one click.

### Payment Monitoring (Zara)
Daily comparison of expected vs. actual rent payments. Zara detects arrears on day 1 and routes to Max for dunning draft preparation.

### Dunning (Max)
Max monitors payment history and drafts dunning letters according to German legal requirements — graded (1st notice → 2nd notice → legal notice). You review and send.

### Document Archive (Diana)
Every incoming document is processed, classified, and filed. Diana tracks which receipts are missing for utility billing and requests them proactively.

### Utility Billing
Annual Nebenkostenabrechnung preparation: collect all receipts, calculate per-unit allocation, generate drafts per tenant — all with legal German templates.

---

## AI Agent Architecture

Authaver uses a Paperclip-inspired agent architecture. Five specialized agents work in the background:

```
You (Board Level — full control)
  └── Paula (Orchestrator · every 15 min)
        ├── Rex (Invoice Specialist · on receipt)
        ├── Max (Dunning Specialist · daily 07:00)
        ├── Diana (Document Archivist · on receipt)
        └── Zara (Payment Monitor · daily 06:00)
```

**Key principles:**
- Each agent has a **monthly token budget** — prevents runaway costs
- Every action is **logged immutably** — full audit trail for legal compliance
- **Confidence thresholds**: below threshold → escalate to owner
- **Human override** always possible — pause, resume, or override any agent

See [docs/architecture/paperclip-patterns.md](docs/architecture/paperclip-patterns.md) for the full pattern documentation.

---

## Tech Stack

| Layer | Technology | Notes |
|---|---|---|
| Frontend | Next.js 15 (App Router) + TypeScript | Server Components, React |
| Styling | Tailwind CSS | shadcn/ui components |
| Database | PostgreSQL + Prisma | ACID transactions for financial data |
| Auth | Lucia Auth | Self-hosted, no third-party auth |
| AI Provider | Pluggable (Anthropic / OpenAI / Ollama) | No vendor lock-in |
| Email | IMAP (imapflow) | Direct mailbox integration |
| Documents | pdf-lib + Tesseract (OCR) | Invoice and contract processing |
| Job Queue | BullMQ + Redis | Agent heartbeats and scheduling |
| Deployment | Docker Compose | Single-command self-hosting |

**AI Provider strategy:** Authaver supports multiple LLM providers through a simple abstraction layer. Use Anthropic Claude or OpenAI for cloud convenience, or run Ollama locally for full data privacy — important for German DSGVO compliance.

---

## Self-Hosting

Authaver is designed to run on a standard VPS (Hetzner, Netcup, or any Linux server). Requirements: 2 vCPU, 4 GB RAM, 20 GB storage.

```bash
# Clone and start
git clone https://github.com/reyk-zepper/authaver
cd authaver
cp .env.example .env   # Add your AI provider key
docker compose up -d

# Open in browser
open http://localhost:3000
```

**Full self-hosting documentation:** Coming with v0.1 release.

---

## Supported Operation Modes

| Mode | Description |
|---|---|
| **Self-Service** | You manage everything yourself. Authaver handles all routine tasks automatically. |
| **Control Layer** | You have an external property manager. Authaver gives you full transparency and oversight. |
| **Managed Setup** | A service provider installs and maintains Authaver for you. You only see what matters. |

---

## Monetization Model

Authaver is MIT-licensed and free to self-host. The software itself costs nothing.

**Professional setup service** (available from us and certified partners):
- Server provisioning and Docker deployment
- Data migration from Excel, CSV, or existing property management software
- Configuration for your specific German federal state (legal templates, tax settings)
- DATEV / lexoffice / sevDesk export integration
- SEPA direct debit configuration
- Initial property and tenant data setup
- 2-hour onboarding session
- Optional: annual support contract

This is the [Red Hat model](https://en.wikipedia.org/wiki/Red_Hat): open software, paid expertise. The moat is knowledge and trust, not code.

---

## German Market Specifics

Authaver is built for the German market first:

- **DSGVO compliance** built-in (self-hosted = data stays on your server)
- **Legal document templates** for all 16 German federal states
- **Nebenkostenabrechnung** according to §556 BGB
- **Mietrecht** compliant dunning process (Mahnung → Abmahnung → Kündigung)
- **SEPA Lastschrift** support
- **DATEV export** for accounting integration
- **Grundsteuer** tracking

---

## Roadmap

**v0.1 — MVP (In Progress)**
- [ ] Property and tenant data management
- [ ] IMAP email ingestion + Paula classification
- [ ] Dashboard with AI priority list
- [ ] Ticket management with timeline
- [ ] Invoice logging and Rex matching
- [ ] Basic document archive
- [ ] Docker Compose deployment

**v0.2 — Core Agents**
- [ ] Max dunning workflow (graded)
- [ ] Zara payment monitoring + bank account sync
- [ ] Diana document gap detection
- [ ] Full audit log UI

**v0.3 — Utility Billing**
- [ ] Nebenkostenabrechnung workflow
- [ ] Tenant-level allocation
- [ ] PDF generation

**v1.0 — Production Ready**
- [ ] Multi-property support
- [ ] DATEV export
- [ ] SEPA integration
- [ ] Mobile-responsive UI
- [ ] Comprehensive test coverage

---

## Contributing

Authaver is open source and welcomes contributions. The codebase is structured as a modular Next.js monorepo — you can contribute to a single module without needing to understand the full system.

```bash
git clone https://github.com/reyk-zepper/authaver
cd authaver
npm install
npm run dev
```

**Good first issues:** [github.com/reyk-zepper/authaver/issues](https://github.com/reyk-zepper/authaver/issues)

Please read [CONTRIBUTING.md](CONTRIBUTING.md) before opening a PR.

---

## License

MIT — use it, fork it, build on it. If you run it as a service for others, you don't have to open-source your changes (MIT, not AGPL). But contributions back are always welcome.

---

## Why "Authaver"?

*Authaver* = *Authority* + *Haven*. Your property, your authority, your safe haven. The KI handles the noise. You keep control.

---

<p align="center">
  Built with ♥ · Open Source · MIT License<br>
  <a href="README.de.md">🇩🇪 Zur deutschen README</a>
</p>
