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
   Read: .github/skills/bc-action-templates/<template-name>.mcs.yml
   ```

3. **Adapt the `connectionReference`** — copy it from the existing cloud-pulled MCP action:
   ```
   Glob: **/actions/Dynamics365BusinessCentral*.mcs.yml
   ```
   Copy the `connectionReference` value from that file. Replace `_REPLACE_CONNECTION_REFERENCE` in the template with this value.

4. **Customize if needed**:
   - Edit `modelDescription` to match the agent's specific domain language
   - Adjust `description` fields to improve orchestrator routing accuracy
   - Copy `ManualTaskInput` entries (`bcenvironment`, `company`, `configurationName`) from the cloud-pulled MCP action and add them to the template
   - Do NOT add `AutomaticTaskInput` or `outputs` — MCP handles user parameters and responses dynamically

5. **Write the action file** to the agent's `actions/` directory:
   ```
   <agent-dir>/actions/<FileName>.mcs.yml
   ```
   Use a descriptive file name matching the operation (e.g., `BC-CustomerBalance.mcs.yml`).

7. **Validate**:
   ```bash
   node .github/scripts/schema-lookup.bundle.js validate <action-file>
   ```

8. **Inform the user** about next steps:
   - Push via the Copilot Studio VS Code Extension
   - The action will be available to the orchestrator based on its `modelDescription`
   - Test with a relevant utterance to verify routing

## Important Notes

- **MCP actions use `InvokeExternalAgentTaskAction`** with `operationId: InvokeMCP`, NOT `InvokeConnectorTaskAction` — this is different from regular connector actions.
- **MCP actions use `ManualTaskInput` for environment config** — three entries: `bcenvironment`, `company`, `configurationName`. Copy these from the cloud-pulled MCP action. They can also be set via Copilot Studio UI → Inputs tab (synced on pull/push).
- **MCP actions have NO `AutomaticTaskInput` or `outputs`** — the MCP server handles user parameters and responses dynamically. Only define `inputs` (ManualTaskInput for env config), `modelDisplayName`, `modelDescription`, `action` block, and `connectionReference`.
- **All templates use `mode: Invoker`** — each end user authenticates with their own BC credentials.
- **The `connectionReference` must match the cloud-pulled MCP action** — copy it from `Dynamics365BusinessCentral-*.mcs.yml`, not from `connectionreferences.mcs.yml`.
- **Templates are starting points** — the MCP server exposes operations dynamically. The `modelDescription` is what the orchestrator uses to decide when to invoke the action.
- **Do not change `operationId: InvokeMCP`** — this is the MCP protocol operation, not a BC-specific operation.
- **In most cases you do NOT need separate action files** — the single cloud-pulled MCP action already gives the orchestrator access to all BC tools. Only create additional action files if you need different `modelDescription` values to improve routing for specific scenarios.
