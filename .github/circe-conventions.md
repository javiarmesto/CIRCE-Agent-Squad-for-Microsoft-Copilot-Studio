# Circe — Mandatory Conventions

> Quick reference. Applies to ALL agents, skills, and operations.
> Full source: `skills/int-project-context` and `skills/int-reference`.

---

### 1. Agent Discovery

Always discover with `Glob: **/agent.mcs.yml`. **NEVER** hardcode paths or agent names.

### 2. ID Generation

Format: `<nodeType>_<6-8 random alphanumeric>` — e.g. `sendMessage_g5Ls09`.
Always replace `_REPLACE` placeholders in templates with fresh unique IDs.

### 3. Power Fx

- Expressions start with `=` (e.g. `value: =Topic.Count + 1`).
- String interpolation uses `{}` (e.g. `activity: "Hola {Topic.Name}"`).
- Only use supported functions — full list in `int-reference`. Key functions:
  - **Text**: `Text`, `Concatenate`, `Left`, `Right`, `Mid`, `Len`, `Substitute`, `Lower`, `Upper`, `Find`, `Split`, `EncodeHTML`
  - **Date**: `Now`, `Today`, `DateAdd`, `DateDiff`, `Text(Today(), DateTimeFormat.LongDate)`
  - **Logic**: `If`, `IsBlank`, `IsEmpty`, `Coalesce`, `Switch`, `Not`, `And`, `Or`
  - **Table**: `Filter`, `First`, `LookUp`, `CountRows`, `ForAll`, `Sort`, `AddColumns`
  - **Conversion**: `ParseJSON`, `JSON`, `Boolean`, `Decimal`, `Float`, `GUID`
- Do **NOT** use functions not listed in `int-reference`.

### 4. Topic Cross-References

Use the full schema name: `<schemaName>.topic.<TopicName>`.
For Circe: `copilots_header_cra1e_Circe.topic.<TopicName>`.

### 5. Mandatory Validation

After **any** YAML edit, run:
```bash
node .github/scripts/schema-lookup.bundle.js validate <file.mcs.yml>
```
Do not push without validating.

### 6. Skill-First Rule

**Always** invoke the matching skill. Never write YAML manually when a skill exists for the task. Skills contain correct templates, required fields, and schema validation.

If no skill exists, work manually but validate afterward.

### 7. Generative Orchestration

When `GenerativeActionsEnabled: true`:
- Use `AutomaticTaskInput` instead of `Question` nodes for topic inputs.
- Prefer topic outputs over `SendActivity` nodes for responses.
- Only create custom topics when deterministic flows are needed (Adaptive Cards, explicit branching).

### 8. Language

All user-facing content (trigger queries, messages, labels, Adaptive Cards) must be in **Spanish**.

### 9. Memory

- Read `circe-memory.md` at the start of each session (if it exists in the agent directory).
- Update with `/copilot-studio:update-memory` after every significant operation.
- The file lives at the same level as `agent.mcs.yml`.

### 10. Protection (Non-Negotiable)

NEVER send external communications if:
- An active dispute exists in BC for the customer
- There is a material inconsistency between ledger and aging (>5% or >€1,000)
- The customer is blocked in BC

Instead: generate an internal reconciliation report.

### 11. Skills Evidencing

After completing any creation or modification operation, declare a `CIRCE-EVIDENCE` block:

```
── CIRCE EVIDENCE ──────────────────
Skills loaded: [list of skills read]
Template used: [template file, if applicable]
Patterns applied: [design patterns chosen]
Validation: [schema validation result]
Decision Record: [DR-NNN if created]
Memory updated: [Yes/No]
────────────────────────────────────
```

**Mandatory for**: topic creation, action additions, knowledge source additions, agent settings modifications, troubleshooting fixes, test execution, and push operations.

**Optional for**: read-only queries (schema lookup, topic listing, etc.).
