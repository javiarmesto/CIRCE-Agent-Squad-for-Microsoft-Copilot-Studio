# CIRCE — Agent Architecture Framework for Microsoft Copilot Studio

> **Specialized agents, modular skills, and domain packs for building production-grade Copilot Studio bots — from code, not clicks.**
>
> Built on top of [Skills for Copilot Studio](https://github.com/microsoft/skills-for-copilot-studio) by **Giorgio Ughini** and the **Microsoft Power CAT team**.

[![Framework](https://img.shields.io/badge/framework-CIRCE-indigo.svg)](#)
[![Base Plugin](https://img.shields.io/badge/base-Skills%20for%20Copilot%20Studio-blue.svg)](https://github.com/microsoft/skills-for-copilot-studio)
[![BC Extension Pack](https://img.shields.io/badge/pack-Business%20Central-green.svg)](#bc-extension-pack)
[![License](https://img.shields.io/badge/license-MIT-green)](./LICENSE)

---

## What is CIRCE?

CIRCE is a framework that extends [Skills for Copilot Studio](https://github.com/microsoft/skills-for-copilot-studio) with enterprise-grade development practices: cross-session memory, decision records, human-in-the-loop gates, skills evidencing, and domain-specific extension packs.

The base plugin by the Power CAT team provides the foundation — YAML authoring, schema validation, CLI scripts, and testing tools. CIRCE adds the layers a professional team needs to build, maintain, and govern Copilot Studio agents at scale.

This repository contains:

1. **The CIRCE Framework** — 5 agents, 33+ skills, conventions, memory, and governance systems
2. **The BC Extension Pack** — Domain skills for Business Central integration via MCP
3. **Medea** — A reference implementation (collections agent) with its own [README](./Medea/README.md)

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     CIRCE Framework                         │
│                                                             │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌───────────────┐  │
│  │  Author   │ │  Manage  │ │   Test   │ │ Troubleshoot  │  │
│  │  Agent    │ │  Agent   │ │  Agent   │ │    Agent      │  │
│  └────┬─────┘ └────┬─────┘ └────┬─────┘ └──────┬────────┘  │
│       │             │            │               │           │
│  ┌────┴─────────────┴────────────┴───────────────┴────────┐  │
│  │                  Conductor Agent                        │  │
│  │         (orchestrates multi-agent workflows)            │  │
│  └─────────────────────┬───────────────────────────────────┘  │
│                        │                                     │
│  ┌─────────────────────┴───────────────────────────────────┐  │
│  │              33+ Modular Skills                         │  │
│  │  Core · Governance · BC Extension Pack                  │  │
│  └─────────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │  Infrastructure: 5 CLI Scripts (esbuild bundled)        │  │
│  │  schema-lookup · manage-agent · chat-with-agent         │  │
│  │  directline-chat · connector-lookup                     │  │
│  └─────────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │  Always-On: circe-conventions.md · hooks.json           │  │
│  │  circe-memory.md · decision records                     │  │
│  └─────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  Skills for Copilot Studio (base plugin)                    │
│  by Giorgio Ughini & Microsoft Power CAT team               │
│  github.com/microsoft/skills-for-copilot-studio             │
└─────────────────────────────────────────────────────────────┘
```

---

## Quick Start: Order Tracker (your first CIRCE agent)

Build a working Business Central agent in three phases. Total time: ~30 min.

### Prerequisites

| What | Why |
|------|-----|
| **VS Code** + [Copilot Studio Extension](https://marketplace.visualstudio.com/items?itemName=ms-CopilotStudio.vscode-copilotstudio) | Clone/push/pull agents |
| **GitHub Copilot** (VS Code) or **Claude Code** | AI agent engine |
| **Node.js 18+** | Bundled CLI scripts |
| **Copilot Studio license** | With Copilot Credits |
| **BC 27+** with MCP Server enabled + `MCP-ADMIN` permission set | For BC integration |

### Step 0 — Install CIRCE

```bash
git clone https://github.com/javiarmesto/circe.git ~/circe
cd ~/circe
```

### Step 1 — Create a blank agent in Copilot Studio

Open [copilotstudio.microsoft.com](https://copilotstudio.microsoft.com), create a new agent with the name **"Order Tracker"** — nothing else. Just the name. This gives the agent an identity (schema name, agent ID, environment URL) that CIRCE needs.

### Step 2 — Clone, configure & build (Phase 1 + 2)

Open the CIRCE workspace in VS Code and use the AI chat:

```
@copilot-studio-manage Clone the "Order Tracker" agent from Copilot Studio.
```

Then load the requirements and build:

```
@copilot-studio-author

Read the file order-tracker-requirements.md as context for this agent.

Then:
1. Set up the MCP connection to Business Central
   (Environment: YOUR_BC_ENVIRONMENT, Company: CRONUS USA Inc., Explicit Tool Mode, read-only)
2. Create action "Get Sales Order" (query by number or customer)
3. Create topic "Check Order Status" with Adaptive Card
   (triggers: order status, track order, where is my order)
4. Ensure greeting, fallback and error handler topics exist
```

### Step 3 — Review, deploy & test (Phase 3)

```
@copilot-studio-conductor

Run pre-push review, push to Copilot Studio, then test with:
- "What's the status of order S-ORD101001?"
- "Check orders for Adatum Corporation"
- "Where is my order?"
```

After push, publish the draft in Copilot Studio UI, then the conductor runs the tests.

> See [`docs/quickstart-order-tracker.md`](docs/quickstart-order-tracker.md) for the full walkthrough with expected results and validation checklists.

---

## Key Features

### From the Base Plugin (Power CAT)
- YAML-first authoring with schema validation
- 5 bundled CLI scripts (Node.js, no install needed)
- Push/pull/clone via VS Code Extension LSP binary
- Point-testing and batch test suites
- Template-based topic and action generation

### CIRCE Adds
- **Cross-session memory** — `circe-memory.md` tracks components, decisions, and project state
- **Decision records** — Lightweight ADRs documenting why each component was built the way it was
- **HITL gates** — Mandatory pre-push review with validation, diff summary, and explicit approval
- **Skills evidencing** — Every operation declares which skills loaded, patterns applied, and validation results
- **Always-on conventions** — Core rules injected via hooks, independent of agent preloads
- **Agent scaffolding** — Generate a functional bot from scratch
- **Conductor agent** — Orchestrates multi-step workflows with gates between steps
- **Coverage metrics** — Topic test coverage, trigger density, knowledge validation
- **BC Extension Pack** — Domain skills for Business Central MCP integration

---

## Agents

| Agent | Role | Tools |
|-------|------|-------|
| **Author** | Creates and edits YAML — topics, actions, knowledge, variables, child agents | read, edit, search, execute |
| **Manage** | ALM operations — push, pull, clone, diff, list environments and agents | read, search, execute |
| **Test** | Tests published agents — point-test, batch suites, DirectLine, evaluation analysis | read, edit, search, execute |
| **Troubleshoot** | Debugs issues — validates YAML, searches known issues, proposes fixes | read, search, execute |
| **Conductor** | Orchestrates multi-agent workflows with HITL gates | read, search, execute |

---

## Development Workflow

```
1. Create blank agent in Copilot Studio (just the name)
        │
        ▼
2. Clone to local workspace
        │
        ▼
3. Build with Author Agent (topics, actions, knowledge, instructions)
        │
        ▼
4. Pre-push Review (HITL gate: validate → diff → approve)
        │
        ▼
5. Push to Copilot Studio (draft)
        │
        ▼
6. Publish in Copilot Studio UI
        │
        ▼
7. Test with Test Agent
```

---

## BC Extension Pack

Domain skills for building agents connected to Dynamics 365 Business Central via MCP:

| Skill | Purpose |
|-------|---------|
| `bc-mcp-setup` | Configure BC MCP Server and connect to Copilot Studio |
| `bc-instructions-patterns` | Composable instruction blocks for BC agents |
| `bc-action-templates` | Pre-built TaskDialog YAML for common BC operations |
| `bc-topic-patterns` | Topic patterns with orchestrator vs custom topic decision guidance |
| `bc-agent-blueprints` | Complete agent specifications (collections, sales, support) |

Three ready-to-implement blueprints: **Collections Agent**, **Sales Assistant**, **Customer Support**. See [BC Extension Pack docs](docs/bc-extension-pack.md).

---

## Project Structure

```
circe/
├── .github/
│   ├── agents/                    # 5 specialized agents
│   ├── skills/                    # 33+ modular skills
│   ├── scripts/                   # 5 bundled CLI scripts
│   ├── templates/                 # Reusable YAML templates
│   ├── tests/                     # Test infrastructure
│   ├── reference/                 # Schema reference
│   ├── hooks/hooks.json           # SessionStart auto-delegation
│   ├── circe-conventions.md       # Always-on rules
│   └── copilot-instructions.md    # Master coordination
│
├── Medea/                         # Reference implementation (collections agent)
│
├── docs/
│   ├── quickstart-order-tracker.md
│   ├── bc-extension-pack.md
│   └── setup-guide.md
│
├── README.md                      # This file
├── LICENSE
└── CONTRIBUTING.md
```

---

## Acknowledgements

**CIRCE is built on top of [Skills for Copilot Studio](https://github.com/microsoft/skills-for-copilot-studio)**, the open-source plugin created by **Giorgio Ughini** and the **Microsoft Power CAT team**. They built the foundation that makes YAML-first agent development possible. CIRCE extends their work; it does not replace it.

---

## Author

**Javier Armesto González**
Microsoft MVP (Business Central & Azure AI Services)
Head of R&D & AI at VS Sistemas
[LinkedIn](https://www.linkedin.com/in/jarmesto/) · [TechSphere Dynamics](https://techspheredynamics.com) · [Substack](https://techspheredynamics.substack.com)

---

## License

MIT — See [LICENSE](LICENSE) for details.
