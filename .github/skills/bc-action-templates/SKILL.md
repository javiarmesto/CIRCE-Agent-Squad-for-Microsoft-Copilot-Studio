---
user-invocable: false
name: bc-action-templates
description: Pre-built TaskDialog templates for common Business Central MCP operations. Provides ready-to-use action YAML files for customer balance, overdue invoices, payment creation, sales orders, incident creation, and item availability.
argument-hint: <template-name: customer-balance|overdue-invoices|create-payment|sales-orders|create-incident|item-availability>
---

# BC Action Templates

Pre-built `TaskDialog` YAML templates for common Business Central MCP operations. Each template is structurally complete and optimized for the generative orchestrator — copy, adjust environment values, and push.

## When to Use This Skill

Use when:
- The user wants to add a BC MCP action and the operation maps to one of the available templates
- The `/copilot-studio:add-action` or `/copilot-studio:edit-action` skills need a starting point for a BC-specific action
- Creating a new BC agent that needs standard BC operations

Do NOT use when:
- The action is for a non-BC connector (Outlook, Teams, SharePoint) — use `/copilot-studio:add-action` instead
- The user needs an operation not covered by these templates — use `/copilot-studio:add-action` with connector-lookup

## Available Templates

| Template | File | Description |
|----------|------|-------------|
| Customer Balance | `customer-balance.mcs.yml` | Query customer ledger entries and current balance |
| Overdue Invoices | `overdue-invoices.mcs.yml` | List overdue invoices with aging bucket classification |
| Create Payment | `create-payment.mcs.yml` | Register a payment journal entry for a customer |
| Sales Orders | `sales-orders.mcs.yml` | Query sales orders by customer, date range, or status |
| Create Incident | `create-incident.mcs.yml` | Create a service incident / case for a customer |
| Item Availability | `item-availability.mcs.yml` | Check item stock levels and availability by location |

## Instructions

1. **Determine which template** the user needs from `$ARGUMENTS` or the request context:
   - If the user names a specific operation, match it to the template table above.
   - If unclear, show the table and ask which one.

2. **Read the template file**:
   ```
   Read: ${CLAUDE_SKILL_DIR}/<template-name>.mcs.yml
   ```

3. **Adapt environment values** — each template uses placeholders that must match the agent's BC configuration. Read the agent's existing BC MCP action to extract the correct values:
   ```
   Glob: **/actions/Dynamics365BusinessCentral*.mcs.yml
   ```
   Copy these values from the existing action:
   - `bcenvironment` — e.g., `SANDBOX_US`
   - `company` — e.g., `CRONUS USA, Inc.`
   - `configurationName` — e.g., `CIRCE`
   - `connectionReference` — the full logical name from `connectionreferences.mcs.yml`

4. **Replace `_REPLACE` placeholders** in all IDs with fresh random alphanumeric values (6–8 chars).

5. **Customize if needed**:
   - Edit `modelDescription` to match the agent's specific domain language
   - Add or remove `AutomaticTaskInput` entries based on the agent's needs
   - Adjust `description` fields to improve orchestrator routing accuracy

6. **Write the action file** to the agent's `actions/` directory:
   ```
   <agent-dir>/actions/<FileName>.mcs.yml
   ```
   Use a descriptive file name matching the operation (e.g., `BC-CustomerBalance.mcs.yml`).

7. **Validate**:
   ```bash
   node ${CLAUDE_SKILL_DIR}/../../scripts/schema-lookup.bundle.js validate <action-file>
   ```

8. **Inform the user** about next steps:
   - Push via the Copilot Studio VS Code Extension
   - The action will be available to the orchestrator based on its `modelDescription`
   - Test with a relevant utterance to verify routing

## Important Notes

- **MCP actions use `InvokeExternalAgentTaskAction`** with `operationId: InvokeMCP`, NOT `InvokeConnectorTaskAction` — this is different from regular connector actions.
- **All templates use `mode: Invoker`** — each end user authenticates with their own BC credentials.
- **`ManualTaskInput` values are environment-specific** — always verify against the agent's existing BC action.
- **Templates are starting points** — the MCP server exposes operations dynamically. The `modelDescription` is what the orchestrator uses to decide when to invoke the action.
- **Do not change `operationId: InvokeMCP`** — this is the MCP protocol operation, not a BC-specific operation.
