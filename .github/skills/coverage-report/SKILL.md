---
user-invocable: false
name: coverage-report
description: >
  Analyzes the Copilot Studio agent and generates a quality/coverage report.
  Reports on topic test coverage, trigger phrase density, knowledge source
  validation, model description completeness, and YAML validation status.
  Integrated with pre-push review.
---

# Coverage Report

Internal skill that inventories the agent, analyzes every component, runs
YAML validation, computes quality metrics, and generates a formatted report.

Coverage is **ADVISORY** — it never blocks push or other operations.

## When this skill is invoked

- Automatically by the **pre-push-review** skill between validation and DR collection.
- Automatically by the **Conductor** at the end of a Full Cycle workflow.
- Manually by any agent when the user asks about test coverage, quality, or agent health.

## Instructions

### Step 1 — Discover and inventory

```
Glob: **/agent.mcs.yml
```

Set `AGENT_DIR` to the directory containing `agent.mcs.yml`.

Then discover all components:

```
Glob: <AGENT_DIR>/topics/*.topic.mcs.yml      → TOPICS
Glob: <AGENT_DIR>/actions/*.mcs.yml            → ACTIONS
Glob: <AGENT_DIR>/knowledge/*.mcs.yml          → KNOWLEDGE
Glob: <AGENT_DIR>/variables/*.mcs.yml          → VARIABLES
Glob: <AGENT_DIR>/agents/*/agent.mcs.yml       → CHILD_AGENTS
Glob: <AGENT_DIR>/tests/*                      → TEST_FILES
Glob: tests/*                                  → TEST_FILES (also check workspace root)
```

Read `agent.mcs.yml` to extract the agent display name.

If a glob returns no results, that component count is 0 — this is normal.

### Step 2 — Analyze each topic

For each file in `TOPICS`, read it and extract:

| Field | How to find |
|-------|------------|
| **Trigger type** | Look for `kind:` under the trigger node — `OnRecognizedIntent`, `OnConversationStart`, `OnError`, `OnUnknownIntent`, etc. |
| **Trigger phrases** | Count items in `triggerQueries:` array (only for `OnRecognizedIntent`) |
| **Model description** | Check if `modelDescription:` is present and non-empty (only for `OnRecognizedIntent`) |
| **Action nodes** | Count nodes with `kind: InvokeConnectorAction` or `kind: InvokeFlowAction` |
| **Has tests** | Match the topic file name (without `.topic.mcs.yml`) against test file names in `TEST_FILES` (case-insensitive, partial match) |

Classify each topic:

| Category | Condition |
|----------|-----------|
| **Intentful** | Trigger type is `OnRecognizedIntent` |
| **System** | Trigger type is `OnConversationStart`, `OnError`, `OnUnknownIntent`, or topic name matches system topics (Fallback, Escalate, OnError, ConversationStart, EndofConversation, Greeting, Goodbye, ThankYou, StartOver, ResetConversation, MultipleTopicsMatched, Signin, Search) |

### Step 3 — Analyze knowledge sources

For each file in `KNOWLEDGE`, read it and extract:

| Field | How to find |
|-------|------------|
| **Type** | `kind:` value — `PublicSite`, `SharePoint`, `GraphConnector`, `Dataverse`, etc. |
| **Trigger condition** | Check if `triggerCondition:` is present |
| **Source URL** | Check the URL/site field is present and non-blank |

If `KNOWLEDGE` is empty, skip this section and report 0 knowledge sources.

### Step 4 — Validate all YAML

Run the schema validator on every `.mcs.yml` file in `AGENT_DIR`:

```bash
node .github/scripts/schema-lookup.bundle.js validate <file>
```

Collect results per file: **PASS** or **FAIL** (with error summary).

Count total files and passing files.

### Step 5 — Calculate metrics

| Metric | Calculation | Thresholds |
|--------|------------|------------|
| **Topic test coverage** | Topics with tests ÷ Total intentful topics | ✅ ≥ 80% · ⚠️ 40–79% · ❌ < 40% |
| **Trigger phrase density** | Average trigger phrases per intentful topic | ✅ ≥ 5 · ⚠️ 3–4 · ❌ < 3 |
| **Model description coverage** | Intentful topics with modelDescription ÷ Total intentful topics | ✅ 100% · ⚠️ 50–99% · ❌ < 50% |
| **Knowledge source health** | Sources with non-blank URL ÷ Total sources | ✅ 100% · ⚠️ 50–99% · ❌ < 50% |
| **YAML validation rate** | Files passing validation ÷ Total .mcs.yml files | ✅ 100% · ⚠️ 80–99% · ❌ < 80% |
| **Decision record coverage** | Components with DRs ÷ Total components in circe-memory.md | ✅ ≥ 50% · ⚠️ 25–49% · ❌ < 25% |

**Special cases:**
- If there are **0 intentful topics**, topic test coverage and trigger phrase density are `N/A` (not ⚠️ or ❌).
- If there are **0 knowledge sources**, knowledge source health is `N/A`.
- If `circe-memory.md` doesn't exist, decision record coverage is `N/A`.
- If there are **0 test files**, topic test coverage is `0%` with ⚠️ (not ❌ — zero tests is a starting point, not a critical failure).

### Step 6 — Generate report

Present the report in this exact format:

```
═══════════════════════════════════════════
  CIRCE COVERAGE REPORT
  Agent: {display name}
  Date: {today}
═══════════════════════════════════════════

📊 Summary
  Topics:           {N} ({M} with OnRecognizedIntent)
  Actions:          {N}
  Knowledge Sources: {N}
  Variables:        {N}
  Child Agents:     {N}

📋 Metrics
  Topic test coverage:         {X}% {✅|⚠️|❌|N/A}
  Trigger phrase density:      {X} avg {✅|⚠️|❌|N/A}
  Model description coverage:  {X}% {✅|⚠️|❌|N/A}
  Knowledge source health:     {X}% {✅|⚠️|❌|N/A}
  YAML validation rate:        {X}% {✅|⚠️|❌}
  Decision record coverage:    {X}% {✅|⚠️|❌|N/A}

⚠️  Gaps Found
  • Topic "PaymentReminder" has no tests
  • Topic "Greeting" has only 2 trigger phrases (min: 5)
  • Knowledge source "hr-docs" has blank URL

═══════════════════════════════════════════
```

**Gaps section rules:**
- List every intentful topic without tests
- List every intentful topic with < 5 trigger phrases (show actual count)
- List every intentful topic missing `modelDescription`
- List every knowledge source with a blank URL
- List every YAML file that failed validation (with 1-line error summary)
- If there are no gaps, show: `✅ No gaps found — all metrics within thresholds`

### Step 7 — Update memory (optional)

If `circe-memory.md` exists in `AGENT_DIR`, append or update a `## Coverage Metrics`
section with the latest metrics and date:

```markdown
## Coverage Metrics

Last run: {YYYY-MM-DD}

| Metric | Value | Status |
|--------|-------|--------|
| Topic test coverage | {X}% | {✅|⚠️|❌|N/A} |
| Trigger phrase density | {X} avg | {✅|⚠️|❌|N/A} |
| Model description coverage | {X}% | {✅|⚠️|❌|N/A} |
| Knowledge source health | {X}% | {✅|⚠️|❌|N/A} |
| YAML validation rate | {X}% | {✅|⚠️|❌} |
| Decision record coverage | {X}% | {✅|⚠️|❌|N/A} |
```

If the section already exists, **replace** it with the new data.
If `circe-memory.md` doesn't exist, skip this step silently.

## Integration with pre-push-review

When invoked from pre-push-review, return a **compact summary** (not the full report)
for inclusion in the review output:

```
📊 Coverage: tests {X}%, triggers {X} avg, descriptions {X}%, validation {X}%
   {⚠️ N gaps found — see coverage-report for details | ✅ No gaps}
```

If any metric is ❌ (critical), add a WARNING line but do **NOT** block the push:

```
⚠️  COVERAGE WARNING: {metric} is below critical threshold ({X}%)
    This is advisory — push is NOT blocked.
```

## Notes

- Coverage is **ADVISORY** — it never blocks push or other operations
- The report must be fast — read files and count, no external API calls
- Works with zero tests configured (all test metrics show 0% with ⚠️)
- Works with zero knowledge sources (knowledge metrics show N/A)
- Works with zero intentful topics (topic metrics show N/A)
- Thresholds are hardcoded for now; future versions may make them configurable
- Uses the same visual style (═══ borders, emoji indicators) as pre-push-review
