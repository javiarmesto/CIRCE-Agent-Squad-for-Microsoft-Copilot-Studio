# Circe — Copilot Studio Agent Project

Circe is a **Microsoft Copilot Studio** bot for collections and payment management.
It integrates with Dynamics 365 Business Central (via MCP) and Microsoft Outlook (via MCP).
All user-facing content is in **Spanish**.

For full technical documentation see [../README.md](../README.md).
For architecture decisions and development patterns see [../CLAUDE.md](../CLAUDE.md).

---

## Always-On Conventions

The file `.github/circe-conventions.md` contains mandatory conventions. Read it before any operation. These rules apply regardless of which agent or skill is active.

All agents must present a CIRCE-EVIDENCE block after completing operations (see `circe-conventions.md` rule #11).

---

## Architecture

| Path | Purpose |
|------|---------|
| `agent.mcs.yml` | Agent metadata display name & model hint |
| `settings.mcs.yml` | schemaName, auth, `GenerativeActionsEnabled`, instructions |
| `connectionreferences.mcs.yml` | Connection references for BC and Outlook |
| `actions/` | External MCP action definitions (BC + Outlook) |
| `topics/` | Conversation topics (`AdaptiveDialog` YAML) |
| `templates/` | Canonical YAML templates — use before writing from scratch |
| `scripts/` | CLI helpers: schema-lookup, connector-lookup, validate |
| `skills/` | Skill guides — read the matching skill before editing a component |

Agent schema name: `copilots_header_cra1e_Circe`  
BC environment: `SANDBOX_US` · company: `CRONUS USA, Inc.` · config: `CIRCE`

---

## Build and Validate

```bash
# Validate any YAML file
node scripts/schema-lookup.bundle.js validate <file.mcs.yml>

# Look up a YAML node schema
node scripts/schema-lookup.bundle.js summary <KindName>

# Look up connector operations
node scripts/connector-lookup.bundle.js operations <connector>
```

After **any** YAML edit, validate the file before pushing. Push via the VS Code Copilot Studio extension; publish via the Copilot Studio UI.

---

## Conventions

### Skill-first rule
**Always invoke the matching skill** — never author YAML manually when a skill exists.
Skills contain correct templates, required fields, and schema validation.

| Task | Skill |
|------|-------|
| New topic | `/copilot-studio:new-topic` |
| Add/edit a node | `/copilot-studio:add-node` |
| Add a connector action | `/copilot-studio:add-action` |
| Edit an existing action | `/copilot-studio:edit-action` |
| Configure an MCP action | `/copilot-studio:configure-mcp-action` |
| Edit agent instructions | `/copilot-studio:edit-agent` |
| Edit trigger phrases | `/copilot-studio:edit-triggers` |
| Add an Adaptive Card | `/copilot-studio:add-adaptive-card` |
| Add generative answers | `/copilot-studio:add-generative-answers` |
| Add knowledge | `/copilot-studio:add-knowledge` |
| Add a global variable | `/copilot-studio:add-global-variable` |
| Add child/connected agents | `/copilot-studio:add-other-agents` |
| Validate YAML | `/copilot-studio:validate` |
| Look up schema definitions | `/copilot-studio:lookup-schema` |
| List available kinds | `/copilot-studio:list-kinds` |
| List all topics | `/copilot-studio:list-topics` |
| Best practices (JIT, user context) | `/copilot-studio:best-practices` |
| Search known issues | `/copilot-studio:known-issues` |
| Scaffold new agent | `/copilot-studio:scaffold-agent` |
| Clone agent from cloud | `/copilot-studio:clone-agent` |
| Push/pull agent content | `/copilot-studio:manage-agent` |
| Test agent with a message | `/copilot-studio:chat-with-agent` |
| DirectLine testing | `/copilot-studio:directline-chat` |
| Run batch test suites | `/copilot-studio:run-tests` |
| Document decision | `/copilot-studio:add-decision-record` |
| Pre-push review gate | `/copilot-studio:pre-push-review` |
| Update project memory | `/copilot-studio:update-memory` |
| Orchestrate multi-agent workflow | `/copilot-studio:copilot-studio-conductor` |
| Coverage report | `/copilot-studio:coverage-report` |
| Configure BC MCP connection | `/copilot-studio:bc-mcp-setup` |
| Use a BC MCP action template | `/copilot-studio:bc-action-templates` |
| Create BC topic pattern | `/copilot-studio:bc-topic-patterns` |
| Write BC agent instructions | `/copilot-studio:bc-instructions-patterns` |
| Full BC agent blueprint | `/copilot-studio:bc-agent-blueprints` |

Only work manually if no skill matches; always validate afterward.

### Agent discovery
**Never hardcode the agent directory.** Always discover with `Glob: **/agent.mcs.yml`.

### ID generation
Format: `<nodeType>_<6-8 random alphanumeric>` — e.g., `sendMessage_g5Ls09`.  
Always replace `_REPLACE` placeholders in templates with fresh random IDs.

### Power Fx
Expressions start with `=`. String interpolation uses `{}`.  
Only use functions listed in the `int-reference` skill.

### Topic cross-references
Use the full schema name: `copilots_header_cra1e_Circe.topic.<TopicName>`

### Generative orchestration
`GenerativeActionsEnabled: true` — the orchestrator routes to MCP actions based on agent instructions. Only create custom topics when you need deterministic flows (Adaptive Cards, explicit branching) that the orchestrator cannot handle from instructions alone.

### Language
All trigger queries, messages, and user-facing labels must be in **Spanish**.

### Date context
Agent instructions use `{Text(Today(),DateTimeFormat.LongDate)}` to inject the current date at runtime (Power Fx). See `skills/best-practices/date-context.md`.

### Traceability
Every action executed (email, meeting, reminder) must be registered in BC with timestamp, user, and result.

### Protection rules (non-negotiable)
Never send external communications when an active dispute or material ledger inconsistency (>5 % or >1 000 €) exists. Generate an internal reconciliation report instead. See `agent.mcs.yml` for the full rule set.

---

## Agents

Use specialist sub-agents for specific tasks:

| Agent | Use for |
|-------|---------|
| `@copilot-studio-author` | Create/edit topics, actions, variables, Adaptive Cards |
| `@copilot-studio-manage` | Push/pull agent content, ALM operations |
| `@copilot-studio-test` | Run test suites, validate topic behaviour |
| `@copilot-studio-troubleshoot` | Debug errors, fix broken YAML |
| `@copilot-studio-conductor` | Orchestrate multi-step, multi-agent workflows with HITL gates |
