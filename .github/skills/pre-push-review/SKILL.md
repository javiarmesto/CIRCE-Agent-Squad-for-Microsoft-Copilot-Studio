---
user-invocable: false
name: pre-push-review
description: >
  Mandatory review gate before pushing changes. Pulls latest state, validates all
  modified files, generates change summary with associated decision records, and
  waits for explicit user approval before proceeding with push.
---

# Pre-Push Review

Mandatory review gate that must run before every push operation. Validates changed
files, collects decision records, presents a formatted summary, and blocks until
the user gives explicit approval.

## When this skill is invoked

The Manage Agent invokes this skill **every time** the user requests a push.
Direct push without the review gate is not allowed unless the user explicitly
requests `--force` after receiving a warning.

## Instructions

### 1. Auto-discover the agent directory

```
Glob: **/agent.mcs.yml
```

Set `AGENT_DIR` to the directory containing `agent.mcs.yml`.
**NEVER** hardcode the agent directory.

### 2. Fresh state — Pull latest

Run `/copilot-studio:manage-agent pull` to get the latest remote state.
This ensures row versions are fresh and the diff is accurate.

If pull fails (e.g., auth error, network issue), **STOP** and report the error.
Do not proceed with the review without a fresh pull.

### 3. Detect changes

Run `/copilot-studio:manage-agent changes` to get the list of modified, added,
and deleted files.

Parse the output and categorize each file:
- `+` → new file (added)
- `~` → modified file
- `-` → deleted file

If **no changes** are detected, report to the user:

```
No changes detected between local and remote. Nothing to push.
```

Then **STOP**. Do not proceed.

### 4. Validate modified/added YAML files

For each `.mcs.yml` file that is new or modified, run:

```bash
node .github/scripts/schema-lookup.bundle.js validate <file>
```

Collect results per file: **PASS** or **FAIL** (with error details).

**If ANY file fails validation → BLOCK the push.** Do not present the confirmation
prompt. Instead, report:

```
❌ PUSH BLOCKED — Validation failed

  ✗ topics/payment-reminder.topic.mcs.yml — <error details>
  ✓ settings.mcs.yml

Fix the validation errors before pushing. Use `/copilot-studio:validate` for details.
```

Only proceed to step 5 if **all files pass**.

### 5. Run coverage report (advisory)

Invoke `/copilot-studio:coverage-report` to analyze the agent.

This step is **advisory** — it never blocks the push. Include the compact summary
in the review output (step 7). If any metric is ❌ (critical), add a WARNING line
but do NOT block.

If the coverage-report skill is unavailable or fails, skip this step silently and
proceed to step 6.

### 6. Collect decision records since last push

Read `circe-memory.md` from `AGENT_DIR` (if it exists).

1. Find the last push timestamp in the "Resumen de Última Sesión" section
   (look for lines containing `Push realizado` or `push:`)
2. Read the "Decisiones de Diseño" table
3. Filter DRs whose date is **on or after** the last push date
4. If no last push timestamp exists, collect **all** DRs

If `circe-memory.md` doesn't exist, skip this step (graceful fallback).
If the "Decisiones de Diseño" table is empty, skip this section in the summary.

### 7. Generate review summary

Present a formatted summary to the user:

```
═══════════════════════════════════════
  CIRCE PRE-PUSH REVIEW
═══════════════════════════════════════

📁 Changes:
  + topics/payment-reminder.topic.mcs.yml  (new)
  ~ settings.mcs.yml  (modified)
  - topics/old-topic.mcs.yml  (deleted)

✅ Validation: 2/2 files pass

📊 Coverage: tests 60%, triggers 7 avg, descriptions 100%, validation 100%
   ⚠️ 2 gaps found — see coverage-report for details

📋 Decision Records since last push:
  DR-003: Added payment reminder topic
  DR-004: Changed greeting card layout

═══════════════════════════════════════
Confirm push? [Yes / No / Show diff / Show DR details]
═══════════════════════════════════════
```

**Formatting rules:**
- Use `+` for added, `~` for modified, `-` for deleted
- Show validation count: `N/N files pass`
- List each DR with its number and title summary
- If no DRs exist since last push, show: `(no new decision records)`
- If `circe-memory.md` doesn't exist, omit the DR section entirely

### 8. Wait for user confirmation

The user must respond with one of:

| Response | Action |
|----------|--------|
| **Yes** | Proceed with push (step 9) |
| **No** | Abort. Explain: "Push cancelled. Make additional changes and run the review again when ready." |
| **Show diff** | Run `/copilot-studio:manage-agent changes` to display the detailed diff, then re-present the review summary |
| **Show DR details** | Read and display the full content of each DR file listed in step 6, then re-present the review summary |

**CRITICAL**: Never proceed without an explicit "Yes" from the user.

### 9. Execute push

After the user confirms with "Yes":

1. Run `/copilot-studio:manage-agent push`
2. If push fails, report the error and suggest resolution
3. If push succeeds, proceed to step 10

### 10. Post-push updates

After a successful push:

#### 10a. Update circe-memory.md

Invoke `/copilot-studio:update-memory` with:
- **operation-type**: `push`
- **component-name**: list of files pushed
- **rationale**: `Pre-push review aprobado por el usuario`

#### 10b. Remind the user

Always end with:

```
✅ Push completado. Archivos enviados como DRAFT.

⚠️  Para que los cambios sean visibles en canales externos (Teams, tests, DirectLine),
    es necesario PUBLICAR en la interfaz de Copilot Studio.
```

## Force push override

If the user explicitly requests `--force` or insists on skipping the review:

1. **Warn the user**:
   ```
   ⚠️  El review pre-push es obligatorio. Saltarlo puede subir archivos inválidos
       o cambios no documentados. ¿Confirmas forzar el push sin review? (Sí/No)
   ```
2. Only proceed if the user confirms again with an explicit "Sí" or "Yes"
3. Skip steps 3–7, go directly to push
4. Still run post-push updates (step 10)
5. Log the force push in circe-memory.md session summary:
   ```
   - **YYYY-MM-DD HH:MM** — ⚠️ Push forzado (sin review). Archivos: <list>
   ```

## Notes

- This skill is **never skippable silently** — it always requires explicit user action
- Validation failure is a **hard block**, not a warning
- The skill works gracefully when `circe-memory.md` or decision records don't exist yet
- All user-facing messages follow agent language conventions
- The review does not modify any YAML files — it is a read-only gate before push
