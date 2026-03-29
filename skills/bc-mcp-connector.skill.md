---
user-invocable: false
name: bc-mcp-connector
description: Reads the CIRCE section of an ALDC extension manifest and guides the user through configuring the BC MCP connector in Copilot Studio, including BC-side prerequisites and agent-side connection.
argument-hint: <optional: path to manifest file>
---

# BC MCP Connector Setup

This skill automates the connection between a Copilot Studio agent and the Business Central MCP Server. It reads the ALDC extension manifest's CIRCE section, extracts connection parameters, and walks the user through BC-side prerequisites and Copilot Studio connector configuration via interactive HITL gates.

The Business Central MCP connector is a prebuilt, first-party Premium connector in Copilot Studio called **"Dynamics 365 Business Central MCP Server (Preview)"**. It uses the Power Platform operation ID `InvokeMCP`. No custom YAML or OpenAPI file is needed.

## Trigger

This skill activates when:

1. The agent detects an ALDC extension manifest in the workspace (file matching `*-manifest.md` containing a "CIRCE — MCP Connection Context" section).
2. The user explicitly asks to configure the BC MCP connection.
3. The agent is starting a new CIRCE project that requires BC data access.

## Instructions

### Phase 1 — Manifest Detection (Gate 1)

Search for the manifest file:

```
Glob: **/*-manifest.md
```

Read the file and locate the `## CIRCE — MCP Connection Context` section. Extract three values: **BC Environment**, **Company**, and **MCP Configuration Code**. Also extract the "Relevant MCP Tools" table, the "Custom tools created by this extension" list, and the "Standard BC tools relevant to this domain" list.

Present the extracted values for confirmation:

> I found the following BC connection details in the manifest:
>
>   Environment: {value from manifest}
>   Company: {value from manifest}
>   MCP Configuration Code: {value from manifest}
>
> Are these correct? (yes / no — if no, provide the correct values)

Do NOT proceed until the user confirms or provides corrected values.

### Phase 2 — BC-Side Prerequisites Check (Gate 2)

Ask the user to confirm BC-side readiness:

> Before connecting from Copilot Studio, verify these BC-side prerequisites:
>
>   1. Feature flag "Enable MCP Server access" is ON (Page 2610)
>   2. Permission set MCP - ADMIN is assigned to the configuring user
>   3. Configuration "{configurationName}" exists on Page 8351 with:
>      — Active = Yes
>      — The custom API pages from this extension are added as Tools
>      — Per-tool permissions (Read/Create/Modify/Delete/Bound Actions) are set
>
> Have you verified these? (yes / not yet)

If "not yet", provide direct URLs:

> Feature management: `https://businesscentral.dynamics.com/?page=2610`
> MCP configurations: `https://businesscentral.dynamics.com/?page=8351`

Do NOT proceed until the user confirms.

### Phase 3 — Tool Mode Decision (Gate 3)

> How many API pages does this environment expose in total?
>
>   (a) 70 or fewer — use Static Mode (each tool appears individually)
>   (b) More than 70 — use Dynamic Tool Mode (3 meta-tools for discovery)
>   (c) Not sure — I'll check Page 8351
>
> Recommendation: if the manifest lists only custom APIs from this extension,
> Static Mode gives the agent clearer tool visibility. If the environment
> exposes many standard BC APIs alongside custom ones, Dynamic Mode avoids
> the 70-tool ceiling.

If over 70, Dynamic Tool Mode is mandatory, not optional. Record the user's choice for template generation.

### Phase 4 — Connection Type (Gate 4)

> Where will this agent run?
>
>   (a) Copilot Studio (prebuilt connector, no code needed)
>   (b) Claude Desktop / Cursor / VS Code (proxy required)
>   (c) Both

Record the user's choice. Generate the appropriate output templates based on their selection.

### Phase 5 — Generate Output Templates

Based on the four gates, generate the configuration artifacts described below. Pre-fill every value that exists in the manifest. Only TenantId and ClientId (proxy path) require user input.

#### Template A — Copilot Studio Connection Steps

Generate `bc-mcp-connection-steps.md` in the agent directory with:

```markdown
# BC MCP Connection — {Agent Name}

## Connection Parameters

**Connector:** Dynamics 365 Business Central MCP Server (Preview)
**Environment:** {from manifest}
**Company:** {from manifest}
**MCP Server Configuration:** {from manifest}

## Tool Mode

{Static Mode / Dynamic Tool Mode} ({reasoning from Gate 3})

## Available Tools

Custom tools from this extension:
{For each custom tool, list with tool naming convention from static mode:
  List<entityName>_PAG<ID> (Read)
  Create<entityName>_PAG<ID> (Create)
  etc., based on manifest permissions}

Standard BC tools in scope:
{For each standard tool, list with APIV2 naming convention}

(If Dynamic Tool Mode: replace tool lists with bc_actions_search, bc_actions_describe, bc_actions_invoke)

## Recommended System Instructions

You are a Business Central agent. The user will ask a question, or ask you to
perform a task or retrieve data. Start by outlining a plan of what you have and
what you must do and then use the available tools to retrieve the relevant information.
Clearly show your reasoning, before trying to invoke any tool.
Additionally: Prefer using semantic search when searching for available actions.

## Verification Checklist

[ ] Connection created and authenticated in Copilot Studio
[ ] Tools section shows expected tools (custom + standard)
[ ] Test panel: "{read query relevant to domain}" returns data
[ ] Test panel: "{create query relevant to domain}" creates record
[ ] Test panel: "{activity query relevant to domain}" returns results

## Skills Evidencing

**Skill loaded:** bc-mcp-connector.skill
**Manifest source:** {manifest filename}
**Connection parameters extracted:** Environment, Company, MCP Configuration Code
**Templates generated:** bc-mcp-connection-steps.md
**HITL gates passed:** Manifest confirmation, BC prerequisites, Tool mode, Connection type
```

Only generate Template A if the user selected (a) or (c) in Gate 4.

#### Template B — Claude Desktop / External Proxy Config

Generate `claude_desktop_config.json` in the agent directory:

```json
{
  "mcpServers": {
    "BC_{configurationName}": {
      "command": "python",
      "args": [
        "-m", "bc_mcp_proxy",
        "--TenantId", "<YOUR-TENANT-ID>",
        "--ClientId", "<YOUR-CLIENT-ID>",
        "--Environment", "{from manifest}",
        "--Company", "{from manifest}",
        "--ConfigurationName", "{from manifest}"
      ]
    }
  }
}
```

Include setup instructions as a companion markdown file or as comments:

> **Prerequisites for proxy path:**
>
>   1. Install the proxy: `python -m pip install --upgrade bc-mcp-proxy`
>   2. Azure AD App Registration with "Allow public client flows" enabled
>   3. Redirect URI: `ms-appx-web://Microsoft.AAD.BrokerPlugin/{ClientId}`
>   4. Delegated permissions: `Financials.ReadWrite.All` and `user_impersonation`
>
> Replace `<YOUR-TENANT-ID>` and `<YOUR-CLIENT-ID>` with your Azure AD values.
>
> Reference: https://github.com/microsoft/BCTech/tree/master/samples/BcMCPProxy

Only generate Template B if the user selected (b) or (c) in Gate 4.

#### Template C — Page 8351 Configuration Guide

Generate `bc-mcp-page8351-setup.md` in the agent directory:

```markdown
# Page 8351 Configuration — {configurationName}

## Configuration Header

| Field | Value |
|-------|-------|
| Name | {configurationName from manifest} |
| Description | MCP configuration for {agent purpose from manifest} |
| Active | Yes |
| Dynamic Tool Mode | {Yes/No based on Gate 3} |
| Discover Additional Objects | {Yes if Dynamic + broad access, No otherwise} |
| Unblock Edit Tools | {Yes if any custom tool requires Create/Modify/Delete, No otherwise} |

## Tools — Custom API Pages

| API Page | Page ID | Read | Create | Modify | Delete | Bound Actions |
|----------|---------|------|--------|--------|--------|---------------|
{rows from manifest's custom tools with recommended permissions}

## Tools — Standard BC API Pages

| API Page | Page ID | Read | Create | Modify | Delete | Bound Actions |
|----------|---------|------|--------|--------|--------|---------------|
{rows from manifest's standard tools, typically Read-only}

## Skills Evidencing

**Skill loaded:** bc-mcp-connector.skill
**Manifest source:** {manifest filename}
**Connection parameters extracted:** Environment, Company, MCP Configuration Code
**Templates generated:** bc-mcp-page8351-setup.md
**HITL gates passed:** Manifest confirmation, BC prerequisites, Tool mode, Connection type
```

Always generate Template C (it is useful for both connection paths).

### Phase 6 — Summary and Next Steps

After generating all templates, present a summary:

> **Configuration complete.** Generated files:
>
> {list of files generated}
>
> **Next steps:**
>   1. Follow `bc-mcp-page8351-setup.md` to configure Page 8351 in BC (if not done)
>   2. Follow `bc-mcp-connection-steps.md` to connect in Copilot Studio
>   3. Run the verification checklist in the test panel
>   4. After verification, pull the agent files locally and use `/copilot-studio:configure-mcp-action` to fine-tune the action YAML

Offer to update `circe-memory.md` with the MCP configuration details.

## Agent Behavior Rules

1. **Always read the manifest first.** Extract BC Environment, Company, and MCP Configuration Code from the CIRCE section. Never ask the user for values that exist in the manifest.

2. **Never skip the prerequisites check (Gate 2).** The BC-side feature flag and Page 8351 configuration are mandatory. If not done, the Copilot Studio connector will show no tools.

3. **Tool mode recommendation must match reality.** Count the total API pages the configuration exposes (custom + standard). If over 70, Dynamic Tool Mode is mandatory, not optional.

4. **Pre-fill everything possible.** Templates should have zero placeholders for data that exists in the manifest. Only TenantId and ClientId (proxy path) require user input because they are environment-specific credentials.

5. **Do not duplicate Microsoft documentation.** Link to the reference URLs instead. The skill provides templates and checklists, not a rewrite of Learn docs.

6. **Skills Evidencing is mandatory in every output file.**

## References

- [Configure BC MCP Server](https://learn.microsoft.com/en-us/dynamics365/business-central/dev-itpro/ai/configure-mcp-server)
- [Create Agent in Copilot Studio](https://learn.microsoft.com/en-us/dynamics365/business-central/dev-itpro/ai/create-agent-in-copilot-studio)
- [BCTech Proxy Samples](https://github.com/microsoft/BCTech/tree/master/samples/BcMCPProxy)
