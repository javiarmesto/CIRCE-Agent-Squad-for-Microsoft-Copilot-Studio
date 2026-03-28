# Agent Requirements: Order Status Tracker

> This document defines the requirements for the Order Status Tracker agent.
> It serves as the initial context for the CIRCE framework before any development begins.
> The Conductor agent should read this document first and use it as reference throughout the build workflow.

---

## Agent Identity

| Property | Value |
|----------|-------|
| Name | Order Tracker |
| Purpose | Query sales order status from Business Central |
| Language | English |
| Audience | Internal sales and customer service teams |
| Tone | Professional, concise, helpful |

## Data Source

| Property | Value |
|----------|-------|
| System | Dynamics 365 Business Central |
| Connection | MCP Server (Explicit Tool Mode, read-only) |
| Environment | YOUR_BC_ENVIRONMENT |
| Company | CRONUS USA, Inc. |
| MCP Configuration | ORDER-TRACKER |

### BC Entities Required

| Entity | API Page | Operations |
|--------|----------|------------|
| Sales Orders | APIV2 - Sales Orders (30023) | Read |
| Sales Order Lines | APIV2 - Sales Order Lines | Read |
| Customers | APIV2 - Customers (30009) | Read |

### Key Fields

**Sales Order:** No., Sell-to Customer Name, Order Date, Status (Open/Released/Pending Approval/Pending Prepayment), Amount Including VAT, Requested Delivery Date, Shipment Date

**Customer:** No., Name, Balance (LCY), Credit Limit

---

## Functional Requirements

### FR-1: Lookup Order by Number

The user provides a sales order number. The agent queries BC and returns:
- Order number, customer name, order date
- Status with visual indicator (color-coded in Adaptive Card)
- Total amount, requested delivery date, shipment date
- If not found: friendly message suggesting to verify the number

### FR-2: Search Orders by Customer

The user provides a customer name (partial match accepted). The agent queries BC and returns:
- Up to 5 most recent orders for that customer
- For each: order number, date, status, total amount
- If multiple customers match: list customers and ask to pick
- If no orders found: inform the user

### FR-3: Guided Input

When the user says "check my order" or "order status" without providing a reference:
- Ask whether they want to search by order number or customer name
- Guide them through the input

### FR-4: Adaptive Card Display

When an order is found, display an Adaptive Card with:
- Header: order number + customer name
- FactSet: date, status, amount, delivery date
- Status color coding: Open → blue accent, Released → green accent, Pending → amber accent

### FR-5: Error Handling

- BC connection failure: "I'm having trouble connecting to Business Central. Please try again in a moment."
- No results: "I couldn't find that order. Please check the order number and try again."
- Multiple results: display a numbered list and let the user pick

---

## Non-Functional Requirements

| Requirement | Target |
|-------------|--------|
| Response time | < 5 seconds for single order lookup |
| Language | English (all user-facing content) |
| Read-only | No write operations to BC |
| Authentication | Invoker mode (user's BC credentials) |

---

## Topics

| Topic | Trigger | Type |
|-------|---------|------|
| Check Order Status | "order status", "track order", "where is my order", "check order", "find order", "lookup order" | OnRecognizedIntent + Adaptive Card |
| Greeting | Conversation start | OnConversationStart |
| Fallback | Unknown intent | OnUnknownIntent |
| Error Handler | System error | OnError |
| Escalate | "talk to a person", "human agent" | OnEscalate |

## Actions

| Action | Type | Input | Purpose |
|--------|------|-------|---------|
| Get Sales Order | MCP (BC) | Order number or customer filter | Query sales orders from BC |

## Out of Scope

- Order creation or modification (read-only agent)
- Invoice or payment queries
- Customer master data management
- Multi-language support (English only for this version)
- Knowledge sources (no document search needed)

---

## Success Criteria

The agent is considered complete when:

1. All 5 test utterances return correct responses
2. Adaptive Card displays with real CRONUS data
3. Error handling works without crashing
4. Response times are under 5 seconds
5. The agent deploys to Copilot Studio and responds correctly to all test utterances

---

## Pre-condition: Agent Creation in Copilot Studio

Before starting the CIRCE workflow, the agent must exist in Copilot Studio as a blank shell:

1. **Create the agent manually** in Copilot Studio UI with only the name "Order Tracker" — no topics, no actions, no configuration. Just the name.
2. This gives the agent an identity in the environment (schema name, environment URL, agent ID) that CIRCE needs to clone and push back to.

This is a one-time prerequisite. Everything else happens from code.

## CIRCE Framework Usage

This agent should be built using the CIRCE orchestrated workflow:

1. **Manage + clone-agent** clones the blank "Order Tracker" from Copilot Studio to the local workspace
2. **Author** loads this requirements document as agent context and begins building
3. **Author + bc-mcp-setup** configures the BC MCP connection
4. **Author + bc-action-templates** creates the actions
5. **Author + bc-topic-patterns + add-adaptive-card** creates the main topic with Adaptive Card
6. **Conductor + pre-push-review** validates everything with HITL gate
7. **Manage + manage-agent** pushes to Copilot Studio (updates the existing agent)
8. User publishes the draft manually in Copilot Studio UI
9. **Test + chat-with-agent** runs the 5 test utterances against the published agent

Each step must produce skills evidencing, update circe-memory.md, and generate a decision record where a design choice is made.

---

## Test Utterances

| # | Utterance | Expected Behavior |
|---|-----------|-------------------|
| 1 | "What's the status of order S-ORD101001?" | Returns order details in Adaptive Card |
| 2 | "Check orders for Adatum Corporation" | Returns list of recent orders |
| 3 | "Where is my order?" | Asks for order number or customer name |
| 4 | "Order status for XXXXX" | Handles not-found gracefully |
| 5 | "Track order" | Triggers the correct topic |

---

## Document History

| Version | Date | Author | Notes |
|---------|------|--------|-------|
| 0.1 | 2026-03-25 | Javier Armesto | Initial requirements |
| 0.2 | 2026-03-25 | Javier Armesto | Updated: clone from blank agent instead of scaffold |
| 1.0 | 2026-03-26 | Javier Armesto | Final version for CIRCE launch |
