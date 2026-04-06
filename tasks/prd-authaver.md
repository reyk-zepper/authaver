# PRD: Authaver — Vollautomatisierte Open-Source Hausverwaltung

**Version:** 0.1  
**Datum:** 2026-04-06  
**Status:** In Arbeit  
**Referenzen:** [README.md](../README.md) · [Paperclip-Patterns](../docs/architecture/paperclip-patterns.md) · [Mockup](../mockup.html)

---

## 1. Introduction / Überblick

Authaver ist eine selbst-hostbare, open-source Hausverwaltungsplattform für Privateigentümer und kleine Verwaltungsgesellschaften. Fünf spezialisierte KI-Agenten (nach Paperclip-Architektur) übernehmen den gesamten Verwaltungsalltag: Posteingang klassifizieren, Rechnungen prüfen, Mietrückstände überwachen, Dokumente archivieren und Mahnschreiben verfassen. Der Eigentümer entscheidet nur noch bei Ausnahmen.

**Das Problem:** Eigentümer mit kleinen bis mittleren Immobilienbeständen zahlen heute entweder €30–50/Einheit/Monat an externe Verwalter, oder sie verbringen 5–10 Stunden pro Woche mit administrativen Routineaufgaben. Beides ist unnötig.

**Die Lösung:** Authaver automatisiert die Routine vollständig. Die KI arbeitet rund um die Uhr. Der Eigentümer sieht täglich ein priorisiertes Dashboard und gibt kritische Entscheidungen frei — in unter 2 Minuten.

**Monetarisierung:** Die Software ist MIT-lizenziert und kostenlos. Der Mehrwert entsteht durch den professionellen Einrichtungsservice (Setup, Migration, Konfiguration) — das Red Hat Modell.

---

## 2. Ziele

- Eigentümer-Dashboard: in unter 2 Minuten den Tagesstatus überblicken
- Nullmanueller Dateneingabe-Aufwand: alle Dokumente, E-Mails und Belege automatisch verarbeitet
- Vollständiger Audit-Trail: jede KI-Aktion nachvollziehbar und überschreibbar
- Selbst-hostbar: One-Command-Deployment via Docker Compose
- DSGVO-konform: alle Daten auf eigenem Server, kein Cloud-Zwang
- Deutschen Mietrechtsanforderungen entsprechen: §556 BGB NK-Abrechnung, Mahnfristen, Rechtsvorlagen
- Open-Source-Contributor-freundlich: modulares Next.js-Monorepo, klare Modul-Grenzen

---

## 3. User Stories

Jede Story ist ein vertikaler Slice (DB → API → UI) und in einer Ralph-Session implementierbar.  
Phase 0–2 bilden den MVP. Phase 3–10 vervollständigen das Produkt.

---

### PHASE 0: FUNDAMENT

---

### US-001: Projekt-Boilerplate
**Beschreibung:** Als Entwickler brauche ich ein lauffähiges Next.js-Projekt mit TypeScript, PostgreSQL, Prisma und Docker Compose, damit alle weiteren Stories auf einer stabilen Basis aufbauen können.

**Acceptance Criteria:**
- [ ] `npx create-next-app` mit App Router, TypeScript, Tailwind CSS, ESLint
- [ ] Prisma konfiguriert mit PostgreSQL-Connection via `DATABASE_URL`
- [ ] `docker-compose.yml` startet PostgreSQL und die Next.js-App
- [ ] `npm run dev` startet ohne Fehler, Browser zeigt Next.js Default-Page
- [ ] `.env.example` enthält alle benötigten Variablen mit Kommentaren
- [ ] `npm run typecheck` läuft ohne Fehler
- [ ] `npm run lint` läuft ohne Fehler

---

### US-002: Authentifizierung
**Beschreibung:** Als Eigentümer möchte ich mich mit E-Mail und Passwort anmelden, damit meine Daten geschützt sind und nur ich Zugriff habe.

**Acceptance Criteria:**
- [ ] Lucia Auth integriert mit Prisma Session-Adapter
- [ ] Login-Seite unter `/login` mit E-Mail + Passwort Formular
- [ ] Erfolgreicher Login leitet auf `/dashboard` weiter
- [ ] Fehlerhafter Login zeigt Fehlermeldung "Ungültige Anmeldedaten"
- [ ] Logout-Button auf jeder geschützten Seite, löscht Session
- [ ] Alle Routen außer `/login` sind ohne Session nicht erreichbar (redirect auf `/login`)
- [ ] Passwort wird mit bcrypt (min. 12 Runden) gehasht gespeichert
- [ ] `npm run typecheck` läuft ohne Fehler
- [ ] Verify in browser: Login-Flow, Logout, geschützte Route ohne Session

---

### US-003: Core-Datenbankschema
**Beschreibung:** Als Entwickler brauche ich das vollständige Datenmodell in Prisma, damit alle Business-Entitäten von Anfang an sauber modelliert sind.

**Schema-Entitäten:**
```
User → Property → Unit → Lease → Tenant
Property → Document
Property → Ticket
Property → Invoice
Property → Payment
AgentLog
AgentConfig
PendingAction
```

**Acceptance Criteria:**
- [ ] Prisma Schema enthält alle Modelle (s. Technical Considerations)
- [ ] `prisma migrate dev --name init` läuft ohne Fehler durch
- [ ] Alle Foreign Keys korrekt gesetzt, kaskadierendes Löschen wo sinnvoll
- [ ] Enum-Typen definiert: `TicketStatus`, `TicketCategory`, `InvoiceStatus`, `PaymentStatus`, `AgentStatus`, `DocumentCategory`, `ActionType`
- [ ] Seed-Script `prisma/seed.ts` legt Demo-Datensatz an (s. Mockup-Daten)
- [ ] `npx prisma db seed` läuft ohne Fehler
- [ ] `npm run typecheck` läuft ohne Fehler

---

### PHASE 1: KERNDATEN-VERWALTUNG

---

### US-004: Liegenschaft anlegen und verwalten
**Beschreibung:** Als Eigentümer möchte ich meine Liegenschaften anlegen und bearbeiten, damit ich die Basis für alle weiteren Verwaltungsaufgaben habe.

**Acceptance Criteria:**
- [ ] `/properties/new` zeigt Formular: Adresse, Stadt, PLZ, Gebäudeart, Bankverbindung (IBAN), Notizen
- [ ] Validierung: Adresse und Stadt Pflichtfelder
- [ ] IBAN wird validiert (Länge und Prüfziffer-Format)
- [ ] Gespeicherte Liegenschaft erscheint in Liegenschaftsliste
- [ ] Liegenschaft editierbar via `/properties/[id]/edit`
- [ ] Liegenschaft löschbar (nur wenn keine Einheiten vorhanden, sonst Fehlermeldung)
- [ ] Server Action erstellt/aktualisiert die Property in PostgreSQL
- [ ] `npm run typecheck` läuft ohne Fehler
- [ ] Verify in browser: Anlegen, Bearbeiten, Löschen

---

### US-005: Einheiten verwalten
**Beschreibung:** Als Eigentümer möchte ich Einheiten (Wohnungen) innerhalb einer Liegenschaft anlegen, damit ich den Bestand strukturieren kann.

**Acceptance Criteria:**
- [ ] `/properties/[id]/units/new` zeigt Formular: Einheitsnummer, Typ (Wohnen/Gewerbe/Garage), Fläche (m²), Zimmer, Etage
- [ ] Einheitenliste unter `/properties/[id]/units` mit Status (Vermietet / Leerstand)
- [ ] Status wird automatisch aus aktivem Mietvertrag abgeleitet
- [ ] Einheit editierbar und löschbar (nur wenn kein aktiver Mietvertrag, sonst Fehlermeldung)
- [ ] `npm run typecheck` läuft ohne Fehler
- [ ] Verify in browser: Anlegen, Bearbeiten, Status-Anzeige

---

### US-006: Mieter und Mietverträge verwalten
**Beschreibung:** Als Eigentümer möchte ich Mieter erfassen und ihnen Mietverträge zuweisen, damit ich Zahlungserwartungen und Kontaktdaten zentral habe.

**Acceptance Criteria:**
- [ ] Mieter-Formular: Vorname, Nachname, E-Mail, Telefon, Adresse
- [ ] Mietvertrag verknüpft Mieter + Einheit: Mietbeginn, Mietende (optional), Kaltmiete, NK-Vorauszahlung, Kaution, Bankverbindung Mieter
- [ ] Aktiver Mietvertrag setzt Einheit-Status auf "Vermietet"
- [ ] Mietvertrag hat Status: aktiv / gekündigt / ausgelaufen
- [ ] Mieterliste mit Schnellsuche (Name, Einheit)
- [ ] `npm run typecheck` läuft ohne Fehler
- [ ] Verify in browser: Mieter anlegen, Vertrag erstellen, Status sichtbar in Einheitenliste

---

### PHASE 2: DASHBOARD

---

### US-007: Dashboard-Layout und Statistiken
**Beschreibung:** Als Eigentümer möchte ich auf dem Dashboard sofort die wichtigsten Kennzahlen meiner Liegenschaft sehen.

**Acceptance Criteria:**
- [ ] `/dashboard` zeigt 4 Stat-Karten: Einheiten (belegt/gesamt), Monatliche Soll-Miete (€), Neue Nachrichten (Anzahl), Offene Vorgänge (Anzahl)
- [ ] Werte werden live aus der Datenbank berechnet
- [ ] Monatsbilanz-Widget: Mieteingang (Ist), Rechnungen bezahlt, Offene Forderungen (aus DB)
- [ ] Nächste Fälligkeiten: Liste der in 14 Tagen fälligen Rechnungen/Zahlungen
- [ ] Sidebar-Navigation: Dashboard, Posteingang, Tickets, Rechnungen, Zahlungen, Archiv, Agenten
- [ ] `npm run typecheck` läuft ohne Fehler
- [ ] Verify in browser: Alle Werte korrekt, Navigation funktioniert

---

### US-008: KI-Prioritätsliste auf Dashboard
**Beschreibung:** Als Eigentümer möchte ich auf dem Dashboard eine von der KI priorisierte Liste offener Vorgänge sehen, damit ich sofort weiß, was heute Handlungsbedarf hat.

**Acceptance Criteria:**
- [ ] Dashboard zeigt Liste offener Vorgänge: Tickets (Status ≠ Erledigt), unbezahlte Rechnungen, überfällige Zahlungen, fehlende NK-Belege
- [ ] Priorität wird berechnet: Tickets nach Dringlichkeit + Alter, Rechnungen nach Fälligkeit, Zahlungen nach Verzugstagen
- [ ] Jeder Eintrag zeigt: Titel, Kategorie-Badge, Kurzbeschreibung, Urgency-Farbe (rot/amber/blau)
- [ ] Klick auf Eintrag navigiert zur entsprechenden Detail-Seite
- [ ] Liste wird bei jedem Seitenaufruf neu berechnet (kein Cache)
- [ ] `npm run typecheck` läuft ohne Fehler
- [ ] Verify in browser: Prioritätsliste zeigt korrekte Einträge, Urgency-Farben stimmen, Links funktionieren

---

### PHASE 3: POSTEINGANG — PAULA

---

### US-009: IMAP-E-Mail-Integration
**Beschreibung:** Als System muss ich E-Mails aus einem konfigurierten Postfach lesen und in der Datenbank speichern, damit Paula sie klassifizieren kann.

**Acceptance Criteria:**
- [ ] IMAP-Verbindung konfigurierbar via `.env`: `IMAP_HOST`, `IMAP_PORT`, `IMAP_USER`, `IMAP_PASS`
- [ ] BullMQ-Job `email-fetch` pollt alle 15 Minuten ungelesene E-Mails
- [ ] Neue E-Mails werden in `InboxItem`-Tabelle gespeichert: Absender, Betreff, Vorschau (erste 500 Zeichen), Zeitstempel, Rohdaten (HTML/Text)
- [ ] Bereits importierte E-Mails werden nicht doppelt importiert (Message-ID als Unique Key)
- [ ] Job-Fehler (IMAP-Verbindung schlägt fehl) werden geloggt, Retry nach 5 Minuten
- [ ] `npm run typecheck` läuft ohne Fehler

---

### US-010: Paula — E-Mail-Klassifikation via KI
**Beschreibung:** Als System möchte ich, dass Paula jede neue E-Mail automatisch klassifiziert und priorisiert, damit der Eigentümer strukturierten Posteingang erhält.

**Acceptance Criteria:**
- [ ] Nach `email-fetch` triggert BullMQ-Job `paula-classify` für jede unklassifizierte E-Mail
- [ ] Paula ruft LLM-API auf (Anthropic Claude / OpenAI, via Provider-Abstraktionsschicht)
- [ ] Klassifikation gibt zurück: `category` (Rechnung / Ticket / Anfrage / Beleg / Bestätigung / Sonstiges), `priority` (hoch / mittel / niedrig), `aiNote` (1-2 Sätze KI-Begründung), `confidenceScore` (0–1)
- [ ] Bei `confidenceScore < 0.6`: Kategorie = Sonstiges, `requiresHumanReview = true`
- [ ] Alle Felder werden auf dem `InboxItem` gespeichert
- [ ] Token-Verbrauch wird auf `AgentLog` geschrieben (Agent: Paula)
- [ ] `npm run typecheck` läuft ohne Fehler

---

### US-011: Posteingang-UI
**Beschreibung:** Als Eigentümer möchte ich meinen Posteingang mit KI-Klassifizierungen sehen und navigieren.

**Acceptance Criteria:**
- [ ] `/inbox` zeigt zweispaltiges Layout: E-Mail-Liste links, Detail-Bereich rechts
- [ ] E-Mail-Liste: Absender, Betreff, Uhrzeit, Kategorie-Badge (farbkodiert), Urgent-Indikator
- [ ] Liste sortiert nach: Priorität (hoch zuerst), dann Zeitstempel (neu zuerst)
- [ ] Aktive E-Mail hebt sich in der Liste visuell ab
- [ ] Kategorie-Badges: Rechnung=amber, Anfrage·Miete=red, Beleg=blue, Ticket=indigo, Bestätigung=emerald
- [ ] Ungelesene E-Mails in Bold, gelesen in Normal-Weight
- [ ] `npm run typecheck` läuft ohne Fehler
- [ ] Verify in browser: Liste lädt, Selektion funktioniert, Badges korrekt

---

### US-012: Posteingang-Detail und Aktionen
**Beschreibung:** Als Eigentümer möchte ich eine E-Mail öffnen, den KI-Hinweis lesen und direkt Aktionen ausführen.

**Acceptance Criteria:**
- [ ] Detail-Bereich zeigt: Absender, E-Mail-Adresse, Betreff, Zeitstempel, KI-Kategorie-Badge, KI-Notiz (blauer Infokasten), E-Mail-Text
- [ ] Aktions-Buttons: "Antworten (Entwurf)", "Ticket anlegen", "Als Beleg archivieren", "Ignorieren"
- [ ] "Ticket anlegen": öffnet vorbefülltes Ticket-Formular (Titel aus Betreff, Beschreibung aus E-Mail-Text)
- [ ] "Als Beleg archivieren": speichert als `Document` mit Kategorie aus KI-Klassifikation
- [ ] "Ignorieren": markiert E-Mail als gelesen, entfernt aus ungelesenen
- [ ] Antwort-Entwurf-Funktion: LLM generiert Antwortentwurf basierend auf E-Mail-Inhalt (in einfachem Textfeld anzeigen)
- [ ] `npm run typecheck` läuft ohne Fehler
- [ ] Verify in browser: Aktionen funktionieren, Ticket-Anlegen navigiert korrekt

---

### PHASE 4: TICKET-MANAGEMENT

---

### US-013: Ticket erstellen und verwalten
**Beschreibung:** Als Eigentümer möchte ich Wartungs- und Schadenstickets anlegen und verwalten.

**Acceptance Criteria:**
- [ ] `/tickets/new` zeigt Formular: Titel, Kategorie (Schaden/Wartung/Anfrage/Sonstiges), Liegenschaft, Einheit, Beschreibung, Priorität (hoch/mittel/niedrig), Anhänge (max. 5 Dateien, je max. 10 MB)
- [ ] Ticket-Liste unter `/tickets` mit Filtern: Status, Kategorie, Liegenschaft, Priorität
- [ ] Tickets als Tabelle: Ticket-ID, Titel, Einheit, Status-Badge, Priorität-Indikator, Erstellt-Datum
- [ ] Sortierung: Priorität × Alter (Standard)
- [ ] `npm run typecheck` läuft ohne Fehler
- [ ] Verify in browser: Ticket anlegen, Filter, Sortierung

---

### US-014: Ticket-Detail mit Verlauf
**Beschreibung:** Als Eigentümer möchte ich den vollständigen Verlauf eines Tickets sehen und Antworten hinzufügen.

**Acceptance Criteria:**
- [ ] `/tickets/[id]` zeigt: Header (Titel, Status, Priorität), Verlaufs-Timeline, Seitenleiste (Ticket-Details, Anhänge)
- [ ] Timeline-Einträge: Zeitstempel, Akteur (Eigentümer / Mieter / KI-Agent), Text, Badge "KI-Aktion" bei Agent-Einträgen
- [ ] Antwort-Formular am Ende der Timeline
- [ ] Status-Dropdown direkt im Header änderbar: Neu → In Bearbeitung → Warte auf Antwort → Erledigt
- [ ] Statuswechsel erzeugt automatisch Timeline-Eintrag
- [ ] KI-Vorschlag-Box: LLM schlägt nächste Schritte vor (basierend auf Ticket-Inhalt und Status)
- [ ] `npm run typecheck` läuft ohne Fehler
- [ ] Verify in browser: Timeline lädt, Antwort hinzufügen, Status ändern

---

### US-015: Ticket-Verlinkung und Suche
**Beschreibung:** Als Eigentümer möchte ich Tickets mit E-Mails, Rechnungen und Dokumenten verknüpfen und Tickets durchsuchen.

**Acceptance Criteria:**
- [ ] Ticket kann mit 1..n InboxItems, Invoices, Documents verknüpft werden
- [ ] Verknüpfte Entitäten sichtbar in Ticket-Seitenleiste mit Link
- [ ] Volltext-Suche auf `/tickets` durchsucht Titel + Beschreibung (PostgreSQL ILIKE)
- [ ] Ergebnisse werden live beim Tippen gefiltert (debounced, 300ms)
- [ ] `npm run typecheck` läuft ohne Fehler
- [ ] Verify in browser: Suche findet korrekte Tickets, Verknüpfungen sichtbar

---

### PHASE 5: RECHNUNGSPRÜFUNG — REX

---

### US-016: Rechnungs-Ingestion und -Verwaltung
**Beschreibung:** Als Eigentümer möchte ich eingehende Rechnungen erfassen — automatisch aus E-Mails und manuell.

**Acceptance Criteria:**
- [ ] Paula markiert E-Mails der Kategorie "Rechnung" → BullMQ-Job `invoice-extract` extrahiert: Aussteller, Rechnungsnummer, Betrag (netto, MwSt., brutto), Datum, Fälligkeit, IBAN aus E-Mail-Text via LLM
- [ ] Extrahierte Felder werden als `Invoice` gespeichert, verknüpft mit `InboxItem`
- [ ] Manuelles Rechnungs-Formular unter `/invoices/new`
- [ ] Rechnungsliste unter `/invoices`: Aussteller, Betrag, Fälligkeit, Status-Badge
- [ ] Status: Offen / In Prüfung / Freigegeben / Bezahlt / Abgelehnt
- [ ] `npm run typecheck` läuft ohne Fehler
- [ ] Verify in browser: Rechnungsliste, manuelles Anlegen

---

### US-017: Rex — KI-Rechnungsanalyse
**Beschreibung:** Als System möchte ich, dass Rex jede neue Rechnung mit offenen Tickets abgleicht und Auffälligkeiten erkennt.

**Acceptance Criteria:**
- [ ] BullMQ-Job `rex-analyze` startet automatisch nach `invoice-extract`
- [ ] Rex sucht: offene Tickets der gleichen Liegenschaft im Zeitraum ±14 Tage
- [ ] Rex vergleicht Betrag mit den letzten 5 Rechnungen desselben Ausstellers (Durchschnitt + Abweichung in %)
- [ ] Rex-Ausgabe wird als `InvoiceAnalysis` gespeichert: ticketMatchIds, averageComparable, deviationPercent, flags (Array: "KEIN_TICKET_MATCH", "BETRAG_ABWEICHUNG_>15%", "NEUER_LIEFERANT"), recommendation, confidenceScore
- [ ] Bei `confidenceScore < 0.7`: `requiresHumanReview = true`, Rechnung erscheint in KI-Prioritätsliste
- [ ] Token-Verbrauch geloggt (Agent: Rex)
- [ ] `npm run typecheck` läuft ohne Fehler

---

### US-018: Rechnungs-Detail und Freigabe-Flow
**Beschreibung:** Als Eigentümer möchte ich eine Rechnung mit Rex-Analyse sehen und freigeben oder ablehnen.

**Acceptance Criteria:**
- [ ] `/invoices/[id]` zeigt: Rechnungsdetails (alle Felder), Rex-Analyse-Box (Flags, Vergleichsrechnungen, Empfehlung, Konfidenz-Badge), Aktions-Buttons
- [ ] Aktionen: "Freigeben & überweisen", "Rückfrage senden", "Leistungsnachweis anfordern", "Ablehnen"
- [ ] "Freigeben": Status → Freigegeben, Timeline-Eintrag, Eigentümer-Bestätigungsmeldung
- [ ] "Ablehnen": Status → Abgelehnt, optional Freitext-Begründung
- [ ] Vergleichsrechnungen desselben Ausstellers als Tabelle (Datum, Beschreibung, Betrag)
- [ ] `npm run typecheck` läuft ohne Fehler
- [ ] Verify in browser: Analyse sichtbar, Freigabe/Ablehnung funktioniert, Status ändert sich

---

### PHASE 6: ZAHLUNGSÜBERWACHUNG — ZARA & MAX

---

### US-019: Zahlungsplan und Soll-Eingang
**Beschreibung:** Als System muss ich wissen, welche Zahlungen jeden Monat erwartet werden, damit Zara Abweichungen erkennen kann.

**Acceptance Criteria:**
- [ ] Aus aktiven Mietverträgen wird monatlich ein `ExpectedPayment`-Datensatz generiert (Cronjob am 1. des Monats): Mieter, Einheit, Betrag (Kaltmiete + NK-Vorauszahlung), Fälligkeitsdatum (01. des Monats)
- [ ] `Payment`-Modell speichert tatsächliche Zahlungseingänge: Betrag, Buchungsdatum, Verwendungszweck, IBAN Absender, verknüpfter Mietvertrag
- [ ] Manuelles Erfassen von Zahlungseingängen unter `/payments/new`
- [ ] `npm run typecheck` läuft ohne Fehler

---

### US-020: Zara — Tägliche Zahlungsüberwachung
**Beschreibung:** Als System möchte ich, dass Zara täglich prüft, ob alle erwarteten Zahlungen eingegangen sind.

**Acceptance Criteria:**
- [ ] Heartbeat-Job `zara-monitor` läuft täglich um 06:00 Uhr
- [ ] Zara vergleicht alle `ExpectedPayment` (Fälligkeit ≤ heute) mit tatsächlichen `Payment`-Einträgen
- [ ] Bei fehlendem Eingang nach 3 Kalendertagen: `ExpectedPayment.status = 'OVERDUE'`, `PendingAction` erstellt (Typ: MAHNUNG_ERFORDERLICH)
- [ ] `PendingAction` erscheint in KI-Prioritätsliste des Dashboards
- [ ] Zara loggt alle Aktionen in `AgentLog` mit Token-Verbrauch
- [ ] `npm run typecheck` läuft ohne Fehler

---

### US-021: Max — Gestufte Mahngenerierung
**Beschreibung:** Als Eigentümer möchte ich, dass Max bei Mietrückstand automatisch den korrekten Mahnungsentwurf nach deutschem Mietrecht vorbereitet.

**Acceptance Criteria:**
- [ ] Max-Job `max-draft` triggert wenn Zara eine `PendingAction` (MAHNUNG_ERFORDERLICH) erstellt
- [ ] Max bestimmt Mahnstufe anhand Zahlungshistorie: 1. Verspätung = Zahlungserinnerung, 2. = Mahnung, 3. = Abmahnung, 4+. = Kündigung
- [ ] Max generiert Mahntextvorschlag via LLM mit: Mieter-Name, Einheit, Betrag, Fälligkeit, korrekte Rechtsformulierung je Mahnstufe, Antwortfrist (7 / 14 / 14 Tage)
- [ ] Entwurf als `DocumentDraft` gespeichert, Status = `PENDING_REVIEW`
- [ ] Entwurf erscheint in KI-Prioritätsliste als "Mahnungsfreigabe erforderlich"
- [ ] `npm run typecheck` läuft ohne Fehler

---

### US-022: Zahlungs- und Mahnungs-UI
**Beschreibung:** Als Eigentümer möchte ich den Zahlungsstatus überblicken und Mahnungen prüfen und abschicken.

**Acceptance Criteria:**
- [ ] `/payments` zeigt Monatstabelle: Einheit, Mieter, Soll-Betrag, Ist-Betrag, Status (Pünktlich/Verspätet/Ausstehend/Mahnung)
- [ ] Filter: Monat/Jahr, Liegenschaft, Status
- [ ] Mahnung-Detail: Vorschautext, Mahnstufe-Badge, Bearbeiten-Möglichkeit (Freitext-Editor), Senden- und Verwerfen-Buttons
- [ ] "Senden": Status → GESENDET, Zeitstempel, Timeline-Eintrag auf Mieter-Akte
- [ ] `npm run typecheck` läuft ohne Fehler
- [ ] Verify in browser: Zahlungsübersicht korrekt, Mahnung-Vorschau, Senden funktioniert

---

### PHASE 7: DOKUMENTEN-ARCHIV — DIANA

---

### US-023: Dokument-Upload und automatische Klassifikation
**Beschreibung:** Als Eigentümer möchte ich Dokumente hochladen und automatisch archivieren lassen.

**Acceptance Criteria:**
- [ ] Drag-and-Drop Upload auf `/archive` (max. 20 MB, Formate: PDF, JPG, PNG, XLSX, DOCX)
- [ ] Dateien werden auf lokalem Dateisystem gespeichert (Docker Volume, Pfad via `UPLOAD_PATH` env)
- [ ] Diana-Job `diana-classify` klassifiziert jedes Dokument: `DocumentCategory` (Mietvertrag / Rechnung / Beleg / Versicherung / Protokoll / Behörde / Sonstiges), extrahiert Datum, Aussteller, Kurzinhalt via LLM
- [ ] PDF-Text-Extraktion via `pdf-parse`; für Bild-Dateien: OCR via Tesseract (`tesseract.js`)
- [ ] Klassifiziertes Dokument erscheint im Archiv mit Kategorie-Badge
- [ ] `npm run typecheck` läuft ohne Fehler
- [ ] Verify in browser: Upload, Klassifikation erscheint, Archivliste zeigt Dokument

---

### US-024: Archiv-UI und Suche
**Beschreibung:** Als Eigentümer möchte ich das Archiv durchsuchen und Dokumente abrufen.

**Acceptance Criteria:**
- [ ] `/archive` zeigt Dokumentenliste: Name, Kategorie-Badge, Datum, Aussteller, Liegenschaft, Download-Link
- [ ] Filter: Kategorie, Liegenschaft, Datumsbereich
- [ ] Volltext-Suche über extrahierten Dokument-Inhalt (PostgreSQL Full-Text-Search)
- [ ] Download als Original-Datei
- [ ] Dokument löschbar (mit Bestätigungsdialog)
- [ ] `npm run typecheck` läuft ohne Fehler
- [ ] Verify in browser: Suche, Filter, Download

---

### US-025: Diana — Beleglücken-Erkennung
**Beschreibung:** Als System möchte ich, dass Diana fehlende Belege für die Nebenkostenabrechnung erkennt und meldet.

**Acceptance Criteria:**
- [ ] Diana-Heartbeat-Job läuft wöchentlich (Montag 08:00)
- [ ] Diana prüft: Gibt es für jede registrierte Nebenkostenposition der Liegenschaft (Heizung, Wasser, Versicherung, Hausmeister, etc.) einen archivierten Beleg für das laufende NK-Jahr?
- [ ] Fehlende Positionen werden als `PendingAction` (Typ: BELEG_FEHLT) erstellt
- [ ] `PendingAction` erscheint in Dashboard-Prioritätsliste
- [ ] Diana loggt in `AgentLog`
- [ ] `npm run typecheck` läuft ohne Fehler

---

### PHASE 8: AGENTEN-INFRASTRUKTUR

---

### US-026: AI-Provider-Abstraktionsschicht
**Beschreibung:** Als Entwickler brauche ich eine Provider-unabhängige Schnittstelle für alle LLM-Aufrufe, damit Nutzer zwischen Anthropic, OpenAI und Ollama wählen können.

**Acceptance Criteria:**
- [ ] Interface `AIProvider` mit Methoden: `classify(text, schema)`, `extract(text, schema)`, `draft(type, context)`, `suggest(context)`
- [ ] Implementierungen: `AnthropicProvider` (claude-3-5-sonnet), `OpenAIProvider` (gpt-4o-mini), `OllamaProvider` (local)
- [ ] Provider wird via `AI_PROVIDER` env gewählt (default: anthropic)
- [ ] Alle bestehenden Agent-Jobs nutzen diese Abstraktion (keine direkten SDK-Importe außerhalb der Provider-Implementierungen)
- [ ] Zod-Schemas für alle strukturierten Outputs (kein freies JSON-Parsing)
- [ ] `npm run typecheck` läuft ohne Fehler

---

### US-027: Agenten-Budget-Kontrolle
**Beschreibung:** Als System muss ich den Token-Verbrauch jedes Agenten überwachen und bei Budget-Überschreitung pausieren.

**Acceptance Criteria:**
- [ ] `AgentConfig`-Tabelle: Agent-ID, monatliches Token-Budget, konsumiertTokens (Monat), Status (ACTIVE/PAUSED/ALERT)
- [ ] Jeder LLM-Aufruf schreibt verbrauchte Tokens in `AgentLog` und aktualisiert `AgentConfig.consumedTokens`
- [ ] Bei `consumedTokens >= budget * 0.8`: Status = ALERT, Eigentümer-Benachrichtigung (Dashboard-Banner)
- [ ] Bei `consumedTokens >= budget`: Status = PAUSED, Agent führt keine weiteren LLM-Aufrufe durch, `PendingAction` erstellt
- [ ] Reset am 1. des Monats via Cron-Job (consumedTokens → 0, Status → ACTIVE wenn nicht manuell pausiert)
- [ ] `npm run typecheck` läuft ohne Fehler

---

### US-028: Unveränderlicher Audit-Log
**Beschreibung:** Als Eigentümer möchte ich eine vollständige, unveränderliche Protokollierung aller Agenten-Aktionen sehen.

**Acceptance Criteria:**
- [ ] `AgentLog`-Einträge sind nie update- oder deletebar (Prisma: kein `update`/`delete` auf `AgentLog`)
- [ ] `/agents/log` zeigt Audit-Log-Tabelle: Zeitstempel, Agent, Aktionstyp, Kurzinhalt, Tokens, Konfidenz, Menschliche Entscheidung (falls vorhanden)
- [ ] Filter: Agent, Zeitraum, Aktionstyp
- [ ] CSV-Export des gefilterten Logs
- [ ] `npm run typecheck` läuft ohne Fehler
- [ ] Verify in browser: Log lädt, Filter, Export

---

### US-029: Agenten-Kontrollpanel
**Beschreibung:** Als Eigentümer möchte ich meine KI-Agenten überwachen, konfigurieren und bei Bedarf pausieren.

**Acceptance Criteria:**
- [ ] `/agents` zeigt 5 Agent-Karten: Name, Rolle, Status-Dot (animiert bei aktiv), letzte Aktion, Budget-Balken, verarbeitete Einheiten (Monat)
- [ ] Pause / Aktivieren per Toggle (Status-Änderung in `AgentConfig`)
- [ ] Monatsbudget anpassbar (Inline-Edit, Speichern via Server Action)
- [ ] Org-Chart-Visualisierung: Du → Paula → Spezialisten (statisch gerendert)
- [ ] Klick auf Agent zeigt letzten `AgentLog`-Eintrag in Overlay
- [ ] `npm run typecheck` läuft ohne Fehler
- [ ] Verify in browser: Karten laden, Pause/Aktivieren, Budget-Edit

---

### PHASE 9: NEBENKOSTENABRECHNUNG

---

### US-030: NK-Beleg-Sammlung und -Verwaltung
**Beschreibung:** Als Eigentümer möchte ich die Nebenkostenbelege für ein Jahr zusammenstellen und fehlende erkennen.

**Acceptance Criteria:**
- [ ] `/nk/[year]` zeigt Belegliste für das NK-Jahr: Position (Heizung, Wasser, Versicherung etc.), Status (✓ Vorhanden / ○ Fehlt), Dateiname, Betrag, Datum
- [ ] NK-Positionen sind pro Liegenschaft konfigurierbar (Admin-Bereich)
- [ ] Beleg kann direkt aus Archiv mit NK-Position verknüpft werden
- [ ] Fortschrittsbalken: X von Y Belegen vollständig
- [ ] Fehlende Belege = `PendingAction` (von Diana, US-025)
- [ ] `npm run typecheck` läuft ohne Fehler
- [ ] Verify in browser: Belegliste, Verknüpfung, Fortschrittsanzeige

---

### US-031: NK-Kostenverteilung berechnen
**Beschreibung:** Als System muss ich die Nebenkosten korrekt auf die einzelnen Einheiten umlegen.

**Acceptance Criteria:**
- [ ] Umlageschlüssel pro Liegenschaft konfigurierbar: nach m² (Standard), nach Personenzahl, nach Einheit (gleiche Teile)
- [ ] Berechnung: Gesamtkosten je Position → Anteil je Einheit nach Schlüssel
- [ ] Berücksichtigung von Ein-/Auszugsdaten (Mietzeit im NK-Jahr in Tagen)
- [ ] Vorschau-Tabelle: Einheit × NK-Position × Betrag, Summe je Einheit
- [ ] Berechnung kann jederzeit neu ausgeführt werden (nicht destruktiv)
- [ ] `npm run typecheck` läuft ohne Fehler
- [ ] Verify in browser: Berechnungsvorschau korrekt, Summen stimmen

---

### US-032: NK-Abrechnung generieren und versenden
**Beschreibung:** Als Eigentümer möchte ich die Nebenkostenabrechnung als PDF für jeden Mieter generieren.

**Acceptance Criteria:**
- [ ] PDF-Generierung via `@react-pdf/renderer` oder `pdfkit`
- [ ] PDF enthält: Liegenschaft, Mieter, Abrechnungszeitraum, NK-Positionen mit Beträgen, Umlageschlüssel, Vorauszahlungen des Mieters, Nachzahlung / Guthaben, Rechtshinweis (§556 BGB), Unterschriftsfeld
- [ ] PDF entspricht gängigen deutschen NK-Abrechnungs-Standards
- [ ] Alle PDFs als ZIP downloadbar
- [ ] Optionaler E-Mail-Versand an Mieter (via SMTP-Konfiguration)
- [ ] `npm run typecheck` läuft ohne Fehler
- [ ] Verify in browser: PDF-Download, ZIP korrekt, E-Mail-Versand (mit Test-Account)

---

### PHASE 10: ONBOARDING & MIGRATION

---

### US-033: Onboarding-Assistent
**Beschreibung:** Als neuer Nutzer möchte ich durch einen geführten Setup-Prozess geleitet werden, damit ich die Plattform schnell in Betrieb nehmen kann.

**Acceptance Criteria:**
- [ ] `/onboarding` zeigt mehrstufigen Wizard: 1. Liegenschaft anlegen, 2. Einheiten anlegen, 3. Mieter + Verträge, 4. E-Mail-Verbindung konfigurieren, 5. KI-Provider wählen und testen
- [ ] Fortschrittsanzeige (Step X von 5)
- [ ] Jeder Schritt validiert vor Weiter-Button
- [ ] Abgeschlossene Schritte bleiben sichtbar (zusammengeklappt)
- [ ] "Setup überspringen" Link auf jeder Stufe → direkt zum Dashboard
- [ ] Nach Abschluss: Redirect auf Dashboard, Onboarding-Banner ausgeblendet
- [ ] `npm run typecheck` läuft ohne Fehler
- [ ] Verify in browser: Wizard durchlaufen, Validierung, Redirect

---

### US-034: CSV/Excel-Datenmigration
**Beschreibung:** Als Eigentümer mit Bestandsdaten möchte ich Mieter, Einheiten und Verträge aus CSV/Excel importieren, damit ich nicht alles manuell erfassen muss.

**Acceptance Criteria:**
- [ ] Upload-Formular unter `/import` für CSV und XLSX (max. 5 MB)
- [ ] CSV-Format definiert: `tenant_name, email, phone, unit_number, rent_amount, nk_amount, start_date, end_date`
- [ ] Vorschau-Tabelle nach Upload: erste 10 Zeilen, erkannte Felder, Fehler-Highlighting bei ungültigen Zeilen
- [ ] Import-Logik: neue Mieter anlegen, neue Einheiten anlegen, Mietverträge verknüpfen
- [ ] Bei Duplikat (gleiche E-Mail): Warnung, Option "Überschreiben" oder "Überspringen"
- [ ] Import-Report: X erfolgreich, Y übersprungen, Z fehlgeschlagen (mit Fehlermeldungen)
- [ ] `npm run typecheck` läuft ohne Fehler
- [ ] Verify in browser: Upload, Vorschau, Import, Report

---

## 4. Funktionale Anforderungen

- **FR-01:** Alle Seiten außer `/login` und `/onboarding` erfordern eine aktive Session (Middleware-Guard)
- **FR-02:** Alle Formulare nutzen Server Actions (Next.js App Router), kein direktes Client-Side-Fetching für Mutationen
- **FR-03:** Alle KI-Ausgaben enthalten `confidenceScore`. Bei Score < 0.6 wird immer `requiresHumanReview = true` gesetzt
- **FR-04:** `PendingAction`-Datensätze verfallen automatisch nach 7 Tagen ohne Entscheidung (Cron-Job, tägliche Bereinigung)
- **FR-05:** Alle Datei-Uploads werden serverseitig auf Typ und Größe validiert (kein Trust auf Client-Angaben)
- **FR-06:** Alle LLM-Aufrufe sind via `AI_PROVIDER` env austauschbar, kein direkter SDK-Import außerhalb von `lib/ai/providers/`
- **FR-07:** Das BullMQ-Dashboard ist unter `/admin/queues` erreichbar (nur in Development-Mode)
- **FR-08:** Alle Datenbankzugriffe laufen über Prisma — kein Raw-SQL außer für Full-Text-Search-Indizes
- **FR-09:** Token-Verbrauch jedes LLM-Aufrufs wird in `AgentLog.tokensConsumed` gespeichert
- **FR-10:** `AgentLog`-Einträge dürfen nie gelöscht oder aktualisiert werden (Append-Only)
- **FR-11:** Docker Compose startet vollständig via `docker compose up -d` — keine manuelle Konfiguration nach `.env` setup
- **FR-12:** Alle monetären Beträge werden als Integer (Cent) in der Datenbank gespeichert, nur in der UI mit `€`-Symbol und Dezimaltrenner dargestellt

---

## 5. Non-Goals (Außerhalb des Scope)

- ❌ Mehrmandanten-SaaS (jede Instanz = ein Eigentümer / ein kleines Team)
- ❌ Mobile App (native iOS/Android) — responsive Web reicht
- ❌ Echtzeit-Kollaboration (kein WebSocket-basiertes Co-Editing)
- ❌ Direkter Bank-API-Anschluss (kein PSD2/Open-Banking in v1 — manuelle Zahlungserfassung)
- ❌ KI-gesteuerte Mietpreisoptimierung oder Marktanalyse
- ❌ Vermieter-Mieter-Portal (Mieter haben keinen eigenen Login in v1)
- ❌ Automatisches E-Mail-Versenden (alle Entwürfe werden nur vorbereitet, nie ohne Freigabe gesendet)
- ❌ WEG-Verwaltung (Wohnungseigentümergemeinschaft mit Abstimmungsprozessen) — zu komplex für v1
- ❌ Buchführung / doppelte Buchführung — nur Zahlungserfassung und NK-Abrechnung
- ❌ Mehrsprachige UI (nur Deutsch in v1)

---

## 6. Design-Überlegungen

- **Referenz-Mockup:** `mockup.html` im Projektroot — alle UI-Screens sind dort interaktiv implementiert und als Designvorgabe zu verwenden
- **Komponentenbibliothek:** shadcn/ui auf Tailwind CSS-Basis
- **Farbsystem:** Urgent=red-500, Warning=amber-500, Info=blue-400, Success=emerald-500, Primary=indigo-600
- **KI-Boxen:** Immer mit indigo-50 Hintergrund, "KI-Vorschlag"-Label, Konfidenz-Badge (emerald/amber/red)
- **Agent-Status-Dots:** Animierter Pulse bei Active und Alert-Status
- **Typografie:** Inter (Google Fonts), System-Fallback: -apple-system, BlinkMacSystemFont
- **Layout:** Sidebar (fixed, 240px) + Main-Content (ml-240), sticky TopBar
- **Mobile:** Responsive ab 768px (Sidebar wird Hamburger-Menü)

---

## 7. Technische Überlegungen

### Datenbank-Schema (Kernmodelle)

```prisma
model Property {
  id        String   @id @default(cuid())
  name      String
  address   String
  city      String
  iban      String?
  units     Unit[]
  documents Document[]
  tickets   Ticket[]
  invoices  Invoice[]
  createdAt DateTime @default(now())
}

model Unit {
  id         String   @id @default(cuid())
  property   Property @relation(...)
  number     String
  type       UnitType
  areaSqm    Float?
  leases     Lease[]
}

model Tenant {
  id     String  @id @default(cuid())
  name   String
  email  String  @unique
  phone  String?
  leases Lease[]
}

model Lease {
  id          String   @id @default(cuid())
  unit        Unit     @relation(...)
  tenant      Tenant   @relation(...)
  startDate   DateTime
  endDate     DateTime?
  rentCents   Int      // Kaltmiete in Cent
  nkCents     Int      // NK-Vorauszahlung in Cent
  depositCents Int?
  status      LeaseStatus
  payments    Payment[]
}

model InboxItem {
  id               String    @id @default(cuid())
  messageId        String    @unique  // IMAP Message-ID
  from             String
  subject          String
  preview          String
  rawText          String    @db.Text
  receivedAt       DateTime
  category         InboxCategory?
  priority         Priority?
  aiNote           String?
  confidenceScore  Float?
  requiresReview   Boolean   @default(false)
  isRead           Boolean   @default(false)
  invoices         Invoice[]
  tickets          Ticket[]
  documents        Document[]
}

model AgentLog {
  id              String    @id @default(cuid())
  agentId         String
  actionType      String
  inputSummary    String
  outputSummary   String
  confidenceScore Float?
  tokensConsumed  Int
  escalated       Boolean   @default(false)
  humanDecision   String?
  humanDecisionAt DateTime?
  createdAt       DateTime  @default(now())
  // Kein update/delete erlaubt — Append-Only
}

model AgentConfig {
  id               String      @id
  name             String
  monthlyBudget    Int         // in Tokens
  consumedTokens   Int         @default(0)
  status           AgentStatus @default(ACTIVE)
  budgetResetAt    DateTime
}

model PendingAction {
  id          String    @id @default(cuid())
  agentId     String
  actionType  ActionType
  title       String
  draft       Json?
  status      String    @default("PENDING")
  expiresAt   DateTime
  createdAt   DateTime  @default(now())
  resolvedAt  DateTime?
}
```

### Abhängigkeiten (package.json)

```json
{
  "dependencies": {
    "next": "15.x",
    "@prisma/client": "^5",
    "lucia": "^3",
    "@lucia-auth/adapter-prisma": "^1",
    "bullmq": "^5",
    "imapflow": "^1",
    "@anthropic-ai/sdk": "^0.36",
    "openai": "^4",
    "ollama": "^0.5",
    "zod": "^3",
    "pdf-parse": "^1",
    "tesseract.js": "^5",
    "bcrypt": "^5",
    "date-fns": "^3",
    "recharts": "^2"
  }
}
```

### Verzeichnisstruktur

```
authaver/
├── app/                    # Next.js App Router
│   ├── (auth)/login/
│   ├── (app)/
│   │   ├── dashboard/
│   │   ├── inbox/
│   │   ├── tickets/
│   │   ├── invoices/
│   │   ├── payments/
│   │   ├── archive/
│   │   ├── agents/
│   │   ├── nk/
│   │   └── import/
│   └── api/
├── lib/
│   ├── ai/
│   │   ├── providers/      # AnthropicProvider, OpenAIProvider, OllamaProvider
│   │   └── index.ts        # AIProvider Interface + Factory
│   ├── agents/             # Paula, Rex, Max, Diana, Zara Job-Definitionen
│   ├── db/                 # Prisma Client Singleton
│   └── auth/               # Lucia Auth Setup
├── components/             # shadcn/ui + Custom Components
├── prisma/
│   ├── schema.prisma
│   └── seed.ts
├── workers/                # BullMQ Worker Prozesse
├── specs/                  # Spec-Driven Dev Constraints
├── tasks/                  # PRD + Story-Tracking
└── docker-compose.yml
```

---

## 8. Erfolgsmetriken

- Dashboard lädt in < 500ms (LCP)
- Paula klassifiziert 90% der E-Mails korrekt (Spot-Check nach 100 E-Mails)
- Rex erkennt 100% der Rechnungen ohne Ticket-Match (kein False-Negative)
- NK-Abrechnung PDF entspricht §556 BGB (manuelle Rechtsprüfung vor v1.0)
- Self-Hosting via `docker compose up -d` funktioniert ohne weitere Schritte (nach `.env` setup)
- `npm run typecheck` und `npm run lint` laufen ohne Fehler (CI-Gate)

---

## 9. Offene Fragen

1. **Bank-Integration:** Sollen wir eine manuelle Kontoauszug-Import-Funktion (CSV/MT940) für Zara einbauen, bevor echtes Open-Banking verfügbar ist?
2. **E-Mail-Versand:** Welcher SMTP-Provider wird empfohlen für die Demo-Einrichtung? (Resend, Mailgun, eigener Postfix?)
3. **Ollama-Hardware:** Welche Modell-Empfehlung für Ollama bei 8 GB RAM? (phi-3-mini, llama3.2:3b?)
4. **DATEV-Export:** Format für DATEV-Buchungssatz (CSV-Format der Kanzlei) — Anforderungen erst bei v0.3 klären?
5. **Multi-Property:** Soll die Sidebar eine Property-Selektion zeigen (wie im Mockup), oder sind alle Properties gleichzeitig sichtbar?
