# Code-Conventions — Authaver

Verbindliche Coding-Standards. Gelten für alle Stories und alle Commits.

---

## TypeScript

- **Strict Mode** ist aktiviert — kein `any`, kein `@ts-ignore`
- Alle Funktionen haben explizite Return-Types wenn nicht trivial inferierbar
- `interface` für Objekt-Shapes, `type` für Unions und Aliases
- Keine `enum` — stattdessen `as const` Objects oder Zod-Enums
- `zod` für alle externen Inputs (Formulare, LLM-Outputs, CSV-Imports)

```typescript
// ✅ Gut
const TICKET_STATUS = ['Neu', 'In Bearbeitung', 'Erledigt'] as const
type TicketStatus = typeof TICKET_STATUS[number]

// ❌ Schlecht
enum TicketStatus { Neu, InBearbeitung, Erledigt }
```

## Dateinamen & Verzeichnisse

- Verzeichnisse: `kebab-case` (z.B. `invoice-analysis/`)
- React-Komponenten: `PascalCase.tsx` (z.B. `InvoiceCard.tsx`)
- Utility-Funktionen: `camelCase.ts` (z.B. `formatCurrency.ts`)
- Server Actions: `actions.ts` in der zugehörigen Route
- API-Routen: `route.ts` (Next.js Konvention)

## Next.js App Router

- **Server Components** sind der Default — `'use client'` nur wenn nötig (Event Handler, Hooks, Browser APIs)
- **Server Actions** für alle Datenmutationen — kein `fetch('/api/...')` aus dem Client für Mutationen
- Jede Route-Group hat eigene `layout.tsx` nur wenn sie eigenes Layout braucht
- Loading States: `loading.tsx` Dateien, kein manuelles `useState(isLoading)`
- Error Handling: `error.tsx` Dateien für Routen-Level-Errors

```typescript
// ✅ Server Action
'use server'
export async function createTicket(data: TicketInput) {
  const validated = TicketSchema.parse(data)
  await prisma.ticket.create({ data: validated })
  revalidatePath('/tickets')
}

// ❌ Client-Side Fetch für Mutation
const res = await fetch('/api/tickets', { method: 'POST', body: JSON.stringify(data) })
```

## Prisma

- Kein `prisma.$queryRaw` außer für Full-Text-Search-Indizes
- Immer `select` oder `include` explizit angeben — kein blindes `findMany()` ohne Feldauswahl
- `AgentLog`: **Kein** `prisma.agentLog.update()` oder `prisma.agentLog.delete()` — Append-Only
- Transactions für Operationen die mehrere Tabellen betreffen

```typescript
// ✅ Explizite Feldauswahl
const tenant = await prisma.tenant.findUnique({
  where: { id },
  select: { id: true, name: true, email: true }
})

// ❌ Alles laden
const tenant = await prisma.tenant.findUnique({ where: { id } })
```

## Monetäre Werte

- In DB: **immer Integer (Cent)** — `rentCents: Int`
- In UI: **immer formatiert** — `formatCurrency(rentCents)` → `"1.200,00 €"`
- Niemals Floats für Geldbeträge

```typescript
// lib/format.ts
export function formatCurrency(cents: number): string {
  return new Intl.NumberFormat('de-DE', {
    style: 'currency',
    currency: 'EUR'
  }).format(cents / 100)
}
```

## KI-Aufrufe

- Niemals direkt `@anthropic-ai/sdk` importieren außerhalb von `lib/ai/providers/`
- Immer via `getAIProvider()` aus `lib/ai/index.ts`
- Alle LLM-Outputs mit Zod-Schema validieren
- `tokensConsumed` immer in `AgentLog` schreiben nach jedem Aufruf

```typescript
// ✅ Korrekt
import { getAIProvider } from '@/lib/ai'
const ai = getAIProvider()
const result = await ai.classify(text, ClassificationSchema)

// ❌ Direkt
import Anthropic from '@anthropic-ai/sdk'
```

## Fehlerbehandlung

- Server Actions geben `{ success: true, data: T }` oder `{ success: false, error: string }` zurück
- Niemals rohe Prisma-Fehler an den Client durchreichen
- `try/catch` in allen Server Actions und Worker-Jobs
- Unerwartete Fehler in Worker-Jobs: Job-Fehler loggen, Retry nach exponential backoff (BullMQ Default)

## Komponenten-Konventionen

- Komponenten-Props immer als `interface` definiert, nicht als inline type
- Keine direkten Datenbankaufrufe in Client-Komponenten
- shadcn/ui Komponenten unter `components/ui/` — nicht modifizieren
- Custom Komponenten unter `components/` (z.B. `components/tickets/TicketCard.tsx`)

## Kommentare

- Nur kommentieren, wenn die Logik nicht selbst-erklärend ist
- Keine JSDoc-Kommentare für triviale Funktionen
- `// TODO:` nur mit GitHub Issue-Nummer: `// TODO: #42 — Bank-CSV-Import`

## Git

- Commit-Messages auf Englisch, Conventional Commits Format: `feat:`, `fix:`, `chore:`, `docs:`
- Eine Story = ein oder mehrere Commits, aber ein klar abgegrenzter PR
- Kein direktes Committen auf `main` — Branch pro Story: `feat/us-001-boilerplate`
