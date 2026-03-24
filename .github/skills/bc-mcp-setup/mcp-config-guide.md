# BC MCP Server Configuration — Reference Guide

Detailed reference for configuring the Business Central MCP Server and connecting it to Copilot Studio agents.

## MCP Server Configuration Page (BC page 8351)

### General Fields

| Field | Type | Description |
|-------|------|-------------|
| **Name** | Text (50) | Unique name for the configuration. Appears in Copilot Studio when assigning to an agent. Use descriptive names matching the agent purpose (e.g., `CIRCE`, `SALES-AGENT`, `SUPPORT-BOT`). |
| **Description** | Text (250) | Brief description of the configuration's purpose. Not visible to the AI orchestrator. |
| **Active** | Boolean | When ON, the configuration and its tools are available for agents. Turning OFF breaks agents currently using this configuration. |
| **Dynamic Tool Mode** | Boolean | Controls how tools are exposed to agents. See [Dynamic Tool Mode](#dynamic-tool-mode-comparison) section. |
| **Discover Additional Objects** | Boolean | Only works when Dynamic Tool Mode is ON. When ON, agents get read-only access to ALL API page objects in the environment, even those not explicitly added. |
| **Unblock Edit Tools** | Boolean | Master switch for write operations. When OFF, all create/modify/delete permissions are forced to false regardless of individual tool settings. |

### Tool Entry Fields (API Page Objects)

Each row in the Tools section represents an API page with its CRUD permissions:

| Field | Type | Description |
|-------|------|-------------|
| **API Page** | Lookup | The API page object to expose as a tool. Only top-level API pages supported — `ListPart` and `CardPart` subtypes are NOT supported. |
| **Allow Read** | Boolean | Enables list/get operations on the API page. |
| **Allow Create** | Boolean | Enables record creation. Requires **Unblock Edit Tools** = ON. |
| **Allow Modify** | Boolean | Enables record updates. Requires **Unblock Edit Tools** = ON. |
| **Allow Delete** | Boolean | Enables record deletion. Requires **Unblock Edit Tools** = ON. |
| **Allow Bound Actions** | Boolean | Enables OData bound actions on the API page. Requires **Unblock Edit Tools** = ON. |

---

## How API Page Entries Map to MCP Tools

When you add an API page to the configuration, each allowed operation generates a corresponding tool in the MCP server with a specific naming convention.

### Dynamic Tool Mode OFF — Tool Naming

Tools are listed explicitly in Copilot Studio with these names:

| Operation | Tool Name Format | Example (APIV2 - Customer, PAG30009) |
|-----------|-----------------|---------------------------------------|
| Allow Read | `List<object_name>_PAG<ID>` | `ListAPIV2 - Customer_PAG30009` |
| Allow Create | `Create<object_name>_PAG<ID>` | `CreateAPIV2 - Customer_PAG30009` |
| Allow Modify | `ListUpdate<object_name>_PAG<ID>` | `ListUpdateAPIV2 - Customer_PAG30009` |
| Allow Delete | `Delete<object_name>_PAG<ID>` | `DeleteAPIV2 - Customer_PAG30009` |
| Allow Bound Actions | `<bound_action_name>_PAG<ID>` | `PostSalesInvoice_PAG30023` |

These tools appear in Copilot Studio's Tools section and can be individually toggled on/off per agent.

### Dynamic Tool Mode ON — Standard Actions

When Dynamic Tool Mode is ON, tools are NOT listed individually. Instead, three standard server-level actions are available:

| Action | Purpose |
|--------|---------|
| `bc_actions_search` | Searches for available tools matching a semantic query |
| `bc_actions_describe` | Returns detailed description of a specific tool |
| `bc_actions_invoke` | Executes a tool operation with parameters |

The AI orchestrator uses these three actions to dynamically discover and invoke tools at runtime.

---

## Dynamic Tool Mode Comparison

| Aspect | Dynamic Tool Mode **OFF** | Dynamic Tool Mode **ON** |
|--------|--------------------------|--------------------------|
| **Tool visibility** | Each tool listed explicitly in Copilot Studio | Only 3 standard actions visible; tools discovered at runtime |
| **Tool limit** | Subject to Copilot Studio's 70-tool limit | No limit — all configured APIs accessible via dynamic discovery |
| **AI routing** | Orchestrator selects from listed tools directly | Orchestrator searches → describes → invokes (multi-step) |
| **Token consumption** | Lower — direct tool invocation | Higher — discovery steps consume additional tokens/credits |
| **Latency** | Lower — single-step invocation | Higher — search + describe + invoke chain |
| **Control** | Granular — toggle individual tools per agent | Less granular — all configured tools available if found |
| **Best for** | Production agents with defined scope (<70 tools) | Prototyping, large API surface, exploratory agents |
| **Observability** | Clear — each tool invocation logged separately | Less clear — invocations go through generic `bc_actions_invoke` |

### Recommendation

- **Production agents** (like Circe): Dynamic Tool Mode **OFF**. Define exactly which APIs the agent needs. Predictable behavior, lower cost, easier debugging.
- **Prototyping / development**: Dynamic Tool Mode **ON**. Quickly test with all available APIs without hitting the 70-tool limit.
- **Large API surfaces** (100+ custom APIs): Dynamic Tool Mode **ON** is required. Static mode can only expose 70 tools.

---

## Cross-Environment Scenarios

Agents can connect to BC data in a different environment than the one where the MCP Server is configured:

| Scenario | Configuration |
|----------|---------------|
| **Same tenant, different environment** | In Copilot Studio, set the **Environment** input to the target environment name (e.g., `PRODUCTION` while developing in `SANDBOX_US`). The MCP configuration must exist in the target environment. |
| **Same tenant, same environment** | Standard setup — Environment and MCP configuration in the same BC environment. |
| **Multi-company** | Set the **Company** input in Copilot Studio to the target company. Each company shares the same MCP configuration within an environment. |

> ⚠️ The MCP Server Configuration must be created in the **target environment** — the environment where the data lives. You cannot create a configuration in sandbox and point it to production data.

---

## Copilot Credits Consumption

MCP operations consume Copilot Credits. Estimated consumption per operation type:

| Operation Type | Estimated Credits | Notes |
|---------------|-------------------|-------|
| Simple read (list/get) | ~1 credit | Single API call |
| Search + read (Dynamic Mode) | ~2-3 credits | bc_actions_search + bc_actions_invoke |
| Create/update record | ~1-2 credits | Single write operation |
| Complex orchestration | ~3-5 credits | Multiple tool calls in one conversation turn |
| Dynamic discovery chain | ~3-4 credits | search → describe → invoke |

> **Note**: These are rough estimates. Actual consumption depends on payload size, model version, and orchestration complexity. Monitor usage in the Power Platform admin center.

---

## Security Recommendations

### 1. Purpose-Specific API Pages

**DO NOT** use "Add All Standard APIs" in production. Instead:

- Create custom API pages in your AL extension that expose only the fields the agent needs
- Name them clearly: `API - Agent Customers`, `API - Agent Invoices`
- Include only business-relevant fields — exclude system fields, internal IDs, sensitive data
- Set appropriate CRUD permissions per API page

### 2. Field-Level Exposure Control

API pages define which fields are visible to the agent:

```al
page 50100 "API - Agent Customers"
{
    PageType = API;
    APIPublisher = 'mycompany';
    APIGroup = 'agentAPIs';
    APIVersion = 'v1.0';
    EntityName = 'agentCustomer';
    EntitySetName = 'agentCustomers';
    SourceTable = Customer;

    layout
    {
        area(Content)
        {
            // Only expose what the agent needs
            field(number; "No.") { }
            field(displayName; Name) { }
            field(email; "E-Mail") { }
            field(balance; "Balance (LCY)") { }
            field(overdueAmount; "Balance Due (LCY)") { }
            // DO NOT expose: Tax ID, bank accounts, credit limits, etc.
        }
    }
}
```

### 3. Delegated vs Service Authentication

| Mode | When to Use | Security Implications |
|------|------------|----------------------|
| **Invoker (Delegated)** | ✅ Production default | Agent operates with the signed-in user's BC permissions. Users only see data they're authorized to access. |
| **Maker (Service-to-service)** | System agents, scheduled tasks | Agent operates with the connection creator's permissions. All users see the same data regardless of their own permissions. |

**Recommendation**: Always use **Invoker** mode unless the agent serves as a system-level automation with no user context.

### 4. MCP Configuration per Agent

Create separate MCP configurations for each agent type:

| Agent | MCP Config Name | APIs Exposed |
|-------|----------------|-------------|
| Collections agent | `COLLECTIONS` | Customers (R), Ledger Entries (R), Invoices (R), Payments (CRUD), Aging (R) |
| Sales agent | `SALES` | Customers (CRUD), Items (R), Orders (CRUD), Quotes (CRUD) |
| Support agent | `SUPPORT` | Customers (R), Orders (R), Items (R) |

This follows the principle of least privilege — each agent only has access to the data and operations it needs.

### 5. Production Checklist

Before deploying an agent with BC MCP to production:

- [ ] Custom API pages created with minimal field exposure
- [ ] MCP configuration uses Dynamic Tool Mode OFF
- [ ] Unblock Edit Tools enabled ONLY for APIs that need write access
- [ ] Authentication mode set to Invoker (delegated)
- [ ] MCP configuration name is descriptive and documented
- [ ] Agent instructions include guidance on available tools
- [ ] Test validation completed with production-like data
- [ ] Copilot Credits budget allocated and monitored

---

## References

- [Configure Business Central MCP Server](https://learn.microsoft.com/en-us/dynamics365/business-central/dev-itpro/ai/configure-mcp-server)
- [Create agents in Copilot Studio that connect to Business Central](https://learn.microsoft.com/en-us/dynamics365/business-central/dev-itpro/ai/create-agent-in-copilot-studio)
- [Copilot Studio Licensing](https://learn.microsoft.com/en-us/microsoft-copilot-studio/billing-licensing)
- [Feature Management in BC](https://learn.microsoft.com/en-us/dynamics365/business-central/dev-itpro/administration/feature-management)
