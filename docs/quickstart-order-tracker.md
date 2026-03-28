# Quick Start: Order Tracker — Your First CIRCE Agent

> Build a working Business Central agent that queries sales order status via MCP, displays results in an Adaptive Card, and deploys to Copilot Studio — all from code.

---

## What You'll Build

An agent called **Order Tracker** that:
- Queries sales orders from Business Central via MCP Server
- Displays order details in a color-coded Adaptive Card (Open=blue, Released=green, Pending=amber)
- Handles "order not found" gracefully
- Lists multiple orders when searching by customer name

**Time:** ~30 minutes (prerequisites already installed)

---

## Prerequisites

### Software

| Tool | Version | Verify |
|------|---------|--------|
| VS Code | 1.85+ | `code --version` |
| Node.js | 18+ | `node --version` |
| Git | 2.30+ | `git --version` |

### VS Code Extensions

```bash
code --install-extension ms-CopilotStudio.vscode-copilotstudio
code --install-extension GitHub.copilot
code --install-extension GitHub.copilot-chat
```

### Licenses & Access

| Requirement | Where |
|-------------|-------|
| GitHub Copilot plan (Pro/Business/Enterprise) | github.com/features/copilot |
| Copilot Studio license with Credits | admin.powerplatform.microsoft.com |
| Power Platform environment | admin.powerplatform.microsoft.com |
| BC 27+ with MCP Server enabled | BC Feature Management (page 2610) |
| MCP-ADMIN permission set assigned | BC → Users → Permission Sets |

---

## Pre-condition: Create a Blank Agent

Before CIRCE can work, the agent needs an identity in Copilot Studio:

1. Open [copilotstudio.microsoft.com](https://copilotstudio.microsoft.com)
2. Click **Create** → **New agent**
3. Name it **"Order Tracker"**
4. Don't configure anything else — no topics, no actions, no instructions
5. Save

This creates the schema name, agent ID, and environment URL that CIRCE needs to clone and push back to. Everything else happens from code.

---

## Phase 1 — Setup: Clone + Context + MCP

**Agents:** Manage → Author
**Skills:** clone-agent, bc-mcp-setup

### Prompt

```
@copilot-studio-manage Clone the "Order Tracker" agent from Copilot Studio.

@copilot-studio-author

Read the file order-tracker-requirements.md as context for this agent.

Then set up the MCP connection to Business Central:
- Environment: YOUR_BC_ENVIRONMENT
- Company: CRONUS USA, Inc.
- Explicit Tool Mode, read-only
- Expose: Sales Orders, Sales Order Lines, Customers (API v2.0)
- Validate with a test query against Sales Orders
```

### What happens

1. **Manage** clones the blank agent → creates local directory with `agent.mcs.yml`, `settings.mcs.yml`
2. **Author** reads the requirements document → understands what to build
3. **Author** walks you through BC MCP setup (portal steps + local config)
4. MCP connection validated with a test query

### Expected result

- Local directory with cloned agent structure (`agent.mcs.yml`, `settings.mcs.yml`, default topics)
- `connectionreferences.mcs.yml` with BC MCP connection configured
- Test query successful against Sales Orders
- `circe-memory.md` initialized with reference to the requirements document
- DR-001 (clone + context loaded), DR-002 (MCP Explicit Tool Mode, read-only)

### Validation checklist

- [ ] Local directory created with `agent.mcs.yml`
- [ ] `settings.mcs.yml` has correct schema name from Copilot Studio
- [ ] BC MCP connection reference exists in `connectionreferences.mcs.yml`
- [ ] `circe-memory.md` initialized with Agent Configuration populated
- [ ] DR-001 generated (clone + context)
- [ ] DR-002 generated (MCP setup decision)

---

## Phase 2 — Build: Actions + Topic + Adaptive Card

**Agent:** Author
**Skills:** bc-action-templates, bc-topic-patterns, add-adaptive-card

### Prompt

```
@copilot-studio-author

Using the requirements in order-tracker-requirements.md, build the agent:

1. Create action "Get Sales Order by Number"
   - Input: orderNumber (text)
   - Output: orderNo, customerName, orderDate, status, totalAmount, shipmentDate

2. Create action "Search Orders by Customer"
   - Input: customerName (text)
   - Output: list of {orderNo, orderDate, status, totalAmount} (max 5)

3. Create topic "Check Order Status"
   - Triggers: "check my order", "order status", "where is my order",
     "track order", "what's the status of order", "lookup order", "find order"
   - Flow: ask for order number or customer name → call action →
     if found: Adaptive Card with order summary (status color-coded:
     Open=blue, Released=green, Pending=amber) →
     if not found: friendly message + retry →
     if multiple: list and let user pick

4. Ensure greeting, fallback, and error handler topics exist with
   appropriate messages per the requirements doc.
```

### What happens

1. **Author** creates 2 action YAML files in `actions/` — one per operation for better orchestrator routing
2. **Author** creates the main topic with full flow (happy path + error + multiple results)
3. **Author** generates the Adaptive Card with color-coded status FactSet
4. **Author** verifies/creates system topics (greeting, fallback, error handler)
5. Each component gets a CIRCE-EVIDENCE block and decision record

### Expected result

- 2 action YAML files in `actions/`:
  - `BC-GetSalesOrderByNumber.mcs.yml` — single order lookup by number
  - `BC-SearchOrdersByCustomer.mcs.yml` — list orders by customer name
- Main topic in `topics/CheckOrderStatus.topic.mcs.yml` with complete flow
- Adaptive Card embedded in the topic with status color coding
- System topics verified/created (greeting, fallback, error handler)
- DR-003 (actions — 2 separate actions for orchestrator routing accuracy)
- DR-004 (topic + Adaptive Card design)
- Memory updated with all components in "Componentes Creados" table

### Validation checklist

- [ ] 2 action files created and validated (0 schema errors)
- [ ] Each action has distinct `modelDescription` for orchestrator routing
- [ ] Topic file created with 7+ trigger phrases
- [ ] Topic has `modelDescription` describing the order status flow
- [ ] Adaptive Card JSON valid (version 1.5, FactSet with status fields)
- [ ] ConditionGroup handles: found, not-found, multiple results
- [ ] System topics exist (greeting, fallback, error handler)
- [ ] Skills evidencing blocks present for all operations
- [ ] DR-003 and DR-004 generated
- [ ] `circe-memory.md` updated with all new components

---

## Phase 3 — Review, Deploy & Test

**Agents:** Conductor → Manage → Test
**Skills:** pre-push-review, manage-agent, chat-with-agent

### Prompt

```
@copilot-studio-conductor

Run the full review-deploy-test cycle for Order Tracker:

1. Pre-push review: validate all YAML, show diff summary and decision records,
   run coverage report. Wait for my explicit approval before continuing.

2. After approval: push to Copilot Studio as draft.

3. After I confirm publication: test with these utterances:
   - "What's the status of order S-ORD101001?"
   - "Check orders for Adatum Corporation"
   - "Where is my order?"
   - "Order status for XXXXX"
   - "Track order"
```

### What happens

1. **Conductor** runs pre-push review → validates all YAML, presents diff, lists DRs
2. You review and approve → **Manage** pushes to Copilot Studio (draft)
3. You publish manually in Copilot Studio UI
4. You confirm publication → **Test** runs 5 utterances against the published agent

### Expected result

- Pre-push review: 0 validation errors, coherent diff, DR-001 through DR-004 listed
- HITL gate: waits for explicit approval before push
- Push successful, agent visible as draft in Copilot Studio
- Pause for manual publication
- 5/5 utterances resolved correctly:

| # | Utterance | Expected behavior |
|---|-----------|-------------------|
| 1 | "What's the status of order S-ORD101001?" | Returns order details in Adaptive Card with status color |
| 2 | "Check orders for Adatum Corporation" | Returns list of recent orders for customer |
| 3 | "Where is my order?" | Asks for order number or customer name (guided input) |
| 4 | "Order status for XXXXX" | Handles not-found gracefully with friendly message |
| 5 | "Track order" | Triggers the correct topic, asks for details |

### Validation checklist

- [ ] Pre-push review passed (0 validation errors)
- [ ] Decision records DR-001 through DR-004 listed in review summary
- [ ] Coverage report generated (advisory, no blocking)
- [ ] Push completed successfully
- [ ] Agent published in Copilot Studio UI
- [ ] Test 1: Adaptive Card with real CRONUS order data
- [ ] Test 2: Multiple orders listed for customer
- [ ] Test 3: Agent asks for order reference (guided input)
- [ ] Test 4: Friendly not-found message, no crash
- [ ] Test 5: Correct topic triggered
- [ ] Response times under 5 seconds

---

## CIRCE Capabilities Validated

| Capability | Phase |
|------------|-------|
| Clone agent | 1 |
| Context loading (requirements doc) | 1 |
| BC MCP Setup | 1 |
| BC Action Templates | 2 |
| BC Topic Patterns | 2 |
| Adaptive Cards | 2 |
| Skills evidencing | 1, 2, 3 |
| Cross-session memory | 1, 2, 3 |
| Decision records | 1, 2, 3 |
| Pre-push review (HITL) | 3 |
| Coverage report | 3 |
| Push to Copilot Studio | 3 |
| Point-test | 3 |
| Conductor orchestration | 3 |

**Not covered** (for subsequent iterations): bc-instructions-patterns with protection rules, bc-agent-blueprints (full blueprint), batch test suites, generative actions, multi-action topics, child agents, knowledge sources.

---

## Next Steps

If Order Tracker works, you've validated the core CIRCE flow. Three natural paths forward:

**Option A — Add collections capabilities:** Query overdue invoices + send payment reminder emails → converts Order Tracker into a mini Collections Agent that validates `bc-instructions-patterns` with protection rules and Outlook MCP.

**Option B — Use a full blueprint:** Build a complete Collections Agent from the BC Extension Pack blueprint → validates `bc-agent-blueprints` end-to-end.

**Option C — Build a non-BC agent:** Create a knowledge-based support agent with SharePoint sources → validates the framework without BC dependencies.

---

## BC Entities

| Entity | API Page | Page ID | Operations |
|--------|----------|---------|------------|
| Sales Orders | APIV2 - Sales Orders | 30023 | Read |
| Sales Order Lines | APIV2 - Sales Order Lines | — | Read |
| Customers | APIV2 - Customers | 30009 | Read |

---

## Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| "No agent.mcs.yml found" | Agent not cloned | Run Phase 1 clone step |
| Clone fails with "access denied" | No Power Platform access | Verify permissions in admin.powerplatform.microsoft.com |
| MCP tools not visible in Copilot Studio | Feature not enabled in BC | Check Feature Management page 2610 → "Enable MCP Server access" |
| Push fails with ConcurrencyVersionMismatch | Stale row versions | Pull first, then push |
| Test returns no data | Agent not published | Publish in Copilot Studio UI after push (draft ≠ published) |
| Adaptive Card not rendering | JSON syntax error | Validate topic with `schema-lookup.bundle.js validate` |
| Wrong topic triggered | Overlapping trigger phrases | Review `modelDescription` — make it specific to order status |
| "Extension not found" | Copilot Studio VS Code extension missing | `code --install-extension ms-CopilotStudio.vscode-copilotstudio` |

---

## Environment Reference

Replace these placeholders with your own values:

```
BC_ENVIRONMENT=YOUR_BC_ENVIRONMENT
BC_COMPANY=CRONUS USA, Inc.
BC_MCP_CONFIG=ORDER-TRACKER

CS_TENANT_ID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
CS_ENVIRONMENT_ID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
CS_ENVIRONMENT_URL=https://your-org.crm.dynamics.com/
CS_AGENT_MGMT_URL=https://powervamg.your-region.gateway.prod.island.powerapps.com
CS_AGENT_ID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
CS_CLIENT_ID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

---

## Document History

| Version | Date | Author | Notes |
|---------|------|--------|-------|
| 0.1 | 2026-03-25 | Javier Armesto | Initial test case |
| 0.2 | 2026-03-25 | Javier Armesto | Updated: clone from blank agent instead of scaffold |
| 0.3 | 2026-03-25 | Javier Armesto | Condensed to 3 phases / 3 prompts |
| 1.0 | 2026-03-26 | Javier Armesto | Unified quickstart for CIRCE launch: merged test case + public docs |
