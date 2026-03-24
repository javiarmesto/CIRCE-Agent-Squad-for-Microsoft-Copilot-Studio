---
user-invocable: false
name: update-memory
description: >
  Updates circe-memory.md after significant operations. Invoked by other agents,
  not directly by users. Maintains a persistent cross-session memory file at the
  agent root with configuration, components, knowledge sources, decisions, and
  session summaries.
argument-hint: <operation-type> <component-name> <rationale> [evidencing]
---

# Update Memory

Internal skill that maintains `circe-memory.md` — a persistent cross-session memory file
that tracks agent configuration, created components, knowledge sources, design decisions,
and session activity.

## When this skill is invoked

Other agents call this skill after completing significant operations:

| Operation type | Trigger | Section updated |
|---------------|---------|-----------------|
| `topic-created` | After creating a new topic | Componentes Creados |
| `topic-modified` | After modifying an existing topic | Componentes Creados (nota) |
| `knowledge-added` | After adding a knowledge source | Fuentes de Conocimiento |
| `action-added` | After adding a connector action | Componentes Creados |
| `variable-added` | After adding a global variable | Componentes Creados |
| `agent-edited` | After editing agent settings/instructions | Configuración del Agente + Resumen |
| `decision` | When a design decision is made | Decisiones de Diseño |
| `push` | After pushing to the cloud | Resumen de Última Sesión |
| `test-run` | After running tests | Resumen de Última Sesión |

## Instructions

### 1. Auto-discover the agent directory

```
Glob: **/agent.mcs.yml
```

NEVER hardcode the agent directory. The memory file lives at the **same level** as `agent.mcs.yml`.

### 2. Check if `circe-memory.md` exists

Look for `circe-memory.md` in the agent root directory (same level as `agent.mcs.yml`).

- **If it does NOT exist**: Copy the template from `.github/templates/circe-memory-template.md`
  and save it as `circe-memory.md` in the agent root. Then proceed to step 3.
- **If it exists**: Read it and proceed to step 3.

### 3. Populate Agent Configuration (if empty)

If the "Configuración del Agente" table has placeholder values (`<!-- ... -->`), populate it:

1. Read `agent.mcs.yml` and extract:
   - `mcs.metadata.componentName` → **Display Name**
2. Read `settings.mcs.yml` and extract:
   - `schemaName` → **Schema Name**
   - `GenerativeActionsEnabled` → **Generative Actions** (true/false)
3. Read `connectionreferences.mcs.yml` (if it exists) and extract configured connections
   → update **Conexiones Activas (MCP)** section

The **Entorno** field should be extracted from `.mcs/conn.json` if available, otherwise
leave it for the user to fill.

### 4. Update the appropriate section based on operation-type

#### `topic-created`

Add a new row to **Componentes Creados**:

```markdown
| YYYY-MM-DD | <component-name> | Topic | <rationale> |
```

#### `topic-modified`

If the component already exists in **Componentes Creados**, append a note to its
Justificación cell: `(modificado YYYY-MM-DD: <rationale>)`.
If it doesn't exist, add a new row with type "Topic" and note "(modificación)".

#### `knowledge-added`

Add a new row to **Fuentes de Conocimiento**:

```markdown
| <source-url-or-name> | <PublicSite/SharePoint/GraphConnector> | <scope> | <triggerCondition or "—"> | Activo |
```

#### `action-added`

Add a new row to **Componentes Creados**:

```markdown
| YYYY-MM-DD | <component-name> | Action | <rationale> |
```

#### `variable-added`

Add a new row to **Componentes Creados**:

```markdown
| YYYY-MM-DD | <component-name> | Variable | <rationale> |
```

#### `agent-edited`

1. Update **Configuración del Agente** table values if they changed
2. Add entry to **Resumen de Última Sesión**

#### `decision`

Add a new row to **Decisiones de Diseño**:

```markdown
| YYYY-MM-DD | DR-NNN | <decision summary> | <!-- enlace pendiente --> |
```

Auto-increment the DR number by reading the last DR-NNN in the table.

#### `push`

Add to **Resumen de Última Sesión**:

```markdown
- **YYYY-MM-DD HH:MM** — Push realizado. Archivos: <list of files pushed>
```

#### `test-run`

Add to **Resumen de Última Sesión**:

```markdown
- **YYYY-MM-DD HH:MM** — Tests ejecutados. Resultado: <pass/fail summary>
```

### 5. Always update the session summary

Regardless of operation type, append a timestamped line to **Resumen de Última Sesión**:

```markdown
- **YYYY-MM-DD HH:MM** — <operation-type>: <component-name> — <rationale>
```

If the optional `evidencing` argument is provided, append the CIRCE-EVIDENCE block
immediately after the component entry in **Componentes Creados** as a collapsed detail:

```markdown
  <details><summary>Evidence</summary>

  ```
  ── CIRCE EVIDENCE ────────────────────
  Skills loaded: ...
  Template used: ...
  Patterns applied: ...
  Validation: ...
  Decision Record: ...
  Memory updated: ...
  ────────────────────────────────────────
  ```

  </details>
```

Update the `**Última actualización:**` line at the bottom with the current timestamp.

### 6. Validate the markdown structure

After editing, verify:
- All table headers still have the correct number of columns
- No broken markdown syntax
- The file still has all 6 required sections:
  1. Configuración del Agente
  2. Componentes Creados
  3. Fuentes de Conocimiento
  4. Decisiones de Diseño
  5. Conexiones Activas (MCP)
  6. Resumen de Última Sesión

## Notes

- **No external dependencies** — this skill only reads and writes markdown files
- **Dates** use ISO 8601 format: `YYYY-MM-DD`
- **Timestamps** use `YYYY-MM-DD HH:MM` (24h, local timezone)
- The memory file is intended to be checked into version control alongside the agent YAML
- All content must be in **Spanish** (column headers, labels, status values)
