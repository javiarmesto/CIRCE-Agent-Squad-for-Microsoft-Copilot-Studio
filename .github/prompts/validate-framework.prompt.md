---
description: "Run a complete CIRCE framework validation — checks every component against specifications and produces a structured scorecard. Use when: validate framework, audit CIRCE, check compliance, framework health check."
agent: "agent"
---

# CIRCE Framework Validation

Run a complete validation of the CIRCE framework implementation. Check every component created during the 8 gaps and the BC Extension Pack against the specifications. Report results in a structured format.

## Instructions

Read ALL files listed below. For each component, verify existence, structure, content quality, and integration. Do NOT skip any check. Present results using this format:

- `[PASS]` Component — what was verified
- `[WARN]` Component — issue found but not blocking
- `[FAIL]` Component — critical issue that must be fixed
- `[SKIP]` Component — file not found or not implemented yet

At the end, produce a summary scorecard.

---

## Phase 1: Foundation Checks

### 1.1 — Always-On Conventions (GAP 8)

Read `.github/circe-conventions.md` and verify:

- [ ] File exists at `.github/circe-conventions.md`
- [ ] Contains ≤500 words (count them)
- [ ] Includes ALL 9+ mandatory rules:
  1. Agent Discovery (glob pattern)
  2. ID Generation (format + _REPLACE)
  3. Power Fx (= prefix, {} interpolation)
  4. Cross-References (full schema name)
  5. Validation (schema-lookup after edit)
  6. Skill-First Rule
  7. Generative Orchestration (AutomaticTaskInput vs Question)
  8. Language (Spanish for user-facing)
  9. Memory (read at start, update after ops)
- [ ] Format is scannable (numbered reference card, not prose)

### 1.2 — Hooks Update (GAP 8)

Read `.github/hooks/hooks.json` and verify:

- [ ] Valid JSON (parse it)
- [ ] SessionStart hook exists
- [ ] Contains instruction to read `circe-conventions.md` BEFORE delegation
- [ ] Original delegation instruction to sub-agents is preserved
- [ ] Conventions instruction comes FIRST in the hook chain

### 1.3 — Cross-Session Memory (GAP 5)

Check for memory template:

- [ ] `.github/templates/circe-memory-template.md` exists
- [ ] Contains sections: Agent Configuration, Components Created, Knowledge Sources, Design Decisions, Active Connections, Last Session Summary
- [ ] Tables have correct column headers

Check for update-memory skill:

- [ ] `.github/skills/update-memory/SKILL.md` exists
- [ ] Has `user-invocable: false` in frontmatter
- [ ] Supports operation types: topic-created, topic-modified, knowledge-added, action-added, variable-added, agent-edited, decision, push, test-run
- [ ] References auto-discovery via `Glob: **/agent.mcs.yml`

### 1.4 — Agent Scaffold (GAP 6)

Check skill:

- [ ] `.github/skills/scaffold-agent/SKILL.md` exists
- [ ] Has `user-invocable: false`
- [ ] Documents the full directory structure it generates (agent.mcs.yml, settings.mcs.yml, connectionreferences.mcs.yml, topics/greeting, topics/fallback, topics/error-handler, empty dirs for actions/knowledge/variables/agents)
- [ ] References existing templates in `.github/templates/`
- [ ] Includes circe-memory.md initialization
- [ ] Includes validation step with schema-lookup

Check Author Agent update:

- [ ] Read `.github/agents/copilot-studio-author.md`
- [ ] "Option 3 — Scaffold" exists in the "Check for existing agent" section
- [ ] References `/copilot-studio:scaffold-agent`

---

## Phase 2: Quality & Control Checks

### 2.1 — Decision Records (GAP 2)

Check template:

- [ ] `.github/templates/decision-record-template.md` exists
- [ ] Contains fields: Date, Status, Agent, Component, Context, Decision, Alternatives (table), Consequences

Check skill:

- [ ] `.github/skills/add-decision-record/SKILL.md` exists
- [ ] Has `user-invocable: false`
- [ ] Documents: auto-discovery, directory creation, DR numbering (DR-NNN), slug generation, memory update

Check integrations:

- [ ] Author Agent mentions invoking `add-decision-record` after creating topics/knowledge/child agents
- [ ] Troubleshoot Agent mentions invoking `add-decision-record` after applying fixes

### 2.2 — HITL Gates (GAP 3)

Check skill:

- [ ] `.github/skills/pre-push-review/SKILL.md` exists
- [ ] Has `user-invocable: false`
- [ ] Documents 7-step flow: pull → changes → validate → collect DRs → generate summary → wait approval → post-push
- [ ] Summary format includes: file changes, validation results, decision records, coverage (optional)
- [ ] User options: Yes / No / Show diff / Show DR details
- [ ] Validation failure BLOCKS push (not just warns)
- [ ] Post-push updates circe-memory.md

Check Manage Agent update:

- [ ] Read `.github/agents/copilot-studio-manage.md`
- [ ] Push workflow references pre-push-review as MANDATORY
- [ ] Task→skill table includes pre-push-review
- [ ] `--force` bypass documented with warning

Check manage-agent skill update:

- [ ] Read `.github/skills/manage-agent/SKILL.md`
- [ ] Push section references pre-push-review before executing

### 2.3 — Skills Evidencing (GAP 4)

Check conventions update:

- [ ] Read `.github/circe-conventions.md`
- [ ] Contains rule #10 about Skills Evidencing
- [ ] Defines the CIRCE-EVIDENCE block format with all fields: Skills loaded, Template used, Patterns applied, Validation, Decision Record, Memory updated

Check all 4 agents have evidencing sections:

- [ ] `.github/agents/copilot-studio-author.md` — has "Skills Evidencing" section with format and example
- [ ] `.github/agents/copilot-studio-test.md` — has evidencing section (test-specific)
- [ ] `.github/agents/copilot-studio-troubleshoot.md` — has evidencing section (diagnostic-specific)
- [ ] `.github/agents/copilot-studio-manage.md` — has evidencing section (ALM-specific)

---

## Phase 3: Automation Checks

### 3.1 — Conductor Agent (GAP 1)

Check agent:

- [ ] `.github/agents/copilot-studio-conductor.md` exists
- [ ] Frontmatter has: name, description (with USE FOR / DO NOT USE FOR), tools: [read, search, execute]
- [ ] Preload section reads circe-memory.md and circe-conventions.md
- [ ] Predefined workflows documented: Create & Deploy, Fix & Verify, Full Cycle, Scaffold & Configure
- [ ] Each workflow has HITL gates between steps
- [ ] Custom workflow support documented (analyze → propose → approve → execute)
- [ ] Delegation format documented (full context passing to specialist agents)
- [ ] Does NOT have edit tool (never edits YAML directly)

Check delegation skill:

- [ ] `.github/skills/copilot-studio-conductor/SKILL.md` exists
- [ ] Delegates to `@copilot-studio:copilot-studio-conductor`

Check hooks update:

- [ ] `.github/hooks/hooks.json` mentions conductor for multi-step requests

Check copilot-instructions.md:

- [ ] Read `.github/copilot-instructions.md`
- [ ] Agents table includes conductor
- [ ] Multi-step workflows section exists

### 3.2 — Coverage Metrics (GAP 7)

Check skill:

- [ ] `.github/skills/coverage-report/SKILL.md` exists
- [ ] Has `user-invocable: false`
- [ ] Documents 7-step flow: discover → analyze topics → analyze knowledge → validate YAML → calculate metrics → generate report → update memory
- [ ] Metrics defined with thresholds: topic test coverage (≥80%), trigger phrase density (≥5), model description coverage (100%), knowledge source health (100%), YAML validation rate (100%), decision record coverage (≥50%)
- [ ] Report format uses visual separators and emoji indicators (✅ ⚠️ ❌)
- [ ] Coverage is ADVISORY (never blocks push)

Check integration:

- [ ] pre-push-review skill references coverage-report
- [ ] Test Agent's task→skill table includes coverage-report
- [ ] copilot-instructions.md skill table includes coverage-report

---

## Phase 4: BC Extension Pack Checks

### 4.1 — bc-mcp-setup

- [ ] `.github/skills/bc-mcp-setup/SKILL.md` exists
- [ ] Has `user-invocable: false`
- [ ] Documents 6 phases: prerequisites → enable feature → configure MCP → connect Copilot Studio → validate → pull to local
- [ ] API page recommendations by agent type (collections, sales, support)
- [ ] Auth modes documented: delegated vs service-to-service
- [ ] Troubleshooting table with common issues
- [ ] `mcp-config-guide.md` companion file exists

### 4.2 — bc-instructions-patterns

- [ ] `.github/skills/bc-instructions-patterns/SKILL.md` exists
- [ ] Has `user-invocable: false`
- [ ] Contains 7 composable blocks: BC Data Context, Data Formatting, Protection Rules, Traceability, Collections, Sales, Support
- [ ] Composition rules table (agent type → blocks)
- [ ] Pre-composed files exist for each agent type:
  - [ ] `collections-agent-instructions.md`
  - [ ] `sales-agent-instructions.md`
  - [ ] `support-agent-instructions.md`
  - [ ] `general-bc-instructions.md`

### 4.3 — bc-action-templates

- [ ] `.github/skills/bc-action-templates/SKILL.md` exists
- [ ] Has `user-invocable: false`
- [ ] Contains or references 6 templates:
  - [ ] query-customer-balance
  - [ ] get-overdue-invoices
  - [ ] create-customer-payment (with confirmation warning in modelDescription)
  - [ ] get-sales-orders
  - [ ] create-support-incident
  - [ ] get-item-availability
- [ ] Templates have: modelDisplayName (Spanish), modelDescription (orchestrator guidance), AutomaticTaskInput with descriptions, inputType/outputType
- [ ] Usage notes explain operationId depends on MCP config

### 4.4 — bc-topic-patterns

- [ ] `.github/skills/bc-topic-patterns/SKILL.md` exists
- [ ] Has `user-invocable: false`
- [ ] Contains 4 patterns:
  - [ ] collections-flow (with dispute check as ConditionGroup, NOT just instructions)
  - [ ] order-inquiry (with note about orchestrator-only option)
  - [ ] incident-management (with AdaptiveCard reference)
  - [ ] customer-onboarding (with JIT user context integration)
- [ ] Decision matrix: when orchestrator-only vs custom topic
- [ ] Template files exist:
  - [ ] `collections-reminder.topic.mcs.yml`
  - [ ] `customer-lookup.topic.mcs.yml`

### 4.5 — bc-agent-blueprints

- [ ] `.github/skills/bc-agent-blueprints/SKILL.md` exists
- [ ] Has `user-invocable: false`
- [ ] Contains 3 blueprints:
  - [ ] collections-agent (MCP config, instructions blocks 1+2+3+4+5, topics, actions, protection rules, credits estimate)
  - [ ] sales-assistant (blocks 1+2+3+4+6)
  - [ ] customer-support (blocks 1+2+3+4+7)
- [ ] Each blueprint references other bc-* skills by name
- [ ] Conductor integration: BC Agent Setup workflow documented

---

## Phase 5: Integration & Consistency Checks

### 5.1 — copilot-instructions.md

Read `.github/copilot-instructions.md` and verify:

- [ ] "Always-On Conventions" section exists at the TOP
- [ ] Skill table includes ALL new skills: update-memory, add-decision-record, pre-push-review, coverage-report, scaffold-agent, bc-mcp-setup, bc-instructions-patterns, bc-action-templates, bc-topic-patterns, bc-agent-blueprints
- [ ] Agents table includes conductor
- [ ] "Business Central Integration" section exists with Giorgio Ughini / CAT team credit
- [ ] "Multi-Step Workflows" section exists

### 5.2 — Agent Cross-References

For each of the 5 agents, verify their task→skill table is complete:

- [ ] Author: includes scaffold-agent, add-decision-record, update-memory, all bc-* skills
- [ ] Manage: includes pre-push-review, update-memory
- [ ] Test: includes coverage-report, update-memory
- [ ] Troubleshoot: includes add-decision-record, bc-mcp-setup (for connection issues)
- [ ] Conductor: references all agents and all workflow skills

### 5.3 — Memory Integration

Verify that ALL agents reference memory:

- [ ] Each agent's preload section includes "Read circe-memory.md" as FIRST item
- [ ] Each agent's body includes "invoke update-memory after successful operations"

### 5.4 — Evidencing Consistency

Verify the CIRCE-EVIDENCE block format is identical across all agents:

- [ ] Same field names: Skills loaded, Template used, Patterns applied, Validation, Decision Record, Memory updated
- [ ] Same visual format (── borders)
- [ ] Mandatory for creation/modification operations in all agents

### 5.5 — README Accuracy

Read the project root `README.md` and verify:

- [ ] Agent count matches (5 agents)
- [ ] Skill count is accurate (count all skills in .github/skills/)
- [ ] Architecture diagram reflects actual structure
- [ ] BC Extension Pack section matches actual skills
- [ ] Circe reference implementation section matches actual agent configuration
- [ ] Giorgio Ughini and Power CAT team credited in Acknowledgements
- [ ] Project structure tree matches actual filesystem

---

## Scorecard

After completing ALL checks, produce this summary:

```
═══════════════════════════════════════════════════
  CIRCE FRAMEWORK VALIDATION REPORT
  Date: {today}
═══════════════════════════════════════════════════
  Phase 1 — Foundations:      {X}/{Y} checks passed
  Phase 2 — Quality:          {X}/{Y} checks passed
  Phase 3 — Automation:       {X}/{Y} checks passed
  Phase 4 — BC Extension:     {X}/{Y} checks passed
  Phase 5 — Integration:      {X}/{Y} checks passed

  TOTAL: {X}/{Y} ({percentage}%)

  PASS:  {count}
  WARN:  {count}
  FAIL:  {count}
  SKIP:  {count}

  Status: {COMPLIANT | PARTIAL | NON-COMPLIANT}
  COMPLIANT = 0 FAIL, ≤5 WARN
  PARTIAL = 0 FAIL, >5 WARN or any SKIP
  NON-COMPLIANT = any FAIL
═══════════════════════════════════════════════════
```

If any FAIL items exist, list them with the specific file and issue.
If any SKIP items exist, list them and note whether they are expected (not yet implemented) or unexpected (should exist but missing).
For WARN items, suggest the fix but don't block.

Finally, if the framework passes validation, suggest: "Framework validated. Next step: clone a real agent and run a full Create & Deploy workflow with the Conductor to verify end-to-end behavior."
