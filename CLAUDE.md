# CIRCE — Architecture Decisions & Development Patterns

> This file captures the key architecture decisions and development patterns that guide the CIRCE framework.
> For the full project documentation, see [README.md](README.md).
> For mandatory conventions, see [.github/circe-conventions.md](.github/circe-conventions.md).

---

## Architecture Decisions

### AD-1: Skill-First Authoring

**Decision:** All agent modifications must go through predefined skills. Manual YAML editing is only permitted when no matching skill exists.

**Rationale:** Skills encode validated templates, required fields, and schema constraints that prevent common authoring errors. They also provide a consistent developer experience across all agents using the framework.

**Trade-off:** Slightly higher initial setup cost (creating skills), but significantly reduced errors and faster onboarding for new contributors.

### AD-2: Multi-Agent Specialization

**Decision:** Split responsibilities across five specialist agents (Author, Manage, Test, Troubleshoot, Conductor) rather than a single monolithic agent.

**Rationale:** Each agent loads only the skills it needs, keeping context focused and reducing prompt size. The Conductor orchestrates multi-step workflows that span agents while enforcing HITL gates between steps.

**Trade-off:** Requires delegation logic (hooks.json) and clear boundary definitions, but improves reliability and makes each agent independently testable.

### AD-3: Cross-Session Memory

**Decision:** Maintain a `circe-memory.md` file at the agent root level, updated after every significant operation via the `update-memory` skill.

**Rationale:** AI coding assistants lose context between sessions. The memory file provides continuity — tracking components, decisions, configuration, and session summaries so the next session starts with full project awareness.

**Trade-off:** Requires discipline to update (enforced by conventions and evidencing). The file grows over time but is structured with collapsible sections.

### AD-4: Decision Records

**Decision:** Document significant design decisions as lightweight `DR-NNN.md` files in the agent's `decisions/` directory.

**Rationale:** Traditional ADRs are too heavy for agent development. Lightweight records (title, context, decision, consequences) capture the "why" behind each component without slowing development.

**Trade-off:** Minor overhead per decision, but invaluable for future maintainers and for AI assistants that need to understand historical context.

### AD-5: Generative Orchestration First

**Decision:** Enable `GenerativeActionsEnabled: true` and rely on the Copilot Studio orchestrator for routing. Custom topics are created only when deterministic logic is required.

**Rationale:** The orchestrator handles most user intents well when given clear agent instructions. Custom topics add maintenance burden and should only be used for flows that require Adaptive Cards, explicit branching, or multi-turn structured collection.

**Trade-off:** Less control over exact conversation flow, but significantly reduced topic count and maintenance.

### AD-6: Double Protection for Business Rules

**Decision:** Enforce critical business rules (dispute checks, ledger consistency, customer blocks) in both agent instructions AND topic logic.

**Rationale:** Instructions guide the orchestrator but are not guaranteed to be followed in edge cases. Topic-level conditions provide a deterministic safety net. Both layers must agree before external communications are sent.

**Trade-off:** Duplicated logic, but the consequences of a protection rule failure (sending collection emails to disputed accounts) justify the redundancy.

### AD-7: BC Extension Pack as Domain Layer

**Decision:** Package Business Central integration patterns as a separate "extension pack" within the framework, with its own skills, templates, and blueprints.

**Rationale:** Not all agents need BC integration. The pack provides domain-specific skills (MCP setup, instruction patterns, action templates, topic patterns, agent blueprints) that can be adopted incrementally without affecting the core framework.

**Trade-off:** Additional skills to maintain, but clear separation of concerns and reusability across different BC agent projects.

---

## Development Patterns

### Pattern 1: JIT Context Loading

Load user context and domain glossaries on the first message via a `conversation-init` or `OnActivity` topic, storing results in global variables. This avoids wasting tokens on every turn and keeps the agent's base instructions lean.

See: `skills/best-practices/` for implementation details.

### Pattern 2: Template-Based Generation

All YAML generation starts from canonical templates in `.github/templates/`. Templates contain `_REPLACE` placeholders for IDs and project-specific values. Skills read the template, generate fresh IDs, substitute values, and validate the result.

### Pattern 3: Evidencing

Every creation or modification operation concludes with a `CIRCE-EVIDENCE` block declaring which skills were loaded, templates used, patterns applied, and validation results. This provides an audit trail and helps AI assistants learn from their own operations.

### Pattern 4: HITL Gates

The Pre-Push Review skill enforces a mandatory human-in-the-loop gate before any push to Copilot Studio. It validates all modified files, collects associated decision records, shows a change summary, and requires explicit user approval. The Conductor agent also uses HITL gates between workflow steps.

### Pattern 5: Schema Validation Loop

After any YAML edit:
1. Run `node .github/scripts/schema-lookup.bundle.js validate <file>`
2. If errors are found, fix them before proceeding
3. Never push unvalidated YAML

This is enforced by conventions (rule #5), the pre-push review skill, and agent evidencing.

---

## File Conventions

| File | Location | Purpose |
|------|----------|---------|
| `circe-memory.md` | Agent root (next to `agent.mcs.yml`) | Cross-session project memory |
| `DR-NNN.md` | `decisions/` in agent directory | Decision records |
| `SKILL.md` | Each skill folder | Skill definition and instructions |
| `*.mcs.yml` | Agent directory | Copilot Studio YAML files |
| `*.bundle.js` | `.github/scripts/` | Bundled CLI tools (esbuild, no install) |

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on extending the framework with new skills, agents, or domain packs.
