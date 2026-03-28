# Quick Start: IT Support KB — Your First CIRCE Agent (No BC Required)

> Build a knowledge-grounded IT support agent that answers employee questions from SharePoint and web sources — all from code, no Business Central needed.

---

## What You'll Build

An agent called **IT Support KB** that:
- Answers IT support questions grounded in SharePoint documents and a public FAQ page
- Loads a glossary of internal acronyms at conversation start (JIT)
- Personalizes answers using the user's M365 profile (department, country)
- Escalates to a human agent when needed
- All user-facing content in Spanish

**Time:** ~20 minutes (prerequisites already installed)

---

## Prerequisites

### Software

| Tool | Version | Verify |
|------|---------|--------|
| VS Code | 1.85+ | `code --version` |
| Node.js | 18+ | `node --version` |
| Git | 2.30+ | `git --version` |

### VS Code Extensions

```bash
code --install-extension ms-CopilotStudio.vscode-copilotstudio
code --install-extension GitHub.copilot
code --install-extension GitHub.copilot-chat
```

### Licenses & Access

| Requirement | Where |
|-------------|-------|
| GitHub Copilot plan (Pro/Business/Enterprise) | github.com/features/copilot |
| Copilot Studio license with Credits | admin.powerplatform.microsoft.com |
| Power Platform environment | admin.powerplatform.microsoft.com |
| SharePoint site with IT documentation | Your tenant's SharePoint |

> **No Business Central required.** This quickstart validates the CIRCE framework using only Copilot Studio + SharePoint + a public website.

---

## Pre-condition: Create a Blank Agent

Before CIRCE can work, the agent needs an identity in Copilot Studio:

1. Open [copilotstudio.microsoft.com](https://copilotstudio.microsoft.com)
2. Click **Create** → **New agent**
3. Name it **"IT Support KB"**
4. Don't configure anything else — no topics, no knowledge, no instructions
5. Save

This creates the schema name, agent ID, and environment URL that CIRCE needs to clone and push back to. Everything else happens from code.

---

## Phase 1 — Setup: Clone + Context + Knowledge Sources

**Agents:** Manage → Author
**Skills:** clone-agent, add-knowledge, best-practices (JIT glossary + user context), edit-agent

### Prompt

```
@copilot-studio-manage Clone the "IT Support KB" agent from Copilot Studio.

@copilot-studio-author

Read the file docs/kb-agent-requirements.md as context for this agent.

Then:
1. Add three knowledge sources:
   - SharePoint: https://YOUR_TENANT.sharepoint.com/sites/ITSupport/Shared%20Documents/Policies
   - SharePoint: https://YOUR_TENANT.sharepoint.com/sites/ITSupport/Shared%20Documents/Guides
   - Public website: https://YOUR_COMPANY.com/it-support/faq

2. Set agent instructions per the requirements doc:
   - Language: Spanish
   - Ground answers in knowledge sources only
   - Cite sources, never fabricate
   - Use glossary for acronym expansion
   - Inject today's date with Power Fx

3. Add conversation starters:
   - "¿Cómo conecto a la VPN?"
   - "¿Cuál es la política de contraseñas?"
   - "Tengo un problema con mi equipo"
   - "¿Cómo solicito acceso a una aplicación?"
```

> **Replace the placeholder URLs** with your actual SharePoint site and FAQ page before running.

### What happens

1. **Manage** clones the blank agent → creates local directory with `agent.mcs.yml`, `settings.mcs.yml`
2. **Author** reads the requirements document → understands what to build
3. **Author** creates 3 knowledge source YAML files in `knowledge/`
4. **Author** writes agent instructions and conversation starters in `agent.mcs.yml`
5. `circe-memory.md` initialized with configuration and knowledge sources

### Expected result

- Local directory with cloned agent structure
- 3 knowledge source files in `knowledge/`:
  - `it-policies.knowledge.mcs.yml` (SharePoint)
  - `how-to-guides.knowledge.mcs.yml` (SharePoint)
  - `it-faqs.knowledge.mcs.yml` (Public website)
- `agent.mcs.yml` with Spanish instructions and conversation starters
- `circe-memory.md` initialized
- DR-001 (clone + context), DR-002 (knowledge source selection)

### Validation checklist

- [ ] Local directory created with `agent.mcs.yml`
- [ ] `settings.mcs.yml` has correct schema name from Copilot Studio
- [ ] 3 knowledge source files in `knowledge/` (validated, 0 errors)
- [ ] Agent instructions written in Spanish with date injection
- [ ] Conversation starters configured
- [ ] `circe-memory.md` initialized with Agent Configuration populated
- [ ] DR-001 and DR-002 generated

---

## Phase 2 — Build: JIT Init + Generative Answers + Topics

**Agent:** Author
**Skills:** best-practices (JIT glossary + user context), add-generative-answers, new-topic

### Prompt

```
@copilot-studio-author

Using the IT Support KB requirements:

1. Create a conversation-init topic (OnActivity) that:
   - Loads a glossary CSV into Global.Glossary
     (acronyms: VPN, MFA, SSO, ITSM, SLA, AD, BYOD — include full expansions)
   - Loads the user's M365 profile into global variables
     (department, country, display name)

2. Create topic "IT Knowledge Search" with:
   - Triggers: "cómo", "qué es", "dónde encuentro", "ayuda con",
     "problema con", "necesito saber", "política de"
   - Flow: SearchAndSummarizeContent grounded in all knowledge sources
   - Handle not-found: friendly message + suggest escalation

3. Ensure greeting, fallback (with knowledge search), error handler,
   and escalation topics exist with appropriate Spanish messages.
```

### What happens

1. **Author** creates the conversation-init topic with glossary + user context (combined OnActivity pattern)
2. **Author** creates IT Knowledge Search topic with generative answers
3. **Author** verifies/creates system topics (greeting, fallback with knowledge fallback, error handler, escalation)
4. Each component gets CIRCE-EVIDENCE blocks and decision records

### Expected result

- `topics/conversation-init.topic.mcs.yml` — JIT glossary + user context loader
- `variables/` — Global variables for Glossary, UserDepartment, UserCountry, UserDisplayName
- `topics/ITKnowledgeSearch.topic.mcs.yml` — Main search topic with generative answers
- System topics verified/created (greeting, fallback, error handler, escalation)
- DR-003 (conversation-init: JIT glossary + user context combined)
- DR-004 (knowledge search topic + generative answers pattern)

### Validation checklist

- [ ] conversation-init topic uses OnActivity trigger with `ActivityType = "message"` and `TurnCount = 1`
- [ ] Glossary CSV loaded into `Global.Glossary` variable
- [ ] User profile loaded via M365 Users connector
- [ ] IT Knowledge Search topic has 7+ trigger phrases in Spanish
- [ ] `SearchAndSummarizeContent` node preceded by `CreateSearchQuery`
- [ ] Fallback topic includes knowledge search (acts as catch-all)
- [ ] Greeting topic displays conversation starters
- [ ] Escalation topic handles "hablar con una persona"
- [ ] All topic files validated (0 schema errors)
- [ ] DR-003 and DR-004 generated
- [ ] `circe-memory.md` updated with all new components

---

## Phase 3 — Review, Deploy & Test

**Agents:** Conductor → Manage → Test
**Skills:** pre-push-review, manage-agent, chat-with-agent

### Prompt

```
@copilot-studio-conductor

Run the full review-deploy-test cycle for IT Support KB:

1. Pre-push review: validate all YAML, show diff summary and decision records,
   run coverage report. Wait for my explicit approval before continuing.

2. After approval: push to Copilot Studio as draft.

3. After I confirm publication: test with these utterances:
   - "¿Cómo conecto a la VPN desde casa?"
   - "¿Cuál es la política de contraseñas?"
   - "¿Qué significa MFA?"
   - "Necesito acceso a SAP"
   - "Quiero hablar con una persona"
```

### What happens

1. **Conductor** runs pre-push review → validates all YAML, presents diff, lists DRs
2. You review and approve → **Manage** pushes to Copilot Studio (draft)
3. You publish manually in Copilot Studio UI
4. You confirm publication → **Test** runs 5 utterances against the published agent

### Expected result

- Pre-push review: 0 validation errors, coherent diff, DR-001 through DR-004 listed
- HITL gate: waits for explicit approval before push
- Push successful, agent visible as draft in Copilot Studio
- Pause for manual publication
- 5/5 utterances resolved correctly:

| # | Utterance | Expected behavior |
|---|-----------|-------------------|
| 1 | "¿Cómo conecto a la VPN desde casa?" | Grounded answer from knowledge sources with citation |
| 2 | "¿Cuál es la política de contraseñas?" | Policy answer from SharePoint Policies folder |
| 3 | "¿Qué significa MFA?" | Glossary-expanded search → finds Multi-Factor Authentication docs |
| 4 | "Necesito acceso a SAP" | Searches guides → if not found, suggests contacting IT |
| 5 | "Quiero hablar con una persona" | Escalation topic triggered |

### Validation checklist

- [ ] Pre-push review passed (0 validation errors)
- [ ] Decision records DR-001 through DR-004 listed in review summary
- [ ] Coverage report generated (advisory, no blocking)
- [ ] Push completed successfully
- [ ] Agent published in Copilot Studio UI
- [ ] Test 1: Answer with citation from knowledge source
- [ ] Test 2: Policy document referenced correctly
- [ ] Test 3: Acronym expanded, relevant content found
- [ ] Test 4: Graceful not-found with escalation suggestion
- [ ] Test 5: Escalation flow triggered correctly
- [ ] Responses in Spanish throughout
- [ ] Response times under 5 seconds

---

## CIRCE Capabilities Validated

| Capability | Phase |
|------------|-------|
| Clone agent | 1 |
| Context loading (requirements doc) | 1 |
| Knowledge sources (SharePoint + Public Website) | 1 |
| Agent instructions editing | 1 |
| Conversation starters | 1 |
| JIT Glossary (best practice) | 2 |
| JIT User Context (best practice) | 2 |
| Combined conversation-init (OnActivity) | 2 |
| Generative answers (SearchAndSummarizeContent) | 2 |
| Global variables | 2 |
| Skills evidencing | 1, 2, 3 |
| Cross-session memory | 1, 2, 3 |
| Decision records | 1, 2, 3 |
| Pre-push review (HITL) | 3 |
| Coverage report | 3 |
| Push to Copilot Studio | 3 |
| Point-test | 3 |
| Conductor orchestration | 3 |

**Not covered** (for subsequent iterations): BC Extension Pack, MCP actions, Adaptive Cards, batch test suites, child agents, custom connectors, topic patterns.

---

## Comparison with Order Tracker Quick Start

| Dimension | IT Support KB (this) | Order Tracker |
|-----------|---------------------|---------------|
| **BC required** | No | Yes (27+ with MCP Preview) |
| **Knowledge sources** | SharePoint + Public Website | None (data from BC) |
| **Generative answers** | Yes | No |
| **Best practices** | JIT Glossary + User Context | None |
| **Adaptive Cards** | No | Yes |
| **MCP actions** | None | 2 BC actions |
| **Prerequisites** | 3 products | 5+ products |
| **Estimated time** | ~20 min | ~30 min |
| **BC Extension Pack** | Not used | Core feature |

Use **this quickstart** to validate the core CIRCE framework without BC dependencies. Use **Order Tracker** to additionally validate the BC Extension Pack.

---

## Next Steps

If IT Support KB works, you've validated the core CIRCE flow without BC. Three natural paths forward:

**Option A — Add ITSM integration:** Connect to ServiceNow or Jira via a connector action to create tickets from the agent → validates `add-action` + topic flows with external connectors.

**Option B — Try Order Tracker:** Follow the [Order Tracker quickstart](quickstart-order-tracker.md) to validate the BC Extension Pack → validates MCP, action templates, Adaptive Cards.

**Option C — Add Adaptive Cards:** Enhance the KB agent with card-based feedback collection and formatted search results → validates `add-adaptive-card` skill.

---

## Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| "No agent.mcs.yml found" | Agent not cloned | Run Phase 1 clone step |
| Clone fails with "access denied" | No Power Platform access | Verify permissions in admin.powerplatform.microsoft.com |
| Knowledge returns no results | Wrong SharePoint URL format | Use direct folder path, not sharing links. Encode spaces as `%20` |
| Glossary not expanding acronyms | conversation-init not firing | Verify OnActivity trigger with `ActivityType = "message"` and `TurnCount = 1` |
| Push fails with ConcurrencyVersionMismatch | Stale row versions | Pull first, then push |
| Test returns no data | Agent not published | Publish in Copilot Studio UI after push (draft ≠ published) |
| Wrong topic triggered | Overlapping trigger phrases | Review `modelDescription` — make it specific to each topic's intent |
| User context empty | M365 connector not authorized | Ensure the M365 Users connector connection is set up in Copilot Studio |
| "Extension not found" | Copilot Studio VS Code extension missing | `code --install-extension ms-CopilotStudio.vscode-copilotstudio` |

---

## Environment Reference

Replace these placeholders with your own values:

```
SP_POLICIES_URL=https://YOUR_TENANT.sharepoint.com/sites/ITSupport/Shared%20Documents/Policies
SP_GUIDES_URL=https://YOUR_TENANT.sharepoint.com/sites/ITSupport/Shared%20Documents/Guides
FAQ_URL=https://YOUR_COMPANY.com/it-support/faq

CS_TENANT_ID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
CS_ENVIRONMENT_ID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
CS_ENVIRONMENT_URL=https://your-org.crm.dynamics.com/
CS_AGENT_MGMT_URL=https://powervamg.your-region.gateway.prod.island.powerapps.com
CS_AGENT_ID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
CS_CLIENT_ID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

---

## Document History

| Version | Date | Author | Notes |
|---------|------|--------|-------|
| 1.0 | 2026-03-28 | Javier Armesto | Initial version — alternative quickstart without BC dependencies |
