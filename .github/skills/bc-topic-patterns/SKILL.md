---
user-invocable: false
name: bc-topic-patterns
description: Topic patterns and templates for common Business Central scenarios in Copilot Studio. Covers collections, order inquiry, incident management, and customer onboarding with decision guidance on orchestrator vs custom topic.
argument-hint: "<pattern: collections|order-inquiry|incident|customer-onboarding>"
---

# BC Topic Patterns

Reusable topic patterns for Business Central agents in Copilot Studio. Each pattern includes flow documentation, key design decisions, and YAML snippets. Full templates are in `templates/`.

## When to Use This Skill

Use when:
- The user asks for a "collections topic", "order status flow", "incident form", "customer onboarding", or similar BC-specific conversation flow
- You need to decide whether to create a custom topic or let the orchestrator handle a BC scenario
- You need structural patterns for aging-based branching, dispute checks, or BC action chaining

Do NOT use when:
- The user wants a generic (non-BC) topic — use `/copilot-studio:new-topic`
- The user wants to add a standalone action — use `/copilot-studio:add-action` or `/copilot-studio:bc-action-templates`
- The user wants instructions, not topics — use `/copilot-studio:bc-instructions-patterns`

## Decision Matrix: Orchestrator vs Custom Topic

Before creating a topic, check this matrix. With `GenerativeActionsEnabled: true`, the orchestrator handles many BC scenarios from instructions + MCP actions alone. Only create a custom topic when deterministic control is required.

| Scenario | Orchestrator Only? | Custom Topic? | Why |
|----------|-------------------|---------------|-----|
| Simple balance query | Yes | No | Orchestrator + MCP tool sufficient |
| Balance with aging breakdown | Yes | No | Format via instructions |
| Collections with dispute check | No | Yes | Business rule must be deterministic |
| Order status | Yes | No | Unless specific formatting needed |
| Order status with card | No | Yes | AdaptiveCard requires custom topic |
| Incident creation with card | No | Yes | AdaptiveCard requires custom topic |
| Customer onboarding with card | No | Yes | AdaptiveCard + conditional flow |
| Payment registration | Yes | No | MCP action with confirmation via instructions |
| Item availability check | Yes | No | Orchestrator + MCP tool sufficient |

**Rule of thumb**: If the scenario needs an AdaptiveCard, deterministic business-rule enforcement, or multi-step conditional branching, create a custom topic. Otherwise, let the orchestrator handle it.

**STOP CHECK**: If the decision matrix says "Orchestrator Only" for your scenario, do NOT create a topic. Instead, use `/copilot-studio:bc-instructions-patterns` to add routing guidance to the agent instructions, or `/copilot-studio:bc-action-templates` to add a specialized MCP action with a better `modelDescription`.

**MCP action limitation**: MCP actions (`InvokeExternalAgentTaskAction` with `operationId: InvokeMCP`) can only be invoked by the generative orchestrator. They CANNOT be called via `BeginDialog` from within a topic. If your custom topic needs BC data, the orchestrator must provide it via topic inputs (AutomaticTaskInput), or you must use a regular connector action (`InvokeConnectorTaskAction` with `operationId: GetItemV3`) instead.

## Available Patterns

| Pattern | File | When to Use |
|---------|------|-------------|
| Collections Flow | [patterns/collections-flow.md](patterns/collections-flow.md) | Dispute-checked collections with aging branching |
| Order Inquiry | [patterns/order-inquiry.md](patterns/order-inquiry.md) | Sales order lookup with status-based responses |
| Incident Management | [patterns/incident-management.md](patterns/incident-management.md) | Incident creation with AdaptiveCard confirmation |
| Customer Onboarding | [patterns/customer-onboarding.md](patterns/customer-onboarding.md) | Customer lookup with profile card and welcome flow |

## Available Templates

| Template | File | Description |
|----------|------|-------------|
| Collections Reminder | [templates/collections-reminder.topic.mcs.yml](templates/collections-reminder.topic.mcs.yml) | Full AdaptiveDialog implementing the collections flow pattern |
| Customer Lookup | [templates/customer-lookup.topic.mcs.yml](templates/customer-lookup.topic.mcs.yml) | Reusable sub-flow that resolves a customer by name/number |

## Instructions

1. **Determine which pattern** the user needs from `$ARGUMENTS` or the request context:
   - If the user names a specific scenario, match it to the patterns table.
   - If unclear, show the decision matrix and ask.

2. **Check the decision matrix** — if the scenario is "Orchestrator Only", explain that a custom topic isn't needed and suggest using `/copilot-studio:bc-instructions-patterns` or `/copilot-studio:bc-action-templates` instead.

3. **Read the matching pattern file**:
   ```
   Read: ${CLAUDE_SKILL_DIR}/patterns/<pattern-name>.md
   ```

4. **If a template exists**, read it and adapt:
   ```
   Read: ${CLAUDE_SKILL_DIR}/templates/<template-name>.topic.mcs.yml
   ```
   - Replace `_REPLACE` placeholders with fresh random IDs
   - Update environment values from the agent's existing BC MCP action
   - Adapt trigger phrases and messages to the agent's domain language

5. **If no template exists**, use the pattern's YAML snippets and build the topic using `/copilot-studio:new-topic` + `/copilot-studio:add-node` as needed.

6. **Validate** the topic:
   ```bash
   node ${CLAUDE_SKILL_DIR}/../../scripts/schema-lookup.bundle.js validate <topic-file>
   ```

7. **Inform the user** about next steps:
   - Push via the Copilot Studio VS Code Extension
   - The topic works alongside MCP actions — the orchestrator chains them based on `modelDescription` and topic inputs/outputs
   - Test with relevant utterances to verify routing

## Important Notes

- **Dispute check is NON-NEGOTIABLE**: The collections pattern enforces the protection rules from `agent.mcs.yml` as a ConditionGroup in the topic YAML — this is double protection (instructions + deterministic check). Never skip this.
- **Spanish content**: All trigger phrases, messages, labels, and AdaptiveCard text must be in Spanish per CIRCE conventions.
- **AutomaticTaskInput for customer identification**: All patterns use `AutomaticTaskInput` so the orchestrator collects customer info from context rather than explicit Question nodes.
- **Topic outputs for orchestrator composition**: Patterns that prepare data for other actions use topic outputs (not SendActivity) so the orchestrator can chain them.
