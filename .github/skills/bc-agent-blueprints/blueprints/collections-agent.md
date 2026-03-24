# Blueprint: Collections Agent (Cobros)

> Agente especializado en gestión de cobros, recuperación de pagos, análisis de aging y priorización de cartera. Integrado con Business Central (MCP) y Outlook (MCP).

## Overview

| Property | Value |
|----------|-------|
| **Agent Type** | Collections / Cobros |
| **BC Integration** | MCP Server (Dynamic Tool Mode OFF recommended) |
| **Connectors** | Dynamics 365 Business Central MCP, Microsoft Outlook Mail MCP |
| **Custom Topics** | 1–2 (collections reminder with dispute check) |
| **MCP Actions** | 3–4 (customer balance, overdue invoices, create payment, create incident) |
| **Instruction Blocks** | 1 (BC Data Context) + 2 (Data Formatting) + 3 (Protection) + 4 (Traceability) + 5 (Collections-Specific) |
| **Language** | Spanish (1034) |

---

## Phase 1 — Scaffold the Agent

**Skill**: `/copilot-studio:scaffold-agent`

**Inputs**:
| Parameter | Recommended Value |
|-----------|-------------------|
| Display Name | `Circe Collections` (or user-chosen name) |
| Language | `1034` (Spanish) |
| Description | `Agente de gestión de cobros y recuperación de pagos` |
| Schema Name Prefix | User-provided (e.g., `copilots_header_<id>_<Name>`) |
| Generative Actions | `true` |

**Conversation Starters** (Spanish):
```yaml
conversationStarters:
  - title: Envía recordatorio de pago al cliente X
    text: Envía recordatorio de pago al cliente X
  - title: Programa reunión de cobros con el cliente X
    text: Programa reunión de cobros con el cliente X
  - title: Analiza la cartera y recomienda acciones
    text: Analiza la cartera y recomienda acciones
  - title: ¿Qué facturas puedo cobrar esta semana?
    text: ¿Qué facturas puedo cobrar esta semana?
```

**Output**: `agent.mcs.yml`, `settings.mcs.yml`, system topics (`ConversationStart`, `Fallback`, `OnError`, etc.)

---

## Phase 2 — MCP Setup (User-Guided)

**Skill**: `/copilot-studio:bc-mcp-setup`

This phase requires portal access — it cannot be automated. Walk the user through:

1. **Prerequisites**: BC version 27+, MCP-ADMIN permission, Copilot Studio license
2. **Enable MCP** in BC Feature Management (page 2610)
3. **Create MCP configuration** in BC (page 8351):
   - Name: `COLLECTIONS` (or matching agent purpose)
   - Dynamic Tool Mode: **OFF** (recommended for production collections agent — controlled tool set)
   - Unblock Edit Tools: **ON** (for payment creation)
4. **Add connection** in Copilot Studio portal
5. **Pull locally** to get the connection reference and action files

**Output**: `connectionreferences.mcs.yml`, `actions/Dynamics365BusinessCentral-*.mcs.yml`

### Outlook MCP (Optional but Recommended)

If the agent needs to send payment reminder emails or schedule meetings:
1. Add Microsoft Outlook Mail MCP connection in Copilot Studio
2. Pull locally to get `actions/MicrosoftMCPServers-MicrosoftOutlookMailMCP.mcs.yml`

---

## Phase 3 — Apply Instructions

**Skill**: `/copilot-studio:bc-instructions-patterns`  
**Instruction set**: `collections`  
**Pre-composed file**: `collections-agent-instructions.md`

The collections instruction set includes blocks:

| Block | Content |
|-------|---------|
| 1 — BC Data Context | Environment, customer referencing, date/currency, Power Fx |
| 2 — Data Formatting | Amount/date/table formatting, aging buckets |
| 3 — Protection Rules | Dispute/inconsistency safeguards, write confirmations |
| 4 — Traceability | Action logging, timestamp/user/result reporting |
| 5 — Collections-Specific | Aging-based workflow, escalation ladder, payment comms |

**Placeholder Substitution**:
| Placeholder | Example Value |
|-------------|---------------|
| `{ENVIRONMENT}` | `SANDBOX_US` |
| `{COMPANY}` | `CRONUS USA, Inc.` |
| `{CONFIG_NAME}` | `COLLECTIONS` |
| `{AGENT_NAME}` | `Circe Collections` |
| `{AGENT_PURPOSE}` | `gestión de cobros y recuperación de pagos` |
| `{SCOPE_DESCRIPTION}` | `cobros, facturas, cashflow, disputas, priorización de cartera y análisis de aging` |

**Apply**: Use `/copilot-studio:edit-agent` to set the `instructions` field in `agent.mcs.yml`.

---

## Phase 4 — Add MCP Actions

**Skill**: `/copilot-studio:bc-action-templates`

Add these action templates in order:

### Required Actions

| # | Template | Purpose |
|---|----------|---------|
| 1 | `customer-balance` | Query customer ledger entries and current balance — the fundamental data source for collections |
| 2 | `overdue-invoices` | List overdue invoices with aging bucket classification — drives aging analysis and prioritization |
| 3 | `create-payment` | Register a payment journal entry — enables recording received payments |

### Recommended Actions

| # | Template | Purpose |
|---|----------|---------|
| 4 | `create-incident` | Create a service incident for escalation cases (>90d overdue, repeated non-payment) |

### Outlook Actions (If Outlook MCP Connected)

The Outlook MCP action (`MicrosoftMCPServers-MicrosoftOutlookMailMCP.mcs.yml`) is pulled from the portal. Configure it with `/copilot-studio:configure-mcp-action` to optimize:
- `modelDescription`: "Send professional payment reminder emails to customers. Use when the user asks to send a reminder, follow-up, or payment request email."
- Ensure protection rules are enforced in instructions (never email if dispute/inconsistency exists)

---

## Phase 5 — Add Custom Topics (If Needed)

**Skill**: `/copilot-studio:bc-topic-patterns`

Consult the decision matrix before creating custom topics. For a collections agent:

| Scenario | Custom Topic? | Rationale |
|----------|---------------|-----------|
| Simple balance query | No | Orchestrator + MCP tool sufficient |
| Aging breakdown | No | Format via instructions |
| Collections with dispute check | **Yes** | Business rule must be deterministic |
| Payment registration | No | MCP action with confirmation via instructions |
| Escalation to incident | No | Orchestrator chains create-incident action |

### Recommended Custom Topic: Collections Reminder

**Pattern**: `collections-flow`  
**Template**: `collections-reminder.topic.mcs.yml`

This topic implements:
1. Customer identification (via `AutomaticTaskInput`)
2. Dispute check (deterministic `ConditionGroup` — NON-NEGOTIABLE)
3. Balance/aging retrieval via MCP action
4. Aging-based branching (>90d escalation, 31–90d reminder, ≤30d gentle follow-up)
5. Email/meeting creation via Outlook MCP
6. Traceability registration in BC

### Optional Sub-Flow: Customer Lookup

**Template**: `customer-lookup.topic.mcs.yml`

Reusable sub-flow that resolves a customer by name or number. Called by the collections reminder topic or available standalone.

---

## Phase 6 — Add Knowledge (Optional)

**Skill**: `/copilot-studio:add-knowledge`

Recommended knowledge sources for a collections agent:

| Source | Type | Purpose |
|--------|------|---------|
| Company collections policy | SharePoint / Public URL | Escalation thresholds, communication templates, legal requirements |
| Payment terms reference | SharePoint | Standard payment terms, discount schedules, credit policies |
| FAQ — collections processes | SharePoint / Public URL | Common questions from internal users about collections workflows |

Knowledge sources ground the agent's generative answers in company-specific policies.

---

## Phase 7 — Validate

**Skill**: `/copilot-studio:validate`

Validate every file created during the blueprint:

```bash
node .github/scripts/schema-lookup.bundle.js validate agent.mcs.yml
node .github/scripts/schema-lookup.bundle.js validate settings.mcs.yml
node .github/scripts/schema-lookup.bundle.js validate actions/*.mcs.yml
node .github/scripts/schema-lookup.bundle.js validate topics/*.mcs.yml
```

Check for:
- Schema validation passes for all files
- Protection rules present in both instructions AND topic ConditionGroups
- All `_REPLACE` placeholders replaced with unique IDs
- All environment values match (`bcenvironment`, `company`, `configurationName`)
- Spanish content in all user-facing text

---

## Phase 8 — Push, Test, Iterate

### Push
**Skill**: `/copilot-studio:pre-push-review` → `/copilot-studio:manage-agent`

### Smoke Test
**Skill**: `/copilot-studio:chat-with-agent`

Recommended test utterances:

| # | Utterance | Expected Behavior |
|---|-----------|-------------------|
| 1 | "¿Cuál es el saldo del cliente 10000?" | Returns balance with aging breakdown |
| 2 | "¿Qué facturas vencidas tiene el cliente 10000?" | Returns overdue invoices list |
| 3 | "Envía un recordatorio de pago al cliente 10000" | Checks for disputes first, then sends email |
| 4 | "Registra un pago de 500€ del cliente 10000" | Creates payment journal entry with confirmation |
| 5 | "Analiza la cartera y recomienda acciones" | Portfolio analysis with prioritized actions |

### Batch Test (Optional)
**Skill**: `/copilot-studio:run-tests`

---

## Component Summary

| Component | Count | Source Skill |
|-----------|-------|-------------|
| Agent files | 2 | `scaffold-agent` |
| Connection references | 1–2 | `bc-mcp-setup` (portal) |
| MCP actions | 3–4 | `bc-action-templates` |
| Custom topics | 1–2 | `bc-topic-patterns` |
| System topics | 8–10 | `scaffold-agent` |
| Knowledge sources | 0–3 | `add-knowledge` |
| **Total files** | **15–21** | |
