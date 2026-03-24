---
user-invocable: false
name: scaffold-agent
description: >
  Scaffolds a complete Copilot Studio agent from scratch with greeting, fallback,
  and error handler topics. Use when no agent exists in the workspace and the user
  wants to create one from scratch instead of cloning.
argument-hint: <agent-name> <language> <description>
---

# Scaffold Agent

Creates a complete, functional Copilot Studio agent project from scratch.

## When to use

The Author Agent invokes this when `Glob: **/agent.mcs.yml` finds **no agent** and
the user chooses to scaffold instead of cloning.

## Instructions

### 1. Gather input

If not provided in the arguments, ask the user for:

| Parameter | Required | Default | Example |
|-----------|----------|---------|---------|
| **Display Name** | Yes | — | `Circe Collections Bot` |
| **Language** | No | Spanish (`1034`) | `English` → `1033`, `Spanish` → `1034` |
| **Description** | Yes | — | `Agente de gestión de cobros` |
| **Schema Name Prefix** | Yes | — | `copilots_header_cra1e_Circe` |
| **Generative Actions** | No | `true` | `true` / `false` |

Explain the schema name prefix:
> "El **schema name** es el identificador interno del agente en Power Platform. Se usa para
> referencias cruzadas entre topics (e.g. `<schemaName>.topic.Escalate`). Normalmente tiene
> el formato `copilots_header_<id>_<Nombre>`. Puedes encontrarlo en Copilot Studio > Settings > Agent details."

### 2. Determine the target directory

The agent files will be created in the **workspace root**. Verify the workspace root is empty
or does not already contain `.mcs.yml` files. If it does, STOP and warn the user.

Set `AGENT_DIR` to the workspace root.
Set `SCHEMA` to the user-provided schema name prefix.

### 3. Generate agent files

Create each file using the templates below. For **every** `_REPLACE` placeholder, generate
a unique random ID with 6-8 alphanumeric characters.

#### 3a. `agent.mcs.yml`

```yaml
mcs.metadata:
  componentName: <Display Name>
kind: GptComponentMetadata
displayName: <Display Name>
instructions: |
  <User-provided description. If the language is Spanish, write instructions in Spanish.>
conversationStarters:
  - title: Empezar
    text: ¿En qué puedes ayudarme?

  - title: Preguntas frecuentes
    text: ¿Qué preguntas puedes responder?

aISettings:
  model:
    modelNameHint: GPT5Chat
```

If the language is NOT Spanish, adjust `conversationStarters` accordingly.

The `instructions` field should contain:
1. Bot identity (name, role) based on the user-provided description
2. Response guidelines (concise, clear, language)
3. Escalation instruction
4. A `{Text(Today(),DateTimeFormat.LongDate)}` date injection line

#### 3b. `settings.mcs.yml`

```yaml
displayName: <Display Name>
schemaName: <SCHEMA>
accessControlPolicy: Any
authenticationMode: Integrated
authenticationTrigger: Always
configuration:
  settings:
    GenerativeActionsEnabled: <true/false>

  isLightweightBot: false
  aISettings:
    useModelKnowledge: true
    isFileAnalysisEnabled: true
    isSemanticSearchEnabled: true
    optInUseLatestModels: false

  recognizer:
    kind: GenerativeAIRecognizer

  analyticsSettings:
    roiCalculation:
      kind: SessionBasedRoiCalculationSettings
      currencyCode: EUR
      savedCost: 10
      savedTimeInSeconds: 1800

template: default-2.1.0
language: <language-code>
```

Use `1034` for Spanish, `1033` for English, or the appropriate LCID.

#### 3c. `connectionreferences.mcs.yml`

Create an empty connection references file:

```yaml
connectionReferences: []
```

#### 3d. Topics directory: `topics/`

Create the following system topics. **All user-facing text must be in the chosen language.**

##### `topics/ConversationStart.mcs.yml`

Based on the greeting template. Customize the greeting message to reflect the agent's purpose.

```yaml
mcs.metadata:
  componentName: <"Inicio de la conversación" (ES) / "Conversation Start" (EN)>
  description: <System topic description in the chosen language>
kind: AdaptiveDialog
beginDialog:
  kind: OnConversationStart
  id: main
  actions:
    - kind: SendActivity
      id: sendMessage_<RANDOM>
      activity:
        text:
          - >-
            <Greeting message referencing {System.Bot.Name} and the agent's purpose>
```

##### `topics/Fallback.mcs.yml`

Based on the fallback template. Includes escalation after 3 failures.

```yaml
mcs.metadata:
  componentName: <"Alternativa" (ES) / "Fallback" (EN)>
  description: <System topic description in the chosen language>
kind: AdaptiveDialog
beginDialog:
  kind: OnUnknownIntent
  id: main
  actions:
    - kind: ConditionGroup
      id: conditionGroup_<RANDOM>
      conditions:
        - id: conditionItem_<RANDOM>
          condition: =System.FallbackCount < 3
          actions:
            - kind: SendActivity
              id: sendMessage_<RANDOM>
              activity: <"Lo siento, no sé muy bien cómo ayudarle. ¿Puede intentar decirlo de otra forma?" (ES)>

      elseActions:
        - kind: BeginDialog
          id: beginDialog_<RANDOM>
          dialog: <SCHEMA>.topic.Escalate
```

##### `topics/OnError.mcs.yml`

Based on the error-handler template. Includes test-mode detection and telemetry logging.

```yaml
mcs.metadata:
  componentName: <"Al producirse un error" (ES) / "On Error" (EN)>
  description: <System topic description in the chosen language>
kind: AdaptiveDialog
startBehavior: UseLatestPublishedContentAndCancelOtherTopics
beginDialog:
  kind: OnError
  id: main
  actions:
    - kind: SetVariable
      id: setVariable_<RANDOM>
      variable: init:Topic.CurrentTime
      value: =Text(Now(), DateTimeFormat.UTC)

    - kind: ConditionGroup
      id: conditionGroup_<RANDOM>
      conditions:
        - id: conditionItem_<RANDOM>
          condition: =System.Conversation.InTestMode = true
          actions:
            - kind: SendActivity
              id: sendMessage_<RANDOM>
              activity: |-
                Error: {System.Error.Message}
                Code: {System.Error.Code}
                Conversation: {System.Conversation.Id}
                Time (UTC): {Topic.CurrentTime}

      elseActions:
        - kind: SendActivity
          id: sendMessage_<RANDOM>
          activity:
            text:
              - |-
                <Error message in the chosen language with {System.Error.Code}, {System.Conversation.Id}, {Topic.CurrentTime}>
            speak:
              - <Spoken error message in the chosen language>

    - kind: LogCustomTelemetryEvent
      id: logTelemetry_<RANDOM>
      eventName: OnErrorLog
      properties: "={ErrorMessage: System.Error.Message, ErrorCode: System.Error.Code, TimeUTC: Topic.CurrentTime, ConversationId: System.Conversation.Id}"

    - kind: CancelAllDialogs
      id: cancelAllDialogs_<RANDOM>
```

##### `topics/Escalate.mcs.yml`

Simple escalation topic.

```yaml
mcs.metadata:
  componentName: <"Remitir a un superior" (ES) / "Escalate" (EN)>
  description: <System topic description in the chosen language>
kind: AdaptiveDialog
startBehavior: CancelOtherTopics
beginDialog:
  kind: OnEscalate
  id: main
  intent:
    displayName: <"Remitir a un superior" (ES) / "Escalate" (EN)>
    includeInOnSelectIntent: false
    triggerQueries:
      - <5-10 trigger phrases in the chosen language for "talk to an agent", "speak to a person", etc.>
  actions:
    - kind: SendActivity
      id: sendMessage_<RANDOM>
      activity:
        text:
          - <Escalation message in the chosen language>
```

##### `topics/Goodbye.mcs.yml`

```yaml
mcs.metadata:
  componentName: <"Despedida" (ES) / "Goodbye" (EN)>
  description: <System topic description in the chosen language>
kind: AdaptiveDialog
beginDialog:
  kind: OnRecognizedIntent
  id: main
  intent:
    displayName: <"Despedida" (ES) / "Goodbye" (EN)>
    triggerQueries:
      - <5-10 goodbye phrases in the chosen language>
  actions:
    - kind: SendActivity
      id: sendMessage_<RANDOM>
      activity:
        text:
          - <Goodbye message in the chosen language>
    - kind: EndDialog
      id: endDialog_<RANDOM>
```

##### `topics/ThankYou.mcs.yml`

```yaml
mcs.metadata:
  componentName: <"Gracias" (ES) / "Thank You" (EN)>
  description: <System topic description in the chosen language>
kind: AdaptiveDialog
beginDialog:
  kind: OnRecognizedIntent
  id: main
  intent:
    displayName: <"Gracias" (ES) / "Thank You" (EN)>
    triggerQueries:
      - <5-10 thank-you phrases in the chosen language>
  actions:
    - kind: SendActivity
      id: sendMessage_<RANDOM>
      activity:
        text:
          - <You're welcome message in the chosen language>
```

##### `topics/StartOver.mcs.yml`

```yaml
mcs.metadata:
  componentName: <"Empezar de nuevo" (ES) / "Start Over" (EN)>
  description: <System topic description in the chosen language>
kind: AdaptiveDialog
beginDialog:
  kind: OnRecognizedIntent
  id: main
  intent:
    displayName: <"Empezar de nuevo" (ES) / "Start Over" (EN)>
    triggerQueries:
      - <5-10 start-over phrases in the chosen language>
  actions:
    - kind: SendActivity
      id: sendMessage_<RANDOM>
      activity:
        text:
          - <Confirmation message that conversation is restarting, in the chosen language>
    - kind: ClearAllVariables
      id: clearVariables_<RANDOM>
    - kind: BeginDialog
      id: beginDialog_<RANDOM>
      dialog: <SCHEMA>.topic.ConversationStart
```

##### `topics/ResetConversation.mcs.yml`

```yaml
mcs.metadata:
  componentName: <"Restablecer conversación" (ES) / "Reset Conversation" (EN)>
  description: <System topic description in the chosen language>
kind: AdaptiveDialog
beginDialog:
  kind: OnSystemRedirect
  id: main
  actions:
    - kind: ClearAllVariables
      id: clearVariables_<RANDOM>
    - kind: BeginDialog
      id: beginDialog_<RANDOM>
      dialog: <SCHEMA>.topic.ConversationStart
```

##### `topics/EndofConversation.mcs.yml`

```yaml
mcs.metadata:
  componentName: <"Fin de la conversación" (ES) / "End of Conversation" (EN)>
  description: <System topic description in the chosen language>
kind: AdaptiveDialog
beginDialog:
  kind: OnRecognizedIntent
  id: main
  intent:
    displayName: <"Fin de la conversación" (ES) / "End of Conversation" (EN)>
    triggerQueries:
      - <3-5 end-conversation phrases in the chosen language>
  actions:
    - kind: SendActivity
      id: sendMessage_<RANDOM>
      activity:
        text:
          - <End conversation message in the chosen language>
    - kind: EndDialog
      id: endDialog_<RANDOM>
```

##### `topics/MultipleTopicsMatched.mcs.yml` (optional — include if generative orchestration is disabled)

If `GenerativeActionsEnabled: false`, include the disambiguation topic based on the
disambiguation template. If `true`, skip this — the orchestrator handles disambiguation.

##### `topics/Search.mcs.yml` (optional — include if generative orchestration is enabled)

If `GenerativeActionsEnabled: true`, include a knowledge search topic based on the
search-topic template to handle fallback with generative answers from knowledge sources.

#### 3e. Empty directories

Create empty placeholder directories (with `.gitkeep` files):

- `actions/`
- `knowledge/`
- `variables/`
- `workflows/`

### 4. Validate all generated files

After creating all files, validate each `.mcs.yml` file:

```bash
node .github/scripts/schema-lookup.bundle.js validate <file>
```

Fix any validation errors before proceeding.

### 5. Update memory

Invoke `/copilot-studio:update-memory` with:
- Operation: `agent-edited`
- Component: `<Display Name>`
- Rationale: `Agente scaffolded desde cero con topics de sistema básicos`

### 6. Report to the user

After successful scaffold, present a summary:

> **Agente scaffolded correctamente** ✓
>
> | Archivo | Descripción |
> |---------|------------|
> | `agent.mcs.yml` | Metadata del agente |
> | `settings.mcs.yml` | Configuración (schema, auth, generative actions) |
> | `connectionreferences.mcs.yml` | Referencias de conexión (vacío) |
> | `topics/ConversationStart.mcs.yml` | Saludo inicial |
> | `topics/Fallback.mcs.yml` | Respuesta cuando no se entiende |
> | `topics/OnError.mcs.yml` | Manejo de errores |
> | `topics/Escalate.mcs.yml` | Escalado a agente humano |
> | `topics/Goodbye.mcs.yml` | Despedida |
> | `topics/ThankYou.mcs.yml` | Agradecimiento |
> | `topics/StartOver.mcs.yml` | Reiniciar conversación |
> | `topics/ResetConversation.mcs.yml` | Reset del sistema |
> | `topics/EndofConversation.mcs.yml` | Fin de conversación |
>
> **Próximos pasos:**
> 1. Revisa las instrucciones del agente en `agent.mcs.yml`
> 2. Configura las conexiones que necesites (MCP, Outlook, BC) con `/copilot-studio:add-action`
> 3. Añade fuentes de conocimiento con `/copilot-studio:add-knowledge`
> 4. Crea topics personalizados con `/copilot-studio:new-topic`
> 5. Haz push con `/copilot-studio:copilot-studio-manage push`
>
> **⚠️ Importante:** Este scaffold crea los archivos YAML locales. Para que funcione en
> Copilot Studio, necesitas hacer push y luego publicar en la UI de Copilot Studio.
> El push requiere que el agente ya exista en el entorno de Power Platform — si es un
> agente completamente nuevo, primero créalo en la UI de Copilot Studio (vacío) y luego
> haz push de estos archivos.

## Language Reference

| Language | LCID | Sample greeting |
|----------|------|----------------|
| Spanish | `1034` | `Hola, soy {System.Bot.Name}. ¿En qué puedo ayudarte?` |
| English | `1033` | `Hello! I'm {System.Bot.Name}. How can I help you?` |
| French | `1036` | `Bonjour ! Je suis {System.Bot.Name}. Comment puis-je vous aider ?` |
| Portuguese | `1046` | `Olá! Sou o {System.Bot.Name}. Como posso ajudá-lo?` |
| German | `1031` | `Hallo! Ich bin {System.Bot.Name}. Wie kann ich Ihnen helfen?` |

## Notes

- **No external dependencies** — only creates files and invokes the schema validation script
- All `<RANDOM>` placeholders must be replaced with unique 6-8 character alphanumeric IDs
- All `<SCHEMA>` placeholders must be replaced with the user-provided schema name prefix
- Cross-references between topics use `<SCHEMA>.topic.<TopicName>` format
- The scaffold produces a local-only agent — push and publish are separate steps
