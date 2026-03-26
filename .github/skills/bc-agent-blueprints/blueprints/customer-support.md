# Blueprint: Customer Support Agent (Soporte al Cliente)

> Agente de soporte para gestión de incidencias, consultas de estado de pedidos, onboarding de clientes y resolución de problemas. Integrado con Business Central (MCP) y opcionalmente Outlook (MCP).

## Overview

| Property | Value |
|----------|-------|
| **Agent Type** | Support / Soporte |
| **BC Integration** | MCP Server (Dynamic Tool Mode OFF recommended) |
| **Connectors** | Dynamics 365 Business Central MCP, Microsoft Outlook Mail MCP (optional) |
| **Custom Topics** | 2–3 (incident creation with card, customer onboarding with card) |
| **MCP Actions** | 1 cloud-pulled (covers all BC tools; optionally 3–5 specialized templates for routing) |
| **Instruction Blocks** | 1 (BC Data Context) + 2 (Data Formatting) + 3 (Protection) + 4 (Traceability) + 7 (Support-Specific) |
| **Language** | Spanish (1034) |

---

## Phase 1 — Scaffold the Agent

**Skill**: `/copilot-studio:scaffold-agent`

**Inputs**:
| Parameter | Recommended Value |
|-----------|-------------------|
| Display Name | `BC Customer Support` (or user-chosen name) |
| Language | `1034` (Spanish) |
| Description | `Agente de soporte al cliente — incidencias, estado de pedidos y onboarding` |
| Schema Name Prefix | User-provided |
| Generative Actions | `true` |

**Conversation Starters** (Spanish):
```yaml
conversationStarters:
  - title: Crear incidencia
    text: Necesito crear una incidencia para el cliente X
  - title: Estado de pedido
    text: ¿Cuál es el estado del pedido del cliente X?
  - title: Información del cliente
    text: Dame la información del cliente X
  - title: Ayuda general
    text: ¿En qué puedo ayudarte hoy?
```

**Output**: `agent.mcs.yml`, `settings.mcs.yml`, system topics

---

## Phase 2 — MCP Setup (User-Guided)

**Skill**: `/copilot-studio:bc-mcp-setup`

Walk the user through:

1. **Prerequisites**: BC version 27+, MCP-ADMIN permission, Copilot Studio license
2. **Enable MCP** in BC Feature Management (page 2610)
3. **Create MCP configuration** in BC (page 8351):
   - Name: `CUSTOMER-SUPPORT` (or matching agent purpose)
   - Dynamic Tool Mode: **OFF** (recommended — controlled tool set)
   - Unblock Edit Tools: **ON** (for incident creation)
4. **Add connection** in Copilot Studio portal
5. **Pull locally**

**Output**: `connectionreferences.mcs.yml`, `actions/Dynamics365BusinessCentral-*.mcs.yml`

### Outlook MCP (Optional)

Useful if the agent needs to:
- Send case confirmation emails to customers
- Notify internal teams about escalated incidents
- Schedule follow-up meetings for unresolved issues

---

## Phase 3 — Apply Instructions

**Skill**: `/copilot-studio:bc-instructions-patterns`  
**Instruction set**: `support`  
**Pre-composed file**: `support-agent-instructions.md`

The support instruction set includes blocks:

| Block | Content |
|-------|---------|
| 1 — BC Data Context | Environment, customer referencing, date/currency, Power Fx |
| 2 — Data Formatting | Amount/date/table formatting |
| 3 — Protection Rules | Dispute/inconsistency safeguards (support agent must not escalate externally if disputes exist) |
| 4 — Traceability | Action logging, incident registration |
| 7 — Support-Specific | Incidents, escalation triggers, order status, customer info |

**Placeholder Substitution**:
| Placeholder | Example Value |
|-------------|---------------|
| `{ENVIRONMENT}` | `SANDBOX_US` |
| `{COMPANY}` | `CRONUS USA, Inc.` |
| `{CONFIG_NAME}` | `CUSTOMER-SUPPORT` |
| `{AGENT_NAME}` | `BC Customer Support` |
| `{AGENT_PURPOSE}` | `soporte al cliente, gestión de incidencias y consultas` |
| `{SCOPE_DESCRIPTION}` | `incidencias, estado de pedidos, información de cliente, onboarding y resolución de problemas` |

**Apply**: Use `/copilot-studio:edit-agent` to set the `instructions` field.

---

## Phase 4 — MCP Action Configuration

**Skill**: `/copilot-studio:bc-action-templates` (optional — see note below)

> **IMPORTANT — MCP Architecture**: In most cases, the single cloud-pulled MCP action (`Dynamics365BusinessCentral-Dynamics365BusinessCentralMCPPreview.mcs.yml`) already gives the orchestrator access to ALL BC tools. You do NOT need to create separate action files per operation. Only create additional action files if you need different `modelDescription` values to improve orchestrator routing.
>
> Environment config (`bcenvironment`, `company`, `configurationName`) is stored as `ManualTaskInput` entries in the action YAML. Copy these values from the cloud-pulled MCP action when creating additional action files. They can also be set via Copilot Studio UI → Tools → Inputs tab (synced on pull/push).

If the orchestrator routes poorly with a single MCP action, consider adding specialized templates:

### Required Actions

| # | Template | Purpose |
|---|----------|---------|
| 1 | `customer-balance` | Check customer account status — needed for customer info queries |
| 2 | `sales-orders` | Query order status — core support query |
| 3 | `create-incident` | Create a service incident/case — the primary support action |

### Recommended Actions

| # | Template | Purpose |
|---|----------|---------|
| 4 | `item-availability` | Check product availability for replacement or re-order scenarios |
| 5 | `overdue-invoices` | View payment status — common support query ("¿por qué no me envían el pedido?") |

### Support-Specific Model Descriptions

Adjust `modelDescription` values for the support context:
- `customer-balance`: "Query customer account information, balance, and status. Use when the user asks about a customer's account, contact details, or profile."
- `sales-orders`: "Query sales orders and shipment status. Use when the user asks about order status, delivery tracking, or shipment details."
- `create-incident`: "Create a service incident or support case for a customer. Use when the user wants to register a complaint, issue, problem, or service request."
- `item-availability`: "Check product availability. Use when the user asks about replacements, re-orders, or product stock for resolving a support issue."

---

## Phase 5 — Add Custom Topics

**Skill**: `/copilot-studio:bc-topic-patterns`

Support agents typically need more custom topics than other agent types because of AdaptiveCard-based forms and multi-step flows.

| Scenario | Custom Topic? | Rationale |
|----------|---------------|-----------|
| Simple order status | No | Orchestrator + MCP tool sufficient |
| Order status with card | **Yes** | AdaptiveCard requires custom topic |
| Incident creation with card | **Yes** | AdaptiveCard form for structured input |
| Customer onboarding with card | **Yes** | Welcome flow with profile card |
| Simple customer info | No | Orchestrator sufficient |
| Item availability | No | Orchestrator + MCP tool sufficient |

### Recommended Custom Topic 1: Incident Creation

**Pattern**: `incident-management` (from `/copilot-studio:bc-topic-patterns`)

This topic implements:
1. Customer identification via `AutomaticTaskInput`
2. Incident details collection (category, description, priority) via AdaptiveCard form
3. Confirmation card showing incident summary
4. Create incident via MCP action
5. Success message with incident number
6. Traceability registration

### Recommended Custom Topic 2: Customer Onboarding

**Pattern**: `customer-onboarding` (from `/copilot-studio:bc-topic-patterns`)

This topic implements:
1. Customer identification via `AutomaticTaskInput`
2. Customer data retrieval via MCP action
3. Profile card displaying customer info (name, balance, credit limit, contact)
4. Welcome message with available services
5. Guided next steps (check orders, create incident, account overview)

### Optional Custom Topic 3: Order Status with Card

**Pattern**: `order-inquiry` (from `/copilot-studio:bc-topic-patterns`)

Only if the user wants formatted order cards instead of text responses.

---

## Phase 6 — Add Knowledge (Optional)

**Skill**: `/copilot-studio:add-knowledge`

Recommended knowledge sources for a support agent:

| Source | Type | Purpose |
|--------|------|---------|
| Support procedures | SharePoint / Public URL | Escalation paths, SLA definitions, resolution workflows |
| Product documentation | SharePoint | Common product issues, troubleshooting guides |
| FAQ — customer questions | SharePoint / Public URL | Common customer questions about orders, invoices, returns |
| Return & warranty policy | SharePoint | Return windows, warranty terms, RMA procedures |

Knowledge sources are especially valuable for support agents — they allow the agent to answer common questions without querying BC.

---

## Phase 7 — Validate

**Skill**: `/copilot-studio:validate`

```bash
node .github/scripts/schema-lookup.bundle.js validate agent.mcs.yml
node .github/scripts/schema-lookup.bundle.js validate settings.mcs.yml
node .github/scripts/schema-lookup.bundle.js validate actions/*.mcs.yml
node .github/scripts/schema-lookup.bundle.js validate topics/*.mcs.yml
```

Check for:
- Schema validation passes for all files
- Protection rules in instructions
- AdaptiveCard JSON valid in custom topics
- All `_REPLACE` placeholders replaced
- All environment values match across action files (`bcenvironment`, `company`, `configurationName` ManualTaskInput entries)
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
| 1 | "Necesito crear una incidencia para el cliente 10000" | Shows incident form card, creates incident in BC |
| 2 | "¿Cuál es el estado del pedido del cliente 10000?" | Returns order status (card or text) |
| 3 | "Dame la información del cliente 10000" | Returns customer profile with balance and contact info |
| 4 | "¿Hay stock del artículo 1000?" | Returns availability by location |
| 5 | "El cliente 10000 tiene un problema con su último pedido" | Guides through incident creation |
| 6 | "¿Por qué no se envía el pedido del cliente 10000?" | Checks order status and payment status for issues |

---

## Component Summary

| Component | Count | Source Skill |
|-----------|-------|-------------|
| Agent files | 2 | `scaffold-agent` |
| Connection references | 1–2 | `bc-mcp-setup` (portal) |
| MCP actions | 3–5 | `bc-action-templates` |
| Custom topics | 2–3 | `bc-topic-patterns` |
| System topics | 8–10 | `scaffold-agent` |
| Knowledge sources | 0–4 | `add-knowledge` |
| **Total files** | **16–24** | |
