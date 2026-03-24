# Bitácora de cambios

## 2026-03-21

### Agentes (`.github/agents/`)

**Migración de `skills:` a `tools:` en los 4 agentes**

El atributo `skills:` no es soportado por VS Code — pertenece a Claude Code / GitHub Copilot CLI. VS Code usa `tools:` para declarar herramientas disponibles.

| Agente | `tools:` asignado | Motivo |
|--------|-------------------|--------|
| `copilot-studio-author` | `[read, edit, search, execute]` | Necesita leer, editar YAML, buscar ficheros y ejecutar scripts de validación |
| `copilot-studio-troubleshoot` | `[read, search, execute]` | Lee y valida YAML, ejecuta scripts — no edita directamente |
| `copilot-studio-manage` | `[read, search, execute]` | Ejecuta `manage-agent.bundle.js`, lee config |
| `copilot-studio-test` | `[read, search, execute]` | Ejecuta scripts de test, lee resultados |

**Corrección: `copilot-studio-test` actualizado a `[read, edit, search, execute]`**

La skill `run-tests` requiere Write/Edit (para crear `tests/settings.json`). Detectado al analizar `allowed-tools` de las skills vinculadas.

**Sección "Preload" añadida al cuerpo de cada agente**

Las skills que antes se inyectaban vía `skills:` ahora se referencian como instrucción explícita en el cuerpo markdown. Esto garantiza que el modelo las cargue antes de empezar cualquier tarea, sin depender del auto-descubrimiento.

| Agente | Skills referenciadas |
|--------|---------------------|
| `copilot-studio-author` | `int-project-context`, `int-reference` |
| `copilot-studio-troubleshoot` | `int-project-context`, `int-reference`, `known-issues`, `validate` |
| `copilot-studio-manage` | `int-project-context`, `manage-agent`, `clone-agent` |
| `copilot-studio-test` | `int-project-context` |

### Instrucciones del workspace (`.github/copilot-instructions.md`)

Creado fichero nuevo con:
- Tabla de arquitectura (path → propósito)
- Comandos de build/validación
- Regla skill-first con tabla completa de task → skill
- Convenciones: agent discovery, IDs, Power Fx, cross-references, generative orchestration
- Idioma: español obligatorio
- Trazabilidad y reglas de protección
- Tabla de sub-agentes disponibles
- Links a `README.md` y `CLAUDE.md` (sin duplicar contenido)

### Skills (`.github/skills/`)

**Eliminados atributos no soportados por VS Code en 21 SKILL.md**

| Atributo eliminado | Propósito original (Claude Code) | Skills afectadas |
|---|---|---|
| `allowed-tools` | Restringía herramientas disponibles para la skill | 21 skills |
| `context: fork` | Ejecutaba la skill en subproceso aislado | 16 skills |
| `agent:` | Vinculaba la skill a un agente específico | 16 skills |

**Motivo:** VS Code no soporta estos atributos. Los atributos válidos son: `argument-hint`, `compatibility`, `description`, `disable-model-invocation`, `license`, `metadata`, `name`, `user-invocable`.

**Impacto:**
- Las tools que cada agente necesita se determinaron analizando `allowed-tools` de sus skills vinculadas (vía `agent:`) y se asignaron al `tools:` del agente correspondiente
- La afinidad skill→agente se mantiene mediante las tablas task→skill en el cuerpo de cada agente
- Sin `context: fork`, la skill comparte contexto con el agente (comportamiento ya existente en VS Code)

**Skills editadas:** add-action, add-adaptive-card, add-generative-answers, add-global-variable, add-knowledge, add-node, add-other-agents, best-practices, chat-with-agent, clone-agent, directline-chat, edit-action, edit-agent, edit-triggers, known-issues, list-kinds, list-topics, lookup-schema, manage-agent, new-topic, run-tests, validate
