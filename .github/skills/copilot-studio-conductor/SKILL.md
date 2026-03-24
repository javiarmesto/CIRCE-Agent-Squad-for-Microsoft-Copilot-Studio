---
name: copilot-studio-conductor
description: Orchestrate multi-step, multi-agent workflows with HITL gates.
argument-hint: <what to create or change>
---
> **Trigger**: User request spans multiple agents (e.g., "create a topic and deploy it",
> "fix the issue and verify it works", "scaffold an agent and configure knowledge").

## Purpose

Decomposes multi-step requests into sequenced tasks that are delegated to
specialist agents (Author, Manage, Test, Troubleshoot) with HITL gates between
each step.

## When to use this skill

- The user asks for a **multi-agent task** — creation + deployment, fix + verification, scaffold + configure.
- The user explicitly asks for a "workflow" or "full cycle".
- The task requires sequenced operations where the output of one step feeds the next.

## When NOT to use

- The task maps cleanly to a single specialist agent — delegate directly.
- The user only needs information or a schema lookup.

## Prerequisites

1. Read `circe-memory.md` (if it exists) — understand current project state.
2. Read `.github/circe-conventions.md` — load mandatory rules.

## Workflow Selection

Match the user's request to a predefined workflow:

| Pattern | Workflow |
|---------|----------|
| "create X and deploy" / "add X and push" | **Create & Deploy** |
| "fix X and test" / "debug X and verify" | **Fix & Verify** |
| "create X, deploy, and test" / "full cycle" | **Full Cycle** |
| "scaffold agent and configure" / "new agent from scratch" | **Scaffold & Configure** |
| "create a BC agent" / "collections agent" / "sales agent" / "support agent" | **BC Agent Setup** — read `/copilot-studio:bc-agent-blueprints` for the matching blueprint |
| None of the above | **Custom Workflow** — analyze and propose |

## Execution Protocol

### Step 1 — Present the plan

Before executing anything, present the plan with numbered steps, agent assignments,
and HITL gates. Wait for user confirmation.

### Step 2 — Execute sequentially

For each step:
1. Delegate to the specialist agent with full context (what, why, state, history, decisions)
2. Present the result including the agent's CIRCE-EVIDENCE block
3. Ask the user to proceed, modify, or abort
4. Only move to the next step after explicit approval

### Step 3 — Handle failures

If a step fails:
- Stop the workflow
- Present the error with recovery options (retry, fix, skip, abort)
- Wait for user choice

### Step 4 — Complete the workflow

After the final step:
1. Present a complete workflow summary with all evidencing
2. Invoke `/copilot-studio:update-memory` with the workflow results
3. List any decision records created during the workflow

## Delegation Template

When delegating to a specialist agent, provide:

```
Task: [specific action to perform]
Context: [user's original request and intent]
Project state: [from circe-memory.md — schema name, GenerativeActionsEnabled, etc.]
Previous steps: [what was already done in this workflow]
Decisions: [relevant DRs from this session]
```

## CIRCE-EVIDENCE (post-workflow)

```
── CIRCE EVIDENCE ──────────────────
Skills loaded: copilot-studio-conductor
Workflow executed: [workflow name] ([N] steps)
Agents delegated: [list]
HITL gates passed: [N/total]
Decision Records created: [list or "none"]
Memory updated: Sí/No
────────────────────────────────────
```
