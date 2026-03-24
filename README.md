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
3. **Circe (the agent)** — A financial collections bot for Business Central, serving as both a real-world agent and the framework's reference implementation

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
│  │  ┌─────────────────────────────────────────────────┐    │  │
│  │  │  Core: new-topic, add-node, add-action,         │    │  │
│  │  │  validate, manage-agent, chat-with-agent...     │    │  │
│  │  └─────────────────────────────────────────────────┘    │  │
│  │  ┌─────────────────────────────────────────────────┐    │  │
│  │  │  Governance: update-memory, add-decision-record,│    │  │
│  │  │  pre-push-review, coverage-report, scaffold     │    │  │
│  │  └─────────────────────────────────────────────────┘    │  │
│  │  ┌─────────────────────────────────────────────────┐    │  │
│  │  │  BC Pack: bc-mcp-setup, bc-instructions,        │    │  │
│  │  │  bc-action-templates, bc-topic-patterns,        │    │  │
│  │  │  bc-agent-blueprints                            │    │  │
│  │  └─────────────────────────────────────────────────┘    │  │
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

## Key Features

### From the Base Plugin (Power CAT)
- YAML-first authoring with schema validation
- 5 bundled CLI scripts (Node.js, no install needed)
- Push/pull/clone via VS Code Extension LSP binary
- Point-testing and batch test suites
- Template-based topic and action generation

### CIRCE Adds
- **Cross-session memory** — `circe-memory.md` tracks components, decisions, and project state across sessions
- **Decision records** — Lightweight ADRs documenting why each component was built the way it was
- **HITL gates** — Mandatory pre-push review with validation, diff summary, and explicit approval
- **Skills evidencing** — Every operation declares which skills loaded, patterns applied, and validation results
- **Always-on conventions** — Core rules injected via hooks, independent of agent preloads
- **Agent scaffolding** — Generate a functional bot from scratch (greeting + fallback + error handler)
- **Conductor agent** — Orchestrates multi-step workflows (create → validate → push → test) with gates between steps
- **Coverage metrics** — Topic test coverage, trigger density, knowledge validation, model description completeness
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

## Skills

### Core Skills (from base plugin)

| Skill | Purpose |
|-------|---------|
| `new-topic` | Create topic from template |
| `add-node` | Add nodes to existing topics |
| `add-action` | Guide through connector action setup |
| `edit-action` | Modify action inputs, outputs, descriptions |
| `add-adaptive-card` | Generate and insert Adaptive Cards |
| `add-generative-answers` | SearchAndSummarizeContent patterns |
| `add-knowledge` | Add knowledge sources (web, SharePoint, Graph) |
| `add-global-variable` | Create conversation-scoped variables |
| `add-other-agents` | Child agents and connected agents |
| `edit-agent` | Modify agent settings and instructions |
| `edit-triggers` | Modify trigger phrases and model descriptions |
| `validate` | YAML validation against schema |
| `lookup-schema` | Query schema definitions |
| `list-kinds` | List valid kind values |
| `list-topics` | Inventory all topics |
| `manage-agent` | Push, pull, clone, diff |
| `clone-agent` | Guided agent cloning |
| `chat-with-agent` | Point-test utterances |
| `directline-chat` | DirectLine v3 testing |
| `run-tests` | Batch test suites and evaluation analysis |
| `known-issues` | Search GitHub issues database |
| `best-practices` | JIT glossary, user context, orchestrator patterns |
| `int-project-context` | Shared project conventions |
| `int-reference` | YAML reference tables |

### CIRCE Governance Skills

| Skill | Purpose |
|-------|---------|
| `update-memory` | Update circe-memory.md after operations |
| `add-decision-record` | Create lightweight ADRs |
| `pre-push-review` | Mandatory review gate before push |
| `coverage-report` | Quality and coverage metrics |
| `scaffold-agent` | Generate complete agent from scratch |

### BC Extension Pack Skills

| Skill | Purpose |
|-------|---------|
| `bc-mcp-setup` | Configure BC MCP Server and Copilot Studio connection |
| `bc-instructions-patterns` | Composable instruction blocks for BC agents |
| `bc-action-templates` | Pre-built TaskDialog templates for BC operations |
| `bc-topic-patterns` | Topic patterns for collections, orders, incidents |
| `bc-agent-blueprints` | Complete agent specifications (collections, sales, support) |

---

## BC Extension Pack

The BC Extension Pack provides domain-specific skills for building Copilot Studio agents that integrate with Dynamics 365 Business Central via the MCP Server.

### Agent Blueprints

Three ready-to-implement blueprints define the full configuration for common BC scenarios:

**Collections Agent** — Manages overdue payments with dispute-aware protection rules, aging-based escalation, and payment registration. Based on the Circe reference implementation.

**Sales Assistant** — Customer lookup, item availability, quote and order management with delivery date verification.

**Customer Support** — Incident creation with Adaptive Cards, order tracking, and human escalation triggers.

Each blueprint specifies: MCP tool configuration, instruction blocks, topics, actions, variables, protection rules, and estimated Copilot Credits consumption.

### MCP Integration

The `bc-mcp-setup` skill guides through the complete setup:

1. Enable MCP Server in BC Feature Management (BC27+)
2. Configure API page exposure with appropriate operations
3. Connect in Copilot Studio (environment, company, auth mode)
4. Validate with test queries
5. Pull to local workspace

Supports both **Dynamic Tool Mode** (agent discovers tools at runtime) and **Explicit Tool Mode** (agent maker selects tools at design time).

---

## Reference Implementation: Circe — Collections Agent

The `Circe/` directory contains a complete, working Copilot Studio bot that serves as both a production agent and the framework's reference implementation.

### What Circe Does

Circe manages collections and overdue payments for a Business Central environment. It integrates with BC (customer data, ledger entries, payment journals) and Outlook (email reminders, meeting scheduling) via MCP.

### Architecture

| Component | Count | Purpose |
|-----------|-------|---------|
| Topics | 5+ | Greeting, fallback, error handler, collections flow, conversation init |
| Actions | 6+ | Customer balance, overdue invoices, payment registration, email, meetings |
| Knowledge | As needed | Company policies, collection procedures |
| Variables | 2+ | UserCountry (JIT), Glossary (JIT) |
| Connections | 2 | Business Central MCP, Outlook MCP |

### Protection Rules

Circe enforces non-negotiable business rules in both instructions AND topic logic (double protection):

- **No external communications** when an active dispute or material ledger inconsistency (>5% or >1.000€) exists
- **Internal reconciliation report** generated instead when protection triggers
- **Human confirmation required** before creating payments or posting journals
- **Full traceability** — every action logged with timestamp, customer, and result

### BC Configuration

| Setting | Value |
|---------|-------|
| Environment | SANDBOX_US |
| Company | CRONUS USA, Inc. |
| MCP Config | CIRCE |
| Schema | `copilots_header_cra1e_Circe` |
| Language | Spanish |
| Generative Actions | Enabled |

### How to Use as Reference

Circe demonstrates every CIRCE framework pattern in practice:

- **Skill-first development** — Every component was created via skills, never manual YAML
- **Generative orchestration** — `GenerativeActionsEnabled: true` with custom topics only where deterministic logic is required
- **Protection rules** — Business rules enforced at both instruction and topic level
- **JIT user context** — User profile loaded on first message via OnActivity
- **MCP integration** — BC and Outlook connections via connection references
- **Adaptive Cards** — Debt summary and payment confirmation cards

---

## Project Structure

```
Circe/
├── .github/
│   ├── agents/                    # 5 specialized agents
│   │   ├── copilot-studio-author.md
│   │   ├── copilot-studio-manage.md
│   │   ├── copilot-studio-test.md
│   │   ├── copilot-studio-troubleshoot.md
│   │   └── copilot-studio-conductor.md
│   ├── skills/                    # 33+ modular skills
│   │   ├── [core skills]         # From base plugin
│   │   ├── [governance skills]   # CIRCE additions
│   │   └── [bc-* skills]         # BC Extension Pack
│   ├── scripts/                   # 5 bundled CLI scripts
│   │   ├── schema-lookup.bundle.js
│   │   ├── manage-agent.bundle.js
│   │   ├── chat-with-agent.bundle.js
│   │   ├── directline-chat.bundle.js
│   │   └── connector-lookup.bundle.js
│   ├── templates/                 # Reusable YAML templates
│   │   ├── topics/
│   │   ├── actions/
│   │   ├── agents/
│   │   ├── knowledge/
│   │   └── variables/
│   ├── tests/                     # Test infrastructure
│   ├── reference/                 # Schema reference
│   ├── hooks/
│   │   └── hooks.json            # SessionStart auto-delegation
│   ├── circe-conventions.md      # Always-on rules (≤500 words)
│   ├── copilot-instructions.md   # Master coordination
│   ├── README.md                 # Framework technical docs
│   └── CHANGELOG.md
│
├── [agent-name]/                  # Circe agent (cloned from Copilot Studio)
│   ├── agent.mcs.yml             # Agent metadata
│   ├── settings.mcs.yml          # Settings, instructions, generative config
│   ├── connectionreferences.mcs.yml  # BC + Outlook MCP connections
│   ├── topics/                   # Conversation topics
│   ├── actions/                  # MCP action definitions
│   ├── knowledge/                # Knowledge sources
│   ├── variables/                # Global variables
│   ├── agents/                   # Child agents (if any)
│   ├── decisions/                # Decision records (DR-001, DR-002...)
│   └── circe-memory.md           # Cross-session project memory
│
├── docs/
│   ├── bc-extension-pack.md      # BC Pack documentation
│   └── setup-guide.md            # End-to-end setup guide
│
├── README.md                     # This file
├── LICENSE
└── CONTRIBUTING.md
```

---

## Getting Started

### Prerequisites

- **Node.js** 18+ (for CLI scripts)
- **VS Code** with [Copilot Studio Extension](https://marketplace.visualstudio.com/items?itemName=ms-CopilotStudio.vscode-copilotstudio)
- **GitHub Copilot** (VS Code or CLI) or **Claude Code**
- **Copilot Studio license** with Copilot Credits

For Business Central integration:
- **BC 27+** (2025 Wave 2 or later) with MCP Server feature enabled
- **MCP-ADMIN** permission set in BC

### Quick Start

**1. Clone this repository**

```bash
git clone https://github.com/javiarmesto/circe.git
cd circe
```

**2. Install the base plugin**

For GitHub Copilot CLI:
```bash
/plugin marketplace add microsoft/skills-for-copilot-studio
```

For Claude Code:
```bash
claude plugin install /path/to/skills-for-copilot-studio --scope project
```

**3. Clone your agent from Copilot Studio**

```
@copilot-studio-manage clone
```

Or use the Copilot Studio VS Code Extension directly.

**4. Start building**

```
@copilot-studio-author Create a topic that checks customer balance
```

The framework handles the rest: skill loading, template selection, schema validation, and memory updates.

### For Business Central Agents

```
@copilot-studio-conductor

Set up a collections agent connected to Business Central.
Use the Collections Agent blueprint from the BC Extension Pack.
```

The conductor orchestrates the full workflow: scaffold → MCP setup → instructions → actions → topics → review → push.

---

## Development Workflow

```
Describe what you need
        │
        ▼
┌───────────────────┐
│  Conductor Agent   │  (or direct to specialist agent)
└───────┬───────────┘
        │
        ▼
┌───────────────────┐
│   Author Agent     │  Creates/edits YAML via skills
│   + Skills         │  Validates with schema-lookup
│   + Templates      │  Generates decision record
│   + Evidencing     │  Updates memory
└───────┬───────────┘
        │
        ▼
┌───────────────────┐
│  Pre-Push Review   │  Validates all files
│   (HITL Gate)      │  Shows diff + DRs
│                    │  Requires explicit approval
└───────┬───────────┘
        │
        ▼
┌───────────────────┐
│   Manage Agent     │  Push to Copilot Studio (draft)
└───────┬───────────┘
        │
        ▼
   User publishes
   in Copilot Studio UI
        │
        ▼
┌───────────────────┐
│    Test Agent      │  Point-test, batch suite, or
│                    │  evaluation analysis
└───────────────────┘
```

---

## Acknowledgements

**CIRCE is built on top of [Skills for Copilot Studio](https://github.com/microsoft/skills-for-copilot-studio)**, the open-source plugin created by **Giorgio Ughini** and the **Microsoft Power CAT team**. They built the foundation that makes YAML-first agent development possible — the schema validation engine, the CLI scripts, the skill architecture, and the testing infrastructure. CIRCE extends their work; it does not replace it.

Additional references:
- [Microsoft Copilot Studio Documentation](https://learn.microsoft.com/en-us/microsoft-copilot-studio/)
- [Business Central MCP Server](https://learn.microsoft.com/en-us/dynamics365/business-central/dev-itpro/ai/configure-mcp-server)
- [BC Agent SDK](https://learn.microsoft.com/en-us/dynamics365/business-central/dev-itpro/ai/ai-development-toolkit-overview)
- [ALDC — AL Development Collection](https://github.com/javiarmesto/AL-Development-Collection-for-GitHub-Copilot) (sister framework for BC AL development)
- [Power Fx Reference](https://learn.microsoft.com/en-us/power-platform/power-fx/reference/function-text)

---

## Related Projects

| Project | Description |
|---------|-------------|
| [ALDC](https://github.com/javiarmesto/AL-Development-Collection-for-GitHub-Copilot) | Skills-based development framework for Business Central AL with GitHub Copilot |
| [Skills for Copilot Studio](https://github.com/microsoft/skills-for-copilot-studio) | Base plugin by Giorgio Ughini & Power CAT team |
| [TechSphere Dynamics](https://techspheredynamics.com) | Articles and content on BC, AI, and agentic development |

---

## Author

**Javier Armesto González**
Microsoft MVP (Business Central & Azure AI Services)
Head of R&D & AI at VS Sistemas
[LinkedIn](https://www.linkedin.com/in/jarmesto/) · [TechSphere Dynamics](https://techspheredynamics.com) · [Substack](https://techspheredynamics.substack.com)

---

## License

MIT — See [LICENSE](LICENSE) for details.
