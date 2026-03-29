# BC MCP Connection — Lead Tracking Agent

## Connection Parameters

**Connector:** Dynamics 365 Business Central MCP Server (Preview)
**Environment:** Production
**Company:** CRONUS International Ltd.
**MCP Server Configuration:** SALES-AGENT-V1

## Tool Mode

Static Mode (fewer than 70 API pages exposed)

## Available Tools

Custom tools from this extension:

| Tool Name | Permission |
|-----------|------------|
| ListcustomLeadTracking_PAG50100 | Read |
| CreatecustomLeadTracking_PAG50100 | Create |
| ListcustomLeadActivities_PAG50101 | Read |
| CreatecustomLeadActivities_PAG50101 | Create |

Standard BC tools in scope:

| Tool Name | Permission |
|-----------|------------|
| ListAPIV2 - Customer_PAG30009 | Read |
| ListAPIV2 - Sales Order_PAG30042 | Read |
| ListAPIV2 - Sales Invoice_PAG30043 | Read |

## Recommended System Instructions

```
You are a Business Central sales agent. You help users manage leads and
track sales activities. Start by outlining a plan, then use available tools
to retrieve or update data. Clearly show your reasoning before invoking tools.
Prefer using semantic search when searching for available actions.
```

## Verification Checklist

- [ ] Connection created and authenticated in Copilot Studio
- [ ] Tools section shows expected tools (custom + standard)
- [ ] Test panel: "List all customers" returns data
- [ ] Test panel: "Create a new lead for Adatum Corporation" creates record
- [ ] Test panel: "Show my recent lead activities" returns activity log

## Skills Evidencing

**Skill loaded:** bc-mcp-connector.skill
**Manifest source:** bc-lead-tracking-manifest.md
**Connection parameters extracted:** Environment, Company, MCP Configuration Code
**Templates generated:** bc-mcp-connection-steps.md, bc-mcp-page8351-setup.md
**HITL gates passed:** Manifest confirmation, BC prerequisites, Tool mode, Connection type
