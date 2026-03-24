# BC Extension Pack

> Domain skills for building Copilot Studio agents that integrate with Dynamics 365 Business Central via the MCP Server.

---

## Overview

The BC Extension Pack provides five skills that cover the full lifecycle of building a BC-connected Copilot Studio agent:

| Skill | Purpose |
|-------|---------|
| `bc-mcp-setup` | Configure BC MCP Server and connect to Copilot Studio |
| `bc-instructions-patterns` | Composable instruction blocks optimized for BC agents |
| `bc-action-templates` | Pre-built TaskDialog YAML for common BC operations |
| `bc-topic-patterns` | Topic patterns with orchestrator vs. custom topic decision guidance |
| `bc-agent-blueprints` | Complete agent specifications for common BC scenarios |

---

## Prerequisites

- **Business Central 27+** (2025 Wave 2 or later)
- MCP Server feature enabled in Feature Management
- `MCP-ADMIN` permission set assigned to the connecting user
- Copilot Studio license with Copilot Credits

---

## Quick Start

### 1. Set Up MCP Connection

```
@copilot-studio-author /copilot-studio:bc-mcp-setup
```

The skill walks through: enabling the feature in BC, configuring API page exposure, connecting in Copilot Studio, and validating with test queries.

### 2. Choose a Blueprint

```
@copilot-studio-author /copilot-studio:bc-agent-blueprints
```

Three blueprints are available:

- **Collections Agent** — Overdue payment management with dispute-aware protection rules
- **Sales Assistant** — Customer lookup, quotes, orders, item availability
- **Customer Support** — Incident creation, order tracking, human escalation

### 3. Add Instructions

```
@copilot-studio-author /copilot-studio:bc-instructions-patterns
```

Pre-composed instruction blocks for collections, sales, and support agents. Mix and match blocks for custom scenarios.

### 4. Add Actions

```
@copilot-studio-author /copilot-studio:bc-action-templates
```

Six ready-to-use action templates:

| Template | Description |
|----------|-------------|
| `customer-balance.mcs.yml` | Query customer balance and credit limit |
| `overdue-invoices.mcs.yml` | List overdue invoices with aging bands |
| `create-payment.mcs.yml` | Register a payment in the journal |
| `sales-orders.mcs.yml` | Query sales orders by customer |
| `create-incident.mcs.yml` | Create a support incident |
| `item-availability.mcs.yml` | Check item stock and availability |

### 5. Add Topics (Optional)

```
@copilot-studio-author /copilot-studio:bc-topic-patterns
```

Custom topics for flows that need deterministic logic (Adaptive Cards, explicit branching). Use the decision matrix in the skill to determine whether the orchestrator or a custom topic is appropriate.

---

## MCP Tool Modes

The BC MCP Server supports two modes:

### Dynamic Tool Mode
The agent discovers BC tools at runtime. Simpler setup, but the agent must be given clear instructions about which tools to use and when.

### Explicit Tool Mode
The agent maker selects specific BC tools at design time. More control, and the selected tools appear directly in the agent's action list.

See `skills/bc-mcp-setup/mcp-config-guide.md` for detailed configuration.

---

## Protection Rules

All BC Extension Pack blueprints enforce protection rules:

- **No external communications** when disputes or ledger inconsistencies exist
- **Human confirmation** before financial operations (payments, journal postings)
- **Full traceability** — every action logged with timestamp, user, and result

These rules are implemented at both the instruction level and topic level (double protection pattern).

---

## Further Reading

- [BC MCP Server Documentation](https://learn.microsoft.com/en-us/dynamics365/business-central/dev-itpro/ai/configure-mcp-server)
- [BC Agent SDK Overview](https://learn.microsoft.com/en-us/dynamics365/business-central/dev-itpro/ai/ai-development-toolkit-overview)
- [CIRCE Framework README](../README.md)
