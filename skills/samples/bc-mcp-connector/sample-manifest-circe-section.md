# Sample — CIRCE MCP Connection Context in an ALDC Manifest

This is an example of the CIRCE section that an ALDC extension manifest includes when the extension exposes API pages for use by a Copilot Studio agent.

---

## CIRCE — MCP Connection Context

**BC Environment:** Production
**Company:** CRONUS International Ltd.
**MCP Configuration Code:** SALES-AGENT-V1

## Relevant MCP Tools

| Tool Name (API entityName) | Domain | Purpose |
|----------------------------|--------|---------|
| customers | Sales | Customer master data |
| salesOrders | Sales | Order management |
| salesInvoices | Sales | Invoice lookup |
| customLeadTracking | Sales | Lead tracking (custom extension) |
| customLeadActivities | Sales | Lead activity log (custom extension) |

**Custom tools created by this extension:** customLeadTracking, customLeadActivities
**Standard BC tools relevant to this domain:** customers, salesOrders, salesInvoices, salesQuotes, salesCreditMemos

**Repo reference:** https://github.com/contoso/bc-lead-tracking
