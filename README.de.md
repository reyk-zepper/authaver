# Authaver — Open-Source KI-Hausverwaltung

> **Selbst hostbar. KI-gestützt. Keine monatlichen Gebühren.**
>
> Authaver ist eine open-source Hausverwaltungsplattform mit einem Team aus KI-Agenten, die im Hintergrund arbeiten. Sie verarbeiten deinen Posteingang, prüfen Rechnungen, überwachen Zahlungseingänge und archivieren Dokumente — automatisch. Du entscheidest nur noch bei Ausnahmen.

[![Lizenz: MIT](https://img.shields.io/badge/Lizenz-MIT-blue.svg)](LICENSE)
[![Status: Early Alpha](https://img.shields.io/badge/Status-Early%20Alpha-orange.svg)]()
[![Stack: Next.js + TypeScript](https://img.shields.io/badge/Stack-Next.js%20%2B%20TypeScript-black.svg)]()
[![PRs willkommen](https://img.shields.io/badge/PRs-willkommen-brightgreen.svg)]()

[🇬🇧 English Version](README.md) · [Live Demo](mockup.html) · [Architektur-Dokumentation](docs/architecture/)

---

## Was ist Authaver?

Immobilienverwaltung ist geprägt von repetitiver, zeitintensiver Arbeit: Post sortieren, Rechnungen prüfen, Mietrückstände nachverfolgen, Nebenkostenbelege sammeln. Die meisten Vermieter verbringen viele Stunden pro Woche mit Verwaltungsaufgaben — oder bezahlen eine externe Hausverwaltung für €30–50 pro Einheit und Monat.

Authaver verfolgt einen anderen Ansatz: **Ein Team spezialisierter KI-Agenten übernimmt die Routine. Du behältst die Kontrolle über alle Entscheidungen.**

Das System ist inspiriert von der [Paperclip-Agentenarchitektur](https://paperclip.ing) — jeder Agent hat eine definierte Rolle, ein monatliches Token-Budget, einen Heartbeat-Takt und schreibt jede Aktion in ein unveränderliches Protokoll. Du bist das Board. Du kannst jeden Agenten jederzeit pausieren, übersteuern oder neu konfigurieren.

---

## Kernfunktionen

### Dashboard
In unter 2 Minuten wissen, was heute Handlungsbedarf hat. Die KI bereitet jeden Morgen eine priorisierte Liste vor — dringende Vorgänge oben, Routineinfos unten.

### KI-Posteingang (Paula)
Jede eingehende E-Mail wird automatisch klassifiziert: Rechnung, Reparaturanfrage, Mietzahlung, Mahnung, Bestätigung. Paula leitet sie weiter und bereitet einen Antwortvorschlag vor.

### Rechnungsprüfung (Rex)
Jede Rechnung wird mit offenen Tickets abgeglichen. Rex flaggt Abweichungen, erklärt Unterschiede zu Vergleichsrechnungen und schlägt eine Buchung vor. Du gibst frei oder lehnst ab — ein Klick.

### Zahlungsüberwachung (Zara)
Täglicher Abgleich zwischen erwarteten und tatsächlichen Mietingängen. Zara erkennt Rückstände am ersten Tag und leitet sie an Max weiter.

### Mahnwesen (Max)
Max überwacht die Zahlungshistorie und erstellt gestufte Mahnschreiben nach deutschem Mietrecht — Zahlungserinnerung, Mahnung, Abmahnung. Du prüfst und sendest ab.

### Dokumenten-Archiv (Diana)
Jedes eingehende Dokument wird verarbeitet, klassifiziert und abgelegt. Diana erkennt fehlende Belege für die Nebenkostenabrechnung und fordert sie proaktiv an.

### Nebenkostenabrechnung
Jahresabrechnung vorbereiten: Belege sammeln, Umlageschlüssel anwenden, Entwürfe pro Mietpartei generieren — mit rechtskonformen deutschen Vorlagen.

---

## KI-Agenten-Architektur

Authaver nutzt eine Paperclip-inspirierte Agentenarchitektur. Fünf spezialisierte Agenten arbeiten im Hintergrund:

```
Du (Board-Ebene · vollständige Kontrolle)
  └── Paula (Orchestratorin · alle 15 Min.)
        ├── Rex (Rechnungsspezialist · bei Eingang)
        ├── Max (Mahnspezialist · täglich 07:00)
        ├── Diana (Dokumenten-Archivarin · bei Eingang)
        └── Zara (Zahlungsmonitor · täglich 06:00)
```

**Kernprinzipien:**
- Jeder Agent hat ein **monatliches Token-Budget** — verhindert unkontrollierte Kosten
- Jede Aktion wird **unveränderlich protokolliert** — vollständiger Audit-Trail für rechtliche Compliance
- **Konfidenz-Schwellenwerte**: Unter dem Schwellenwert → Eskalation an den Eigentümer
- **Menschliche Kontrolle** jederzeit möglich — pausieren, fortsetzen, übersteuern

Vollständige Muster-Dokumentation: [docs/architecture/paperclip-patterns.md](docs/architecture/paperclip-patterns.md)

---

## Tech-Stack

| Schicht | Technologie | Hinweis |
|---|---|---|
| Frontend | Next.js 15 (App Router) + TypeScript | Server Components, React |
| Styling | Tailwind CSS | shadcn/ui Komponenten |
| Datenbank | PostgreSQL + Prisma | ACID-Transaktionen für Finanzdaten |
| Authentifizierung | Lucia Auth | Self-hosted, kein Drittanbieter |
| KI-Provider | Austauschbar (Anthropic / OpenAI / Ollama) | Kein Vendor-Lock-in |
| E-Mail | IMAP (imapflow) | Direktanbindung ans Postfach |
| Dokumente | pdf-lib + Tesseract (OCR) | Rechnungs- und Vertragsverarbeitung |
| Job-Queue | BullMQ + Redis | Agenten-Heartbeats und Scheduling |
| Deployment | Docker Compose | Self-Hosting mit einer Zeile |

**KI-Provider-Strategie:** Authaver unterstützt mehrere LLM-Anbieter über eine einfache Abstraktionsschicht. Verwende Anthropic Claude oder OpenAI für Cloud-Convenience — oder betreibe Ollama lokal für vollständige Datensouveränität. Gerade für DSGVO-konforme Nutzung wichtig.

---

## Self-Hosting

Authaver läuft auf einem Standard-VPS (Hetzner, Netcup oder beliebigem Linux-Server). Anforderungen: 2 vCPU, 4 GB RAM, 20 GB Speicher.

```bash
# Klonen und starten
git clone https://github.com/reyk-zepper/authaver
cd authaver
cp .env.example .env   # KI-Provider-Key eintragen
docker compose up -d

# Im Browser öffnen
open http://localhost:3000
```

**Vollständige Self-Hosting-Anleitung:** Erscheint mit Release v0.1.

---

## Betriebsmodelle

| Modell | Beschreibung |
|---|---|
| **Self-Service** | Du verwaltest selbst. Authaver übernimmt alle Routineaufgaben automatisch. |
| **Control Layer** | Du hast eine externe Hausverwaltung. Authaver gibt dir vollständige Transparenz und Kontrolle. |
| **Managed Setup** | Ein Dienstleister installiert und betreibt Authaver für dich. Du siehst nur was zählt. |

---

## Monetarisierungsmodell

Authaver ist MIT-lizenziert und kostenlos self-hostbar. Die Software selbst kostet nichts.

**Professioneller Einrichtungsservice** (von uns und zertifizierten Partnern):
- Server-Provisionierung und Docker-Deployment
- Datenmigration aus Excel, CSV oder bestehender Hausverwaltungssoftware
- Konfiguration für dein spezifisches Bundesland (Rechtsvorlagen, Steuereinstellungen)
- DATEV / lexoffice / sevDesk Export-Integration
- SEPA-Lastschrift-Konfiguration
- Erstbefüllung mit Liegenschaften, Einheiten, Mietverträgen
- 2-stündige Einweisungssession per Video-Call
- Optional: Jahressupportvertrag

Das ist das **Red Hat Modell**: offene Software, bezahlte Expertise. Der Burggraben ist Wissen und Vertrauen, nicht Code.

---

## Deutsche Markspezifika

Authaver ist primär für den deutschen Markt entwickelt:

- **DSGVO-Konformität** eingebaut (self-hosted = Daten bleiben auf deinem Server)
- **Rechtsvorlagen** für alle 16 Bundesländer
- **Nebenkostenabrechnung** nach §556 BGB
- **Mietrechtkonformes** Mahnverfahren (Zahlungserinnerung → Mahnung → Abmahnung → Kündigung)
- **SEPA-Lastschrift** Support
- **DATEV-Export** für Steuerberater
- **Grundsteuer-Tracking**

---

## Roadmap

**v0.1 — MVP (In Entwicklung)**
- [ ] Liegenschafts- und Mieterverwaltung
- [ ] IMAP E-Mail-Integration + Paula Klassifikation
- [ ] Dashboard mit KI-Prioritätsliste
- [ ] Ticket-Management mit Verlauf
- [ ] Rechnungserfassung und Rex-Matching
- [ ] Basis-Dokumentenarchiv
- [ ] Docker Compose Deployment

**v0.2 — Kern-Agenten**
- [ ] Max Mahnwesen-Workflow (gestuft)
- [ ] Zara Zahlungsüberwachung + Kontoanbindung
- [ ] Diana Dokumenten-Lückenerkennung
- [ ] Vollständige Audit-Log-Oberfläche

**v0.3 — Nebenkostenabrechnung**
- [ ] NK-Abrechnungs-Workflow
- [ ] Mietermäßige Umlagenberechnung
- [ ] PDF-Generierung

**v1.0 — Produktionsreif**
- [ ] Multi-Liegenschafts-Support
- [ ] DATEV-Export
- [ ] SEPA-Integration
- [ ] Mobile-responsive UI
- [ ] Umfassende Testabdeckung

---

## Mitmachen

Authaver ist open source und freut sich über Beiträge. Die Codebasis ist als modulares Next.js-Monorepo aufgebaut — du kannst an einem einzelnen Modul beitragen, ohne das Gesamtsystem verstehen zu müssen.

```bash
git clone https://github.com/reyk-zepper/authaver
cd authaver
npm install
npm run dev
```

**Gut geeignete erste Issues:** [github.com/reyk-zepper/authaver/issues](https://github.com/reyk-zepper/authaver/issues)

Bitte lies [CONTRIBUTING.md](CONTRIBUTING.md) bevor du einen PR öffnest.

---

## Lizenz

MIT — nutze es, forke es, baue darauf auf. Wenn du es als Service für andere betreibst, musst du deine Änderungen nicht veröffentlichen (MIT, nicht AGPL). Beiträge zurück sind aber immer willkommen.

---

## Warum "Authaver"?

*Authaver* = *Authority* + *Haven*. Dein Eigentum, deine Autorität, dein sicherer Hafen. Die KI übernimmt den Lärm. Du behältst die Kontrolle.

---

<p align="center">
  Mit ♥ gebaut · Open Source · MIT-Lizenz<br>
  <a href="README.md">🇬🇧 English README</a>
</p>
