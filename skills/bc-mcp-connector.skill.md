# BC MCP Connector Setup

**Description:** Reads the CIRCE section of an ALDC extension manifest and guides the user through configuring the BC MCP connector in Copilot Studio, including BC-side prerequisites and agent-side connection.

## Trigger

This skill activates when:

1. The agent detects an ALDC extension manifest in the workspace (file matching `*-manifest.md` with a "CIRCE — MCP Connection Context" section).
2. The user explicitly asks to configure the BC MCP connection.
3. The agent is starting a new CIRCE project that requires BC data access.

## Instructions

### Step 0 — Manifest Discovery

Search the workspace for a manifest file:

```
Glob: **/*-manifest.md
```

If found, read the file and look for the section header `## CIRCE — MCP Connection Context`. Extract three values: **BC Environment**, **Company**, **MCP Configuration Code**. Also extract the "Relevant MCP Tools" table and the fields "Custom tools created by this extension" and "Standard BC tools relevant to this domain".

If no manifest is found, ask the user for the three connection parameters directly and skip to Gate 2.

### Gate 1 — Manifest Confirmation

Present the extracted values and wait for explicit user confirmation:

> I found the following BC connection details in the manifest:
>
>   Environment: {value from manifest}
>   Company: {value from manifest}
>   MCP Configuration Code: {value from manifest}
>
> Are these correct? (yes / no — if no, provide the correct values)

If the user corrects any value, use the corrected values for all subsequent steps.

### Gate 2 — BC-Side Prerequisites Check

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

If **"not yet"**, provide direct URLs:

> Feature management: `https://businesscentral.dynamics.com/?page=2610`
> MCP configurations: `https://businesscentral.dynamics.com/?page=8351`

Do not proceed until the user confirms. The Copilot Studio connector will show zero tools if Page 8351 is not configured.

### Gate 3 — Tool Mode Decision

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

If the user selects **(b)** or the count exceeds 70, Dynamic Tool Mode is mandatory. Record the choice for template generation.

### Gate 4 — Connection Type

> Where will this agent run?
>
>   (a) Copilot Studio (prebuilt connector, no code needed)
>   (b) Claude Desktop / Cursor / VS Code (proxy required)
>   (c) Both

Record the selection. This determines which output templates to generate.

### Step 5 — Generate Output Templates

Based on the four gates, generate the appropriate artifacts in the agent directory (next to `agent.mcs.yml`). Discover the agent directory first:

```
Glob: **/agent.mcs.yml
```

Use the parent directory of the found file as the output location. If no agent exists, use the workspace root.

#### Template A — Copilot Studio Connection Steps

Generate when Gate 4 = (a) or (c). Output file: `bc-mcp-connection-steps.md`.

Contents:

1. **Connection Parameters** block with connector name ("Dynamics 365 Business Central MCP Server (Preview)"), the three parameters pre-filled from the manifest.
2. **Tool Mode** section reflecting the Gate 3 choice (Static or Dynamic).
3. **Available Tools** section split into "Custom tools from this extension" and "Standard BC tools in scope", with tool names formatted per the naming convention (`List<object>_PAG<ID>`, `Create<object>_PAG<ID>`, etc.) for Static mode, or the three meta-tools for Dynamic mode.
4. **Recommended System Instructions** tailored to the agent's domain (derived from manifest description or MCP Configuration Code).
5. **Verification Checklist** with test utterances matching the tools exposed.
6. **Skills Evidencing** block (see section below).

#### Template B — Claude Desktop / External Proxy Config

Generate when Gate 4 = (b) or (c). Output file: `claude-desktop-config.json`.

Pre-fill `Environment`, `Company`, `ConfigurationName` from the manifest. Use `<YOUR-TENANT-ID>` and `<YOUR-CLIENT-ID>` as the only placeholders, with a comment block explaining:

> Azure AD App Registration requirements: "Allow public client flows" enabled,
> redirect URI `ms-appx-web://Microsoft.AAD.BrokerPlugin/{ClientId}`,
> delegated permissions: Financials.ReadWrite.All, user_impersonation.

Proxy install command: `python -m pip install --upgrade bc-mcp-proxy && python -m bc_mcp_proxy setup`.

#### Template C — Page 8351 Configuration Guide

Always generate. Output file: `bc-mcp-page8351-setup.md`.

Contents:

1. **Configuration Header** table with Name, Description, Active, Dynamic Tool Mode, Discover Additional Objects, Unblock Edit Tools — all pre-filled from manifest data and Gate 3 choice.
2. **Tools — Custom API Pages** table listing each custom tool from the manifest with columns: API Page, Page ID, Read, Create, Modify, Delete, Bound Actions. Derive permissions from the manifest's tool purpose descriptions (read-only for lookup tools, read+create for data-entry tools).
3. **Tools — Standard BC API Pages** table listing standard tools from the manifest with read-only permissions by default.
4. **Skills Evidencing** block.

### Step 6 — Summary

After generating all templates, present a summary:

> **BC MCP Connector setup complete.**
>
> Generated files:
>   {list of files generated with paths}
>
> Next steps:
>   For Copilot Studio: follow `bc-mcp-connection-steps.md` to add the connector via the portal.
>   For Claude Desktop: copy `claude-desktop-config.json` to your Claude Desktop config directory.
>   For Page 8351: follow `bc-mcp-page8351-setup.md` to configure the MCP server in BC.

## Agent Behavior Rules

1. **Always read the manifest first.** Extract BC Environment, Company, and MCP Configuration Code from the CIRCE section. Never ask the user for values that exist in the manifest.

2. **Never skip the prerequisites check (Gate 2).** The BC-side feature flag and Page 8351 configuration are mandatory. If not done, the Copilot Studio connector will show no tools.

3. **Tool mode recommendation must match reality.** Count the total API pages the configuration exposes (custom + standard). If over 70, Dynamic Tool Mode is mandatory, not optional.

4. **Pre-fill everything possible.** Templates should have zero placeholders for data that exists in the manifest. Only TenantId and ClientId (proxy path) require user input because they are environment-specific credentials.

5. **Do not duplicate Microsoft documentation.** Link to the reference URLs below. The skill provides templates and checklists, not a rewrite of Learn docs.

6. **Skills Evidencing is mandatory in every output file.**

## Skills Evidencing

Every generated output file must include this block at the end:

```markdown
## Skills Evidencing

**Skill loaded:** bc-mcp-connector.skill
**Manifest source:** {manifest filename}
**Connection parameters extracted:** Environment, Company, MCP Configuration Code
**Templates generated:** {list of templates generated based on user selections}
**HITL gates passed:** Manifest confirmation, BC prerequisites, Tool mode, Connection type
```

## Reference

| Topic | URL |
|-------|-----|
| Configure BC MCP Server | https://learn.microsoft.com/en-us/dynamics365/business-central/dev-itpro/ai/configure-mcp-server |
| Create Agent in Copilot Studio | https://learn.microsoft.com/en-us/dynamics365/business-central/dev-itpro/ai/create-agent-in-copilot-studio |
| BCTech Proxy Samples | https://github.com/microsoft/BCTech/tree/master/samples/BcMCPProxy |

## Samples

See `skills/samples/bc-mcp-connector/` for example manifest sections and generated outputs.
