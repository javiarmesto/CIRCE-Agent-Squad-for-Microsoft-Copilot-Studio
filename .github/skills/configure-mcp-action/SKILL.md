---
user-invocable: false
name: configure-mcp-action
description: Interactively configure an MCP (Model Context Protocol) action in a Copilot Studio agent. Walks the user through each configurable field, showing current values and asking for changes.
argument-hint: <optional: which MCP action to configure>
---

# Configure MCP Action

Interactively configure an existing MCP action (`kind: TaskDialog`, `action.kind: InvokeExternalAgentTaskAction`). The skill reads the current configuration, presents each field to the user, and applies only the changes they confirm.

## MCP Action Structure

```yaml
mcs.metadata:
  componentName: <Display Name - Connector Display Name>
kind: TaskDialog
inputs:
  - kind: ManualTaskInput
    propertyName: bcenvironment
    value: <BC environment name>
  - kind: ManualTaskInput
    propertyName: configurationName
    value: <MCP Server Configuration name>
  - kind: ManualTaskInput
    propertyName: company
    value: <BC company name>
modelDisplayName: <Name shown to AI orchestrator>
modelDescription: <Description for AI routing>
action:
  kind: InvokeExternalAgentTaskAction
  connectionReference: <schemaName>.<connectorRef>.<connectionId>
  connectionProperties:
    mode: Invoker                # or Maker
  operationDetails:
    kind: ModelContextProtocolMetadata
    operationId: InvokeMCP
```

> **NOTE**: MCP actions use `ManualTaskInput` entries for environment configuration (`bcenvironment`, `company`, `configurationName`). These values can be set in the YAML directly or via the Copilot Studio UI → Tools → MCP action → **Inputs tab** (they sync on pull). MCP actions have **no `outputs`** and **no `AutomaticTaskInput`** — the MCP server handles parameters and responses dynamically.

### Key Differences from Connector Actions

| Property | Connector Action | MCP Action |
|----------|-----------------|------------|
| `action.kind` | `InvokeConnectorTaskAction` | `InvokeExternalAgentTaskAction` |
| Operation ID | `action.operationId` (top-level) | `action.operationDetails.operationId` (nested) |
| Operation metadata | — | `action.operationDetails.kind: ModelContextProtocolMetadata` |
| Inputs | `AutomaticTaskInput` + `ManualTaskInput` | `ManualTaskInput` only (bcenvironment, company, configurationName) — no `AutomaticTaskInput` |
| Outputs | Explicit `outputs` list | **None** — MCP returns responses dynamically |

## Instructions

1. **Auto-discover the agent directory**:
   ```
   Glob: **/agent.mcs.yml
   ```
   If multiple agents found, ask which one.

2. **Find and select the MCP action file**:
   ```
   Glob: <agent-dir>/actions/*.mcs.yml
   ```
   Read each action file and identify MCP actions by `action.kind: InvokeExternalAgentTaskAction`.
   - If only one MCP action exists, select it automatically.
   - If multiple MCP actions exist, list them to the user and ask which one to configure.
   - If none found, inform the user that no MCP actions exist and suggest using `/copilot-studio:add-action` first.

3. **Read the current action file** and extract all configurable values.

4. **Present current configuration and ask for changes interactively**. Show the user a summary of the current values and ask what they want to change. Structure the interaction by section:

   ### Section 1: Environment Inputs (BC MCP)
   If the action has `ManualTaskInput` entries for BC environment config, present each one:

   > **Configuración actual de inputs:**
   >
   > | Input | Valor actual |
   > |-------|-------------|
   > | `bcenvironment` | `<current value>` |
   > | `company` | `<current value>` |
   > | `configurationName` | `<current value>` |
   >
   > ¿Quieres cambiar alguno de estos valores? Indica cuáles y sus nuevos valores, o escribe "OK" para mantenerlos.

   If the action has no inputs (e.g., Outlook MCP), skip this section.

   **Typical `bcenvironment` values**: `SANDBOX`, `AGENTNEW`, `PRODUCTION`, or any custom environment name from BC admin center.

   > **TIP**: These values can also be set via Copilot Studio UI → Tools → MCP action → **Inputs tab**. Changes sync on pull/push.

   ### Section 2: Display Name & Description
   Present the current orchestrator-facing metadata:

   > **Metadatos para el orquestador:**
   >
   > | Campo | Valor actual |
   > |-------|-------------|
   > | `modelDisplayName` | `Dynamics 365 Business Central MCP (Preview)` |
   > | `modelDescription` | `Provides MCP Server access to Dynamics 365 Business Central (Preview).` |
   >
   > El `modelDescription` es lo que el orquestador IA usa para decidir cuándo invocar esta acción. Un buen description incluye los tipos de datos y operaciones que el MCP server expone.
   >
   > ¿Quieres modificar alguno? Indica los nuevos valores o "OK" para mantenerlos.

   ### Section 3: Connection Mode
   Present the current connection mode:

   > **Modo de conexión:** `Invoker`
   >
   > - **Invoker** (recomendado): cada usuario se autentica con sus propias credenciales
   > - **Maker**: todas las peticiones usan las credenciales del desarrollador
   >
   > ¿Cambiar? Indica `Invoker` o `Maker`, o "OK" para mantener.

   ### Section 4: Additional Inputs (optional)
   Ask if the user wants to add new `ManualTaskInput` entries beyond the standard three:

   > ¿Quieres añadir algún input manual adicional? Indica el `propertyName` y `value`, o "No" para continuar.

5. **Wait for the user's response before making any changes.** Do NOT edit the file until the user confirms what they want to change. If the user says "OK" or equivalent to all sections, inform them that no changes are needed.

6. **Apply all confirmed changes** using the Edit tool. Make all edits in a single batch.

7. **Validate the edited file**:
   ```bash
   node ${CLAUDE_SKILL_DIR}/../../scripts/schema-lookup.bundle.js validate <action-file-path>
   ```

8. **If environment values changed, check agent instructions for sync**. Read `agent.mcs.yml` and check if the instructions section references the old environment, company, or configuration. If so, ask the user:

   > Las instrucciones del agente en `agent.mcs.yml` referencian valores de entorno. ¿Quieres que las actualice también para que coincidan con la nueva configuración?

   If yes, use `/copilot-studio:edit-agent` to update.

9. **Verify connection reference** exists in `connectionreferences.mcs.yml`:
   ```
   Read: <agent-dir>/connectionreferences.mcs.yml
   ```
   The `connectionReference` value must match a `connectionReferenceLogicalName`. If it doesn't, warn the user.

10. **Summarize changes** to the user:

    > **Cambios aplicados:**
    > - `bcenvironment`: `SANDBOX` → `AGENTNEW`
    > - `modelDescription`: actualizado para mejor enrutamiento
    > - Validación: ✅ OK
    >
    > Recuerda hacer Push (extensión VS Code) y Publish (portal Copilot Studio).

## Important Rules

- **Never change `action.operationDetails.operationId`** — this identifies which MCP operation runs. Changing it breaks the action.
- **Never change `action.connectionReference`** — this links to the authenticated connection created via the portal. Changing it breaks the action.
- **Never change `action.operationDetails.kind`** — must remain `ModelContextProtocolMetadata`.
- **MCP actions use `ManualTaskInput` for environment config only** — `bcenvironment`, `company`, `configurationName`. Do NOT add `AutomaticTaskInput` entries or `outputs`. These values can be edited in YAML or via Copilot Studio UI → Inputs tab (they sync on pull/push).
- **Keep `mcs.metadata.componentName` in sync** — it should reflect the connector and action name. Format: `<Connector> - <Action Display Name>`.
- **After changing environment values (in UI)**, always check that the `agent.mcs.yml` instructions section references the same environment, company, and configuration. Mismatches cause confusing AI behavior.
- **Wait for user confirmation** before editing. Never apply changes preemptively.

## Workflow Summary

```
Discover → Read action → Present current values → Ask for changes →
Wait for confirmation → Apply edits → Validate → Sync instructions → Summarize
```
