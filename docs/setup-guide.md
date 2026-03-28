# CIRCE — Setup Guide

> End-to-end guide for setting up the CIRCE framework and building your first Copilot Studio agent.

---

## Prerequisites

| Requirement | Details |
|-------------|---------|
| **Node.js** | 18+ (for CLI scripts) |
| **VS Code** | With [Copilot Studio Extension](https://marketplace.visualstudio.com/items?itemName=ms-CopilotStudio.vscode-copilotstudio) |
| **AI Assistant** | GitHub Copilot (VS Code or CLI) or Claude Code |
| **Copilot Studio** | License with Copilot Credits |

For Business Central integration, also:
| **Business Central** | 27+ (2025 Wave 2) with MCP Server enabled |
| **BC Permission** | `MCP-ADMIN` permission set |

---

## Step 1: Clone the Repository

```bash
git clone https://github.com/javiarmesto/CIRCE---Agent-Architecture-Framework-for-Microsoft-Copilot-Studio.git
cd CIRCE---Agent-Architecture-Framework-for-Microsoft-Copilot-Studio
```

---

## Step 2: Verify the Framework

Confirm scripts are accessible:

```bash
node .github/scripts/schema-lookup.bundle.js --help
```

No `npm install` is needed — all scripts are pre-bundled with esbuild.

---

## Step 3: Clone Your Agent (Option A — Existing Agent)

If you already have a Copilot Studio agent:

```
@copilot-studio-manage clone
```

This downloads the agent YAML files to a local directory. The framework auto-discovers the agent via `**/agent.mcs.yml`.

---

## Step 3: Scaffold a New Agent (Option B — From Scratch)

If starting fresh:

```
@copilot-studio-author /copilot-studio:scaffold-agent
```

This generates:
- `agent.mcs.yml` — Agent metadata
- `settings.mcs.yml` — Settings and instructions
- `topics/greeting.topic.mcs.yml` — Welcome topic
- `topics/fallback.topic.mcs.yml` — Fallback handler
- `topics/error-handler.topic.mcs.yml` — Error handler

---

## Step 4: Build Your Agent

Use the specialist agents for different tasks:

### Create a Topic
```
@copilot-studio-author Create a topic that greets the user and asks how to help
```

### Add an Action
```
@copilot-studio-author /copilot-studio:add-action
```

### Edit Agent Instructions
```
@copilot-studio-author /copilot-studio:edit-agent
```

### Add Knowledge
```
@copilot-studio-author /copilot-studio:add-knowledge
```

---

## Step 5: Validate

After every YAML edit:

```bash
node .github/scripts/schema-lookup.bundle.js validate <file.mcs.yml>
```

Or use the validate skill:
```
@copilot-studio-troubleshoot /copilot-studio:validate
```

---

## Step 6: Push to Copilot Studio

The pre-push review gate runs automatically:

```
@copilot-studio-manage push
```

This validates all files, shows a change summary with associated decision records, and requests your explicit approval before pushing.

After pushing, **publish** the agent in the Copilot Studio UI to make changes live.

---

## Step 7: Test

### Point Test
```
@copilot-studio-test Send "Hola, necesito ayuda" to the agent
```

### Batch Test Suite
```
@copilot-studio-test /copilot-studio:run-tests
```

---

## For Business Central Agents

Use the BC Extension Pack for a streamlined setup:

```
@copilot-studio-conductor

Set up a collections agent connected to Business Central.
Use the Collections Agent blueprint from the BC Extension Pack.
```

The Conductor orchestrates the full workflow:
1. Scaffold agent
2. Configure MCP connection
3. Apply instruction patterns
4. Add action templates
5. Create topics
6. Pre-push review
7. Push to cloud
8. Test

See [BC Extension Pack](bc-extension-pack.md) for detailed documentation.

---

## Troubleshooting

### YAML validation fails
```
@copilot-studio-troubleshoot Validate and fix <file.mcs.yml>
```

### Agent not responding correctly
```
@copilot-studio-test Send "<test utterance>" to the agent
```

### Known issues
```
@copilot-studio-troubleshoot /copilot-studio:known-issues <symptom>
```

---

## Further Reading

- [CIRCE Framework README](../README.md)
- [Architecture Decisions](../CLAUDE.md)
- [BC Extension Pack](bc-extension-pack.md)
- [Copilot Studio Documentation](https://learn.microsoft.com/en-us/microsoft-copilot-studio/)
