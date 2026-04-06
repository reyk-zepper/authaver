# Paperclip Agent Patterns in Authaver

> Reference: [paperclip.ing](https://paperclip.ing) — Open-source AI company orchestration platform.
> Paperclip reframes agent management as running a company: agents have roles, budgets, reporting lines, and heartbeats.
> We adopt these patterns and adapt them for property management.

---

## Why Paperclip as Reference Architecture

Paperclip solves the same fundamental problem we face: **coordinating multiple AI agents reliably, cheaply, and transparently** — without the agent system growing out of control or becoming a black box.

The key insight from Paperclip: treat your AI agents like employees, not chatbots.
- Each agent has a **role description** (job title + responsibilities)
- Each agent has a **monthly budget** (prevents runaway costs)
- Each agent has a **heartbeat** (scheduled wake-up or event-triggered)
- Every action is **logged immutably** (full audit trail)
- Humans are the **board**: they can override, pause, or fire any agent at any time

We don't embed Paperclip as a dependency. We implement these patterns ourselves in our Next.js/TypeScript stack, tailored to the specific needs of property management.

---

## Patterns We Adopt

### 1. Agent Role Definition

Every background agent in Authaver has a structured role definition:

```typescript
interface AgentRole {
  id: string
  name: string                    // Human-readable name (e.g. "Paula")
  role: string                    // Job title (e.g. "Posteingang-Assistentin")
  description: string             // What this agent does
  trigger: AgentTrigger           // How it wakes up
  monthlyBudgetTokens: number     // Max tokens this agent may consume per month
  escalationTarget: 'owner' | AgentId  // Who gets alerted on uncertainty
  confidenceThreshold: number     // Below this → escalate to human
}

type AgentTrigger =
  | { type: 'heartbeat'; cron: string }   // e.g. "0 7 * * *" = daily at 07:00
  | { type: 'event'; eventType: string }  // e.g. "email.received"
  | { type: 'manual' }
```

**Our agents:**

| Name   | Role                   | Trigger              | Heartbeat       |
|--------|------------------------|----------------------|-----------------|
| Paula  | Posteingang-Assistentin | Event: email.received | Every 15 min   |
| Rex    | Rechnungsprüfer         | Event: document.invoice | On arrival    |
| Max    | Mahnwesen               | Heartbeat            | Daily 07:00     |
| Diana  | Dokumenten-Archivarin   | Event: document.any  | On arrival      |
| Zara   | Zahlungsmonitor         | Heartbeat            | Daily 06:00     |

---

### 2. Org Chart — Orchestrator + Specialists

Paperclip uses an org chart to define agent hierarchies. We do the same.

```
Owner (Board Level)
  └── Paula (Orchestrator)
        ├── Rex (Invoice Specialist)
        ├── Max (Payment / Dunning Specialist)
        ├── Diana (Document Specialist)
        └── Zara (Payment Monitor)
```

**Rules:**
- Paula is the single point of contact between the owner and all specialist agents
- Specialist agents escalate to Paula, not directly to the owner
- Paula escalates to the owner only when confidence is below threshold or budget is exceeded
- The owner can bypass this hierarchy at any time (board override)

**Implementation:** Each agent call includes the orchestrator agent's context. Paula maintains a "current priority state" that all specialist agents read from.

---

### 3. Monthly Budget Control

Each agent has a monthly token budget. When it reaches 100%, the agent pauses automatically and notifies the owner.

```typescript
interface AgentBudget {
  agentId: string
  monthlyLimitTokens: number
  consumedTokens: number
  resetDate: Date               // First of each month
  onBudgetExhausted: 'pause' | 'alert' | 'downgrade-model'
}
```

**Budget strategy:**
- High-frequency agents (Paula, Diana): generous budget — they're the first line
- Low-frequency agents (Max, Zara): small budget — they do one specific thing
- Owner sees real-time budget consumption per agent in the dashboard
- Budget resets on the 1st of each month

**Why this matters:** Without budgets, a single broken loop could exhaust API credits in hours. This is a critical guard.

---

### 4. Heartbeat Scheduling

Agents with heartbeat triggers use cron schedules. Event-triggered agents wake up when their event fires.

```typescript
// Heartbeat agent (Max - Dunning)
{
  trigger: { type: 'heartbeat', cron: '0 7 * * *' },
  // Wakes at 07:00 every day, checks for overdue rent, generates drafts, sleeps
}

// Event-triggered agent (Paula - Inbox)
{
  trigger: { type: 'event', eventType: 'email.received' },
  // Wakes when new email arrives via IMAP, classifies, routes, sleeps
}
```

**Implementation:** We use a lightweight job queue (BullMQ or similar) for heartbeat scheduling. Event triggers are fired by the IMAP polling service.

---

### 5. Immutable Audit Trail

Every agent action is logged immutably. No action is "silent". This is non-negotiable for property management (legal compliance, disputes, Nebenkostenabrechnung).

```typescript
interface AgentActionLog {
  id: string
  timestamp: Date
  agentId: string
  actionType: string              // e.g. "email.classify", "invoice.flag", "draft.create"
  inputSummary: string            // What the agent saw
  outputSummary: string           // What the agent decided/produced
  confidenceScore: number         // 0–1
  tokensConsumed: number
  escalated: boolean
  humanDecision?: 'approved' | 'rejected' | 'modified'
  humanDecisionAt?: Date
}
```

The audit log is append-only. Agents cannot modify past entries. The owner can export the full log at any time.

---

### 6. Human Override — Board Level

The owner is always the board. They can:
- Pause any agent instantly
- Override any pending decision
- Inspect any action in the audit trail
- Adjust budgets and heartbeat schedules
- Permanently disable an agent

```typescript
// Any pending agent action can be overridden
interface PendingAction {
  id: string
  agentId: string
  actionType: string
  draft: ActionDraft             // What the agent wants to do
  status: 'pending' | 'approved' | 'rejected' | 'expired'
  expiresAt: Date                // Auto-expires if owner doesn't respond in N hours
  escalationReason?: string
}
```

**Default behavior:** High-confidence routine actions execute automatically. Low-confidence or high-stakes actions wait for human approval. The confidence threshold is configurable per agent.

---

### 7. Goal Alignment

Paperclip ensures every agent task traces back to the company mission. In Authaver:

```typescript
interface AgentContext {
  property: PropertyContext
  currentPriorities: Priority[]   // From Paula's latest orchestration run
  ownerPreferences: OwnerConfig   // Risk tolerance, automation level, etc.
  legalContext: RegionConfig       // German state-specific rules
}
```

Every agent receives this context before acting. This prevents "helpful but wrong" actions like a dunning agent sending a harsh reminder to a tenant the owner is in active negotiation with.

---

## What We Do Differently from Paperclip

| Paperclip | Authaver |
|---|---|
| General-purpose org management | Domain-specific: property management |
| Agents can hire sub-agents | Fixed org chart (simpler for end users) |
| Multi-company support | Multi-property support |
| Agents communicate via structured messages | Agents share a common database context |
| Built for internal company use | Built to be embedded in a user-facing product |
| Node.js standalone process | Integrated into Next.js app via background jobs |
| Provider-agnostic | Provider-agnostic (Claude, OpenAI, Ollama) |

---

## Implementation Checklist

When implementing a new agent, ensure:

- [ ] Role definition created (`AgentRole` interface)
- [ ] Trigger configured (heartbeat or event)
- [ ] Monthly budget set and enforced
- [ ] All actions written to `AgentActionLog`
- [ ] Confidence threshold defined → escalation path clear
- [ ] Owner-facing description written (shown in dashboard)
- [ ] Pause/resume functionality implemented
- [ ] Integration test with mock LLM provider

---

## References

- [Paperclip](https://paperclip.ing) — the source of inspiration
- [BullMQ](https://docs.bullmq.io) — job queue for heartbeat scheduling
- [Anthropic API](https://docs.anthropic.com) — primary AI provider
- [imapflow](https://imapflow.com) — IMAP client for email ingestion
