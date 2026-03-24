---
user-invocable: false
name: add-decision-record
description: >
  Creates a lightweight architecture decision record. Invoked automatically by
  agents when making significant design decisions, or manually when the user
  wants to document a decision.
argument-hint: <title> <context> <decision>
---

# Add Decision Record

Creates a Lightweight Architecture Decision Record (LADR) and registers it in the
agent's `decisions/` directory and in `circe-memory.md`.

## When this skill is invoked

Agents call this skill after significant design decisions:

| Calling agent | When |
|--------------|------|
| **Author** | After creating a new topic, knowledge source, or child agent |
| **Troubleshoot** | After proposing and applying a fix |
| **Any agent** | When the user explicitly requests documenting a decision |

## Instructions

### 1. Gather input

The calling agent provides these arguments (ask if missing):

| Parameter | Required | Example |
|-----------|----------|---------|
| **Title** | Yes | `Greeting topic uses Adaptive Card` |
| **Context** | Yes | `User needs a rich welcome UX with quick-action buttons` |
| **Decision** | Yes | `Used AdaptiveCardPrompt node with hero image and 3 action buttons` |
| **Alternatives** | No | List of alternatives considered with pros/cons |
| **Agent** | Auto | Name of the calling agent (Author, Troubleshoot, etc.) |
| **Component** | No | The affected file or component path |

### 2. Auto-discover the agent directory

```
Glob: **/agent.mcs.yml
```

Set `AGENT_DIR` to the directory containing `agent.mcs.yml`.
**NEVER** hardcode the agent directory.

### 3. Ensure `decisions/` directory exists

The decisions directory lives inside the **agent directory** (alongside `topics/`, `actions/`, etc.):

```
<AGENT_DIR>/decisions/
```

Create it if it doesn't exist.

### 4. Determine the next DR number

List existing DRs:
```bash
ls <AGENT_DIR>/decisions/DR-*.md
```

Count the files and add 1 to get the next number.
Format: zero-padded 3 digits — `DR-001`, `DR-002`, etc.

If no files exist, the next number is `DR-001`.

### 5. Generate the slug

From the title:
1. Convert to lowercase
2. Replace spaces and non-alphanumeric characters with hyphens
3. Remove consecutive hyphens
4. Trim to max 40 characters
5. Remove trailing hyphens

Example: `Greeting topic uses Adaptive Card` → `greeting-topic-uses-adaptive-card`

### 6. Create the DR file

Use the template at `.github/templates/decision-record-template.md` as the base.

Save as: `<AGENT_DIR>/decisions/DR-{NNN}-{slug}.md`

Fill in the template fields:

| Template placeholder | Value |
|---------------------|-------|
| `{NNN}` | The zero-padded number from step 4 |
| `{Title}` | The title argument |
| `{YYYY-MM-DD}` | Today's date in ISO 8601 format |
| `Status` | Set to `Accepted` (default for agent-created DRs) |
| `{which agent made this decision}` | The calling agent name |
| `{affected file or component}` | The component argument, or `—` if not provided |
| `Context` section | The context argument |
| `Decision` section | The decision argument |
| `Alternatives Considered` table | Fill from alternatives argument; if none provided, write a single row: `| (ninguna evaluada) | — | — | — |` |
| `Consequences` section | Infer from the context and decision. Keep concise (1-2 sentences each) |

### 7. Update `circe-memory.md`

#### 7a. Locate or create `circe-memory.md`

Look for `circe-memory.md` in `AGENT_DIR` (same level as `agent.mcs.yml`).

- **If it does NOT exist**: Copy the template from `.github/templates/circe-memory-template.md`
  and save it as `circe-memory.md` in `AGENT_DIR`. Then proceed.
- **If it exists**: Read it and proceed.

#### 7b. Add entry to "Decisiones de Diseño" table

Add a new row:

```markdown
| YYYY-MM-DD | DR-{NNN} | {title} | [DR-{NNN}](decisions/DR-{NNN}-{slug}.md) |
```

#### 7c. Add session summary entry

Append to "Resumen de Última Sesión":

```markdown
- **YYYY-MM-DD HH:MM** — decision: DR-{NNN} {title}
```

Update the `**Última actualización:**` timestamp.

### 8. Report back

Return to the calling agent:

```
✅ Decision record created: DR-{NNN} — {title}
   Path: <AGENT_DIR>/decisions/DR-{NNN}-{slug}.md
   Memory updated: circe-memory.md
```

## Target document size

DRs should be concise: **150–300 words**. The template fields guide this:
- Context: 2-4 sentences
- Decision: 2-4 sentences
- Alternatives: 1-3 rows, brief
- Consequences: 1-2 sentences per category

## Notes

- DR files are stored in the **agent directory**, not in `.github/`
- All DRs use ISO 8601 dates
- The DR system works even if `circe-memory.md` doesn't exist yet (creates it from template)
- DRs are intended to be version-controlled alongside agent YAML files
- Status values: `Proposed`, `Accepted`, `Deprecated`, `Superseded by DR-{NNN}`
- Content language follows agent conventions (Spanish for Circe)
