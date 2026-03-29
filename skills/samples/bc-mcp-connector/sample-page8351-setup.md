# Page 8351 Configuration — SALES-AGENT-V1

## Configuration Header

| Field | Value |
|-------|-------|
| Name | SALES-AGENT-V1 |
| Description | MCP configuration for Lead Tracking sales agent |
| Active | Yes |
| Dynamic Tool Mode | No (fewer than 70 pages) |
| Discover Additional Objects | No |
| Unblock Edit Tools | Yes (extension requires Create on custom pages) |

## Tools — Custom API Pages

| API Page | Page ID | Read | Create | Modify | Delete | Bound Actions |
|----------|---------|------|--------|--------|--------|---------------|
| customLeadTracking | 50100 | Yes | Yes | Yes | No | No |
| customLeadActivities | 50101 | Yes | Yes | No | No | No |

## Tools — Standard BC API Pages

| API Page | Page ID | Read | Create | Modify | Delete | Bound Actions |
|----------|---------|------|--------|--------|--------|---------------|
| APIV2 - Customer | 30009 | Yes | No | No | No | No |
| APIV2 - Sales Order | 30042 | Yes | No | No | No | No |
| APIV2 - Sales Invoice | 30043 | Yes | No | No | No | No |

## Skills Evidencing

**Skill loaded:** bc-mcp-connector.skill
**Manifest source:** bc-lead-tracking-manifest.md
**Connection parameters extracted:** Environment, Company, MCP Configuration Code
**Templates generated:** bc-mcp-page8351-setup.md
**HITL gates passed:** Manifest confirmation, BC prerequisites, Tool mode, Connection type
