---
user-invocable: false
name: bc-agent-blueprints
description: Complete agent blueprints for typical Business Central scenarios — ready-to-implement specifications that combine all BC Extension Pack skills (MCP setup, instructions, actions, topics) into end-to-end agent definitions. Covers collections, sales assistant, and customer support agents.
argument-hint: "<blueprint: collections|sales-assistant|customer-support>"
---

# BC Agent Blueprints

Complete, ready-to-implement agent specifications for typical Business Central scenarios. Each blueprint combines all BC Extension Pack skills into a single, step-by-step implementation guide — from MCP setup through instructions, actions, topics, and validation.

## When to Use This Skill

Use when:
- The user asks to "create a BC agent", "build a collections agent", "set up a sales bot", or similar full-agent requests
- The `/copilot-studio:scaffold-agent` skill is invoked for a BC-specific agent type
- The user wants to understand what a complete BC agent looks like before building one
- The conductor agent needs a reference blueprint for a multi-step agent setup workflow

Do NOT use when:
- The user only needs one component (a single instruction block, action, or topic) — use the matching BC Extension Pack skill directly
- The agent is not BC-connected — use `/copilot-studio:scaffold-agent` directly
- The user wants to clone an existing agent — use `/copilot-studio:clone-agent`

## Available Blueprints

| Blueprint | File | Description |
|-----------|------|-------------|
| Collections Agent | `blueprints/collections-agent.md` | Full collections & payment recovery agent — aging analysis, dispute protection, payment reminders, escalation |
| Sales Assistant | `blueprints/sales-assistant.md` | Sales order management, quotes, item availability, account health monitoring |
| Customer Support | `blueprints/customer-support.md` | Incident management, order status inquiries, customer onboarding |

## How Blueprints Integrate the BC Extension Pack

Each blueprint references the full stack of BC Extension Pack skills:

```
┌─────────────────────────────────────────────────┐
│            BC Agent Blueprint                    │
│  (this skill — end-to-end specification)         │
├─────────────────────────────────────────────────┤
│ 1. MCP Setup         → /bc-mcp-setup            │
│ 2. Instructions      → /bc-instructions-patterns │
│ 3. Actions           → /bc-action-templates      │
│ 4. Topics            → /bc-topic-patterns        │
│ 5. Scaffold          → /scaffold-agent           │
│ 6. Knowledge (opt)   → /add-knowledge            │
│ 7. Validation        → /validate                 │
└─────────────────────────────────────────────────┘
```

## Instructions

### 1. Determine the Blueprint

Match the user's request to a blueprint:

| User Says | Blueprint |
|-----------|-----------|
| "collections agent", "cobros", "payment recovery", "aging" | `collections` |
| "sales agent", "ventas", "orders", "quotes", "availability" | `sales-assistant` |
| "support agent", "soporte", "incidents", "service", "customer service" | `customer-support` |
| unclear | Show the blueprints table and ask |

### 2. Read the Blueprint

```
Read: ${CLAUDE_SKILL_DIR}/blueprints/<blueprint-name>.md
```

### 3. Present the Implementation Plan

Present the blueprint as a numbered implementation plan. Each blueprint has 6–8 phases. Show the user:
- Total phases and estimated components
- Which BC Extension Pack skills will be invoked
- What the user needs to provide (display name, schema prefix, environment details)

Wait for user confirmation before proceeding.

### 4. Execute Phase by Phase

Follow the blueprint's phases sequentially. For each phase:

1. **Invoke the matching skill** — each phase maps to a specific BC Extension Pack skill
2. **Present the result** to the user
3. **Wait for approval** before moving to the next phase (HITL gate)

### 5. Customization

Blueprints are starting points. During execution, adjust based on user preferences:
- Add or remove actions based on which BC MCP tools are needed
- Add or skip custom topics based on the decision matrix in `/bc-topic-patterns`
- Adjust instruction blocks based on the user's domain language and tone
- Add or skip knowledge sources

### 6. Post-Implementation

After all phases are complete:
1. **Validate all files** with `/copilot-studio:validate`
2. **Generate a coverage report** with `/copilot-studio:coverage-report`
3. **Push** with `/copilot-studio:manage-agent` (through pre-push review)
4. **Test** with `/copilot-studio:chat-with-agent` or `/copilot-studio:run-tests`

## Conductor Workflow Integration

When the conductor agent receives a "create a BC agent" request, it should:

1. Read this skill to select the matching blueprint
2. Decompose the blueprint into conductor workflow steps
3. Delegate each step to the Author agent with full context
4. Enforce HITL gates between phases

The conductor workflow pattern is:

```
Workflow: BC Agent Setup — <Blueprint Name>
Steps:
  1. [Author]  Scaffold agent            → /scaffold-agent
  2. [User]    MCP setup (portal steps)  → /bc-mcp-setup
  3. [Author]  Apply instructions        → /bc-instructions-patterns
  4. [Author]  Add actions               → /bc-action-templates
  5. [Author]  Add topics (if needed)    → /bc-topic-patterns
  6. [Author]  Add knowledge (optional)  → /add-knowledge
  7. [Author]  Validate all files        → /validate
  8. [Manage]  Pre-push review + push    → /pre-push-review
  9. [Test]    Smoke test                → /chat-with-agent
```

## Important Notes

- **Blueprints do NOT replace individual skills** — they orchestrate them. Each phase delegates to the matching skill.
- **MCP setup requires portal access** — Phase 2 is always a user-guided step, not automated YAML generation.
- **Protection rules are non-negotiable** — all blueprints include the dispute/inconsistency checks from CIRCE conventions, both in instructions (Block 3) and in custom topic ConditionGroups.
- **Spanish content** — all user-facing text in every blueprint component is in Spanish per CIRCE conventions.
- **Each blueprint is a specification, not generated YAML** — blueprints describe what to build and which skills to invoke. The skills themselves generate the actual YAML.
