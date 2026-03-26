# Blueprint: Sales Assistant Agent (Ventas)

> Agente de asistencia comercial para consultas de pedidos, disponibilidad de producto, presupuestos y salud de cuentas. Integrado con Business Central (MCP).

## Overview

| Property | Value |
|----------|-------|
| **Agent Type** | Sales / Ventas |
| **BC Integration** | MCP Server (Dynamic Tool Mode OFF recommended) |
| **Connectors** | Dynamics 365 Business Central MCP |
| **Custom Topics** | 0–1 (order status with AdaptiveCard, optional) |
| **MCP Actions** | 1 cloud-pulled (covers all BC tools; optionally 3–4 specialized templates for routing) |
| **Instruction Blocks** | 1 (BC Data Context) + 2 (Data Formatting) + 3 (Protection) + 4 (Traceability) + 6 (Sales-Specific) |
| **Language** | Spanish (1034) |

---

## Phase 1 — Scaffold the Agent

**Skill**: `/copilot-studio:scaffold-agent`

**Inputs**:
| Parameter | Recommended Value |
|-----------|-------------------|
| Display Name | `BC Sales Assistant` (or user-chosen name) |
| Language | `1034` (Spanish) |
| Description | `Asistente comercial para gestión de pedidos, disponibilidad y cuentas` |
| Schema Name Prefix | User-provided |
| Generative Actions | `true` |

**Conversation Starters** (Spanish):
```yaml
conversationStarters:
  - title: Consultar pedidos del cliente X
    text: ¿Cuáles son los pedidos abiertos del cliente X?
  - title: Disponibilidad de producto
    text: ¿Hay stock disponible del artículo X?
  - title: Estado de cuenta del cliente
    text: ¿Cuál es el estado de cuenta del cliente X?
  - title: Resumen comercial
    text: Dame un resumen de la actividad comercial reciente
```

**Output**: `agent.mcs.yml`, `settings.mcs.yml`, system topics

---

## Phase 2 — MCP Setup (User-Guided)

**Skill**: `/copilot-studio:bc-mcp-setup`

Walk the user through:

1. **Prerequisites**: BC version 27+, MCP-ADMIN permission, Copilot Studio license
2. **Enable MCP** in BC Feature Management (page 2610)
3. **Create MCP configuration** in BC (page 8351):
   - Name: `SALES-ASSISTANT` (or matching agent purpose)
   - Dynamic Tool Mode: **OFF** (recommended for controlled tool set)
   - Unblock Edit Tools: **OFF** (sales assistant is read-only by default; enable only if quote creation is needed)
4. **Add connection** in Copilot Studio portal
5. **Pull locally**

**Output**: `connectionreferences.mcs.yml`, `actions/Dynamics365BusinessCentral-*.mcs.yml`

### Outlook MCP (Optional)

Only needed if the agent should send order confirmations or meeting invites to customers. Most sales assistants are internal-facing and don't need Outlook.

---

## Phase 3 — Apply Instructions

**Skill**: `/copilot-studio:bc-instructions-patterns`  
**Instruction set**: `sales`  
**Pre-composed file**: `sales-agent-instructions.md`

The sales instruction set includes blocks:

| Block | Content |
|-------|---------|
| 1 — BC Data Context | Environment, customer referencing, date/currency, Power Fx |
| 2 — Data Formatting | Amount/date/table formatting |
| 3 — Protection Rules | Dispute/inconsistency safeguards (still relevant — sales agent should warn about blocked customers or credit limit issues) |
| 4 — Traceability | Action logging |
| 6 — Sales-Specific | Quotes, orders, availability, account health |

**Placeholder Substitution**:
| Placeholder | Example Value |
|-------------|---------------|
| `{ENVIRONMENT}` | `SANDBOX_US` |
| `{COMPANY}` | `CRONUS USA, Inc.` |
| `{CONFIG_NAME}` | `SALES-ASSISTANT` |
| `{AGENT_NAME}` | `BC Sales Assistant` |
| `{AGENT_PURPOSE}` | `asistencia comercial y gestión de pedidos` |
| `{SCOPE_DESCRIPTION}` | `pedidos de venta, presupuestos, disponibilidad de producto, estado de cuentas y actividad comercial` |

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
| 1 | `sales-orders` | Query sales orders by customer, date range, or status — core sales data source |
| 2 | `customer-balance` | Check customer balance, credit limit, and payment history — account health |
| 3 | `item-availability` | Check item stock levels by location — answer availability questions |

### Recommended Actions

| # | Template | Purpose |
|---|----------|---------|
| 4 | `overdue-invoices` | View overdue invoices to warn about payment issues before placing new orders |

### Sales-Specific Model Descriptions

Adjust `modelDescription` values for the sales context:
- `customer-balance`: "Query customer account balance, credit limit usage, and payment history. Use when the user asks about a customer's account status, credit situation, or account health."
- `sales-orders`: "Query sales orders by customer, date range, or status. Use when the user asks about orders, shipments, deliveries, or sales activity."
- `item-availability`: "Check item stock levels and availability by location. Use when the user asks if an item is available, in stock, or can be delivered."

---

## Phase 5 — Add Custom Topics (If Needed)

**Skill**: `/copilot-studio:bc-topic-patterns`

Consult the decision matrix for a sales agent:

| Scenario | Custom Topic? | Rationale |
|----------|---------------|-----------|
| Simple order status | No | Orchestrator + MCP tool sufficient |
| Order status with AdaptiveCard | **Yes** | AdaptiveCard requires custom topic |
| Item availability check | No | Orchestrator + MCP tool sufficient |
| Customer balance query | No | Orchestrator handles with formatting instructions |
| Credit limit warning | No | Instruction-based (Block 6 handles this) |

### Optional Custom Topic: Order Status with Card

**Pattern**: `order-inquiry` (from `/copilot-studio:bc-topic-patterns`)

If the user wants a formatted order status card:
1. Customer identification via `AutomaticTaskInput`
2. Sales order query via MCP action
3. AdaptiveCard displaying order details (order number, status, dates, amounts)
4. Status-based messaging (shipped → tracking info, pending → expected date)

For most sales agents, the orchestrator handles order queries well enough with instruction-based formatting. Only create a custom topic if the user specifically wants AdaptiveCard-formatted responses.

---

## Phase 6 — Add Knowledge (Optional)

**Skill**: `/copilot-studio:add-knowledge`

Recommended knowledge sources for a sales agent:

| Source | Type | Purpose |
|--------|------|---------|
| Product catalog | SharePoint / Public URL | Item descriptions, specifications, pricing tiers |
| Sales policies | SharePoint | Discount rules, credit approval thresholds, return policies |
| FAQ — sales processes | SharePoint / Public URL | Common questions about ordering, delivery, invoicing |

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
- Protection rules in instructions (even sales agents must check for blocked customers)
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
| 1 | "¿Cuáles son los pedidos abiertos del cliente 10000?" | Returns list of open sales orders |
| 2 | "¿Hay stock del artículo 1000?" | Returns availability by location |
| 3 | "¿Cuál es el estado de cuenta del cliente 10000?" | Returns balance, credit limit, and payment status |
| 4 | "¿El cliente 10000 tiene facturas vencidas?" | Returns overdue invoices if any |
| 5 | "Dame un resumen de la actividad comercial de esta semana" | Summarizes recent orders and invoices |

---

## Component Summary

| Component | Count | Source Skill |
|-----------|-------|-------------|
| Agent files | 2 | `scaffold-agent` |
| Connection references | 1 | `bc-mcp-setup` (portal) |
| MCP actions | 3–4 | `bc-action-templates` |
| Custom topics | 0–1 | `bc-topic-patterns` |
| System topics | 8–10 | `scaffold-agent` |
| Knowledge sources | 0–3 | `add-knowledge` |
| **Total files** | **14–18** | |
