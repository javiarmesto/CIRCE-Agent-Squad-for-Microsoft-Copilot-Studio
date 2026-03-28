# Agent Requirements: IT Support Knowledge Base

> This document defines the requirements for the IT Support KB agent.
> It serves as the initial context for the CIRCE framework before any development begins.
> The Conductor agent should read this document first and use it as reference throughout the build workflow.

---

## Agent Identity

| Property | Value |
|----------|-------|
| Name | IT Support KB |
| Purpose | Answer IT support questions from company documentation |
| Language | Spanish |
| Audience | Internal employees |
| Tone | Professional, concise, helpful |

## Data Sources

| Source | Type | URL |
|--------|------|-----|
| IT Policies | SharePoint | `https://YOUR_TENANT.sharepoint.com/sites/ITSupport/Shared%20Documents/Policies` |
| How-To Guides | SharePoint | `https://YOUR_TENANT.sharepoint.com/sites/ITSupport/Shared%20Documents/Guides` |
| FAQs | Public Website | `https://YOUR_COMPANY.com/it-support/faq` |

> **Note:** Replace the placeholder URLs above with your actual SharePoint site and FAQ page. The SharePoint URLs must be direct folder paths (not sharing links).

---

## Functional Requirements

### FR-1: Answer IT Questions from Knowledge

The user asks an IT support question. The agent searches the configured knowledge sources and returns a grounded answer with citations. If no relevant information is found, the agent informs the user and suggests contacting IT support directly.

### FR-2: Guided Entry Point

When the user says "necesito ayuda" or "soporte IT" without specifying a topic, the agent presents conversation starters:
- "¿Cómo conecto a la VPN?"
- "¿Cuál es la política de contraseñas?"
- "Tengo un problema con mi equipo"
- "¿Cómo solicito acceso a una aplicación?"

### FR-3: Glossary Expansion

The agent loads a glossary of internal IT acronyms at conversation start (JIT) so the orchestrator can expand abbreviations before searching knowledge sources. Example acronyms: VPN, MFA, SSO, ITSM, SLA, AD, BYOD.

### FR-4: User Context Awareness

The agent loads the user's M365 profile (department, country) at conversation start. This enables:
- Department-specific policy responses (e.g., different software catalogs per division)
- Country-aware answers (e.g., regional IT contacts)

### FR-5: Escalation to Human Agent

When the user says "hablar con una persona" or "agente humano", the agent escalates to a live IT support agent.

### FR-6: Feedback Collection

After answering a question, the agent asks: "¿Te ha resultado útil esta respuesta?" with Yes/No options. The response is logged for quality tracking.

---

## Non-Functional Requirements

| Requirement | Target |
|-------------|--------|
| Response time | < 5 seconds for knowledge search |
| Language | Spanish (all user-facing content) |
| Read-only | No write operations to any system |
| Authentication | Integrated (user auto-identified via M365) |
| Knowledge freshness | SharePoint content indexed automatically by Copilot Studio |

---

## Topics

| Topic | Trigger | Type |
|-------|---------|------|
| Conversation Init | First message | OnActivity (JIT glossary + user context) |
| Greeting | Conversation start | OnConversationStart |
| IT Knowledge Search | IT questions, "cómo", "qué es", "dónde encuentro" | OnRecognizedIntent + SearchAndSummarizeContent |
| Fallback | Unknown intent | OnUnknownIntent + SearchAndSummarizeContent |
| Error Handler | System error | OnError |
| Escalate | "hablar con una persona", "agente humano" | OnEscalate |

## Knowledge Sources

| Name | Type | URL |
|------|------|-----|
| IT Policies | SharePointSearchSource | SharePoint folder |
| How-To Guides | SharePointSearchSource | SharePoint folder |
| IT FAQs | PublicSiteSearchSource | Public FAQ page |

## Agent Instructions (Summary)

The agent should:
- Always respond in Spanish
- Ground answers in configured knowledge sources — never fabricate information
- Cite the source document when answering
- If unsure, say "No he encontrado información sobre eso en la documentación disponible. Te recomiendo contactar con el equipo de IT directamente."
- Use the glossary to expand acronyms before searching
- Personalize responses using the user's department and country when relevant
- Today's date is `{Text(Today(),DateTimeFormat.LongDate)}`

## Out of Scope

- Ticket creation in ITSM systems (read-only agent)
- Software installation or remote desktop access
- Password resets (direct the user to the self-service portal)
- Business Central integration
- Multi-language support (Spanish only for this version)

---
