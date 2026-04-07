# Authaver — Open-Source AI Property Management

> **Self-hosted. AI-powered. Zero monthly fees.**
>
> Authaver is an open-source property management platform for private landlords, small portfolio holders, and homeowner associations (WEG). A team of AI agents works in the background — processing your inbox, verifying invoices, monitoring rent, archiving documents, and managing dunning. You only decide on exceptions.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Status: Early Alpha](https://img.shields.io/badge/Status-Early%20Alpha-orange.svg)]()
[![Stack: Next.js + TypeScript](https://img.shields.io/badge/Stack-Next.js%20%2B%20TypeScript-black.svg)]()
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)]()

[🇩🇪 Deutsche Version](README.de.md) · [Live Demo](mockup-v2.html) · [Architecture Docs](docs/architecture/)

---

## What is Authaver?

Managing rental properties and homeowner associations is full of repetitive, time-consuming work: sorting mail, checking invoices, chasing rent, filing documents, preparing utility billing, coordinating repairs. Most landlords either spend hours every week on administration or pay professional property managers €30–50 per unit per month.

Authaver takes a different approach: **a team of specialized AI agents handles the routine, you stay in control of decisions.**

Every transaction, document, and communication is tied to a shared operational core — property, unit, cost category, period, process, approval, document. This keeps utility billing, HOA finances, reserves, repairs, and measures cleanly traceable and auditable at all times.

---

## Who is Authaver for?

Authaver is designed to scale from 10 to 1,000 units, serving multiple user types:

| Persona | Goal |
|---|---|
| **Private self-manager** | Manage 1–20 units independently, cut costs, stay in control |
| **Portfolio holder / family office** | Standardize processes across multiple properties, delegate with oversight |
| **Control owner** | Monitor an external property manager — transparency and verification layer |
| **HOA coordinator** | Organize a small homeowner community, manage reserves and resolutions |
| **Certified internal manager** | Run full WEG administration with auditability |

Supporting roles (with limited, role-based access): co-owners, accounting staff, technical coordinators, tenants, contractors, external advisors.

---

## Core Operating Model

Authaver is built around a **unified transaction and process core**. Every expense, income, and document links to:

- **Property** (Liegenschaft)
- **Unit or cost center** (Einheit / Kostenstelle)
- **Cost category** (Kostenart)
- **Period** (Zeitraum)
- **Process** (Vorgang)
- **Approval** (Freigabe)
- **Document** (Beleg)

This foundation makes utility billing, HOA finances, reserves, repairs, and capital measures cleanly auditable — not just trackable.

### Eight Core Modules

| Module | Function |
|---|---|
| **Master Data** | Properties, buildings, units, owners, tenants, contractors, meters |
| **Process Core** | Tickets, tasks, approvals, deadlines, escalations, audit trail |
| **Documents** | Contracts, invoices, protocols, certificates, photos — all linked |
| **Finance** | Receivables, invoices, payments, accounts, reserves, dunning |
| **Billing** | Utility billing (Nebenkosten), cost categories, allocation keys, drafts |
| **Repairs** | Damage tickets, contractor assignment, history |
| **Maintenance** | Recurring obligations, intervals, inspection protocols |
| **Measures** | Capital projects, resolutions, budgets, milestone billing |

---

## AI Agent Architecture

Authaver uses a Paperclip-inspired agent architecture. Specialized agents work in the background, each with a defined role, monthly token budget, heartbeat schedule, and immutable audit log.

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
- **Confidence thresholds**: below threshold → escalate to owner for decision
- **Human override** always possible — pause, resume, or override any agent at any time
- **Agent Builder**: create additional specialized agents from templates (e.g. maintenance agent, reporting agent) — role-gated, with approval workflow

### How agents surface in the UI

- **KI-Center**: dedicated control room — agent status, token budgets, action history, configuration, agent builder
- **Contextual**: inline in every screen where an agent was active — Paula's classification in the inbox, Rex's review on an invoice, Max's dunning draft on an overdue payment

---

## Navigation Concept

Authaver uses a **two-layer navigation**:

**Global layer** (always visible):
```
Dashboard · Portfolio · Posteingang · Finanzen · KI-Center · Einstellungen
```

**Property layer** (when a property is open):
```
Übersicht · Vorgänge · Mieter · Finanzen · WEG · Dokumente
```

The **Dashboard** is role-adaptive — every user sees the same "What needs my attention today?" structure, but the content matches their role. A primary owner sees approvals and arrears. Accounting sees open invoices and payment entries. A technical coordinator sees repairs and overdue maintenance.

---

## Process: Mail → Inbox → Action

Every incoming email is processed by Paula:

1. Paula reads and classifies the mail (repair, invoice, inquiry, legal, document)
2. Paula proposes a typed process with confidence score and plain-language reasoning
3. The user confirms or corrects — and the process is created
4. Mail and process stay linked; the KI-Protokoll tab shows every agent action transparently

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

**AI Provider strategy:** Authaver supports multiple LLM providers through a pluggable abstraction layer. Use Anthropic Claude or OpenAI for cloud convenience, or Ollama locally for full data privacy — important for German DSGVO compliance.

---

## Self-Hosting

Authaver runs on a standard VPS (Hetzner, Netcup, or any Linux server). Requirements: 2 vCPU, 4 GB RAM, 20 GB storage.

```bash
git clone https://github.com/reyk-zepper/authaver
cd authaver
cp .env.example .env   # Add your AI provider key
docker compose up -d
open http://localhost:3000
```

---

## Roadmap

**Phase 0 — Foundation**
- [ ] Next.js boilerplate (App Router, TypeScript, Tailwind, Docker)
- [ ] Authentication + role model (5 MVP roles)
- [ ] Core database schema (unified process/transaction core)

**Phase 1 — Core UI**
- [ ] Role-adaptive global dashboard
- [ ] Property setup + property cockpit
- [ ] Units, tenants, leases

**Phase 2 — Inbox & Processes**
- [ ] Inbox with Paula classification (3-column UI)
- [ ] Adaptive process core (all types)
- [ ] Repair workflow (report → assign → close)

**Phase 3 — Finance Basics**
- [ ] Invoice intake + Rex verification
- [ ] Payment monitoring + Zara
- [ ] Dunning workflow + Max

**Phase 4 — WEG & Billing**
- [ ] WEG module (Hausgeld, reserves)
- [ ] Utility billing preparation (Nebenkosten)
- [ ] Document archive + Diana

**Later**
- Agent Builder (template + form config)
- Maintenance module
- Capital measures / Modernisierung module
- DATEV export
- Tenant self-service portal
- Contractor portal
- SEPA integration
- Mobile-responsive UI

---

## Supported Operation Modes

| Mode | Description |
|---|---|
| **Self-Service** | You manage everything yourself. Authaver handles all routine tasks automatically. |
| **Control Layer** | You have an external property manager. Authaver gives you full transparency and oversight. |
| **Managed Setup** | A service provider installs and maintains Authaver for you. You only see what matters. |

---

## Monetization Model

Authaver is MIT-licensed and free to self-host.

**Professional setup service** (from us and certified partners):
- Server provisioning and Docker deployment
- Data migration from Excel, CSV, or existing property management software
- Configuration for your German federal state (legal templates, tax settings)
- DATEV / lexoffice / sevDesk export integration
- SEPA direct debit configuration
- Initial property and tenant data setup
- 2-hour onboarding session
- Optional: annual support contract

This is the [Red Hat model](https://en.wikipedia.org/wiki/Red_Hat): open software, paid expertise.

---

## German Market Specifics

Authaver is built for the German market first:

- **DSGVO compliance** built-in (self-hosted = data stays on your server)
- **Legal document templates** for all 16 German federal states
- **Nebenkostenabrechnung** according to §556 BGB and BetrKV
- **Mietrecht** compliant dunning process (Mahnung → Abmahnung → Kündigung)
- **WEG-Verwaltung** according to §28 WEG (Wirtschaftsplan, Hausgeld, Jahresabrechnung)
- **SEPA Lastschrift** support
- **DATEV export** for accounting integration
- **Heizkostenverordnung** compliant consumption billing

---

## Contributing

Authaver is open source and welcomes contributions. The codebase is structured as a modular Next.js monorepo — you can contribute to a single module without needing to understand the full system.

```bash
git clone https://github.com/reyk-zepper/authaver
cd authaver
npm install
npm run dev
```

Please read [CONTRIBUTING.md](CONTRIBUTING.md) before opening a PR.

---

## License

MIT — use it, fork it, build on it.

---

## Why "Authaver"?

*Authaver* = *Authority* + *Haven*. Your property, your authority, your safe haven. The AI handles the noise. You keep control.

---

<p align="center">
  Built with ♥ · Open Source · MIT License<br>
  <a href="README.de.md">🇩🇪 Zur deutschen README</a>
</p>
