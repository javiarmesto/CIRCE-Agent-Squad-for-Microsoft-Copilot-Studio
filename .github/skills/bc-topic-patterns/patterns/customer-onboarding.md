# Customer Onboarding Pattern

## Overview

Customer lookup and onboarding flow: identify a customer by email or number, query BC, display a profile card if found, or offer to connect with sales if not found. Uses the JIT user context pattern from best-practices when M365 profile is available for automatic identification.

## When to Use

- **Use this pattern** when the agent needs a structured customer welcome/identification flow with a visual profile card
- **Use this pattern** when combining JIT user context (M365 profile) with BC customer lookup
- **Don't use** for simple "who is customer X?" queries — the orchestrator + MCP handles those
- **Don't use** if no visual card is needed — configure the instructions to return customer details as text

## Flow

```
Check JIT user context (if M365 profile available)
    ↓
AutomaticTaskInput (customer email or number — fallback)
    ↓
Query customer in BC (MCP action)
    ↓
Customer found? (ConditionGroup)
  ├─ Not found → "Connect with sales" message → Output: escalation flag
  └─ Found →
        ↓
    AdaptiveCardPrompt — Customer Profile Card
      ├─ Header: customer name + number
      ├─ FactSet: email, phone, address, credit limit, balance
      └─ Welcome message
        ↓
    Output: customer data for orchestrator
```

## Key Design Decisions

1. **JIT user context integration**: If the agent uses the JIT user context pattern (from `/copilot-studio:best-practices`), the user's M365 email is available in `Global.UserEmail`. Use this to auto-identify the customer in BC without asking — the `description` on `AutomaticTaskInput` tells the orchestrator to use `Global.UserEmail` if available.

2. **AdaptiveCard for profile display**: A profile card provides a structured, visually clean summary of the customer's key data. This is more readable than a text-based response for multi-field profiles.

3. **Fallback to sales**: If the customer is not found in BC, the flow outputs an escalation flag instead of trying to create the customer. Customer creation in BC typically requires more data and validation than a chat-based form can provide.

4. **Topic outputs for chaining**: Customer data is output for the orchestrator to use in subsequent interactions — the agent "remembers" who the customer is for the rest of the conversation.

## YAML Snippet: Customer Onboarding Topic

```yaml
kind: AdaptiveDialog
inputs:
  - kind: AutomaticTaskInput
    propertyName: CustomerEmail
    description: >-
      El email del cliente para buscar en Business Central. Si el usuario tiene
      perfil M365 disponible (Global.UserEmail), usar ese email automáticamente.
      Si no, pedir al usuario su email o número de cliente.
    entity: EMailPrebuiltEntity
    shouldPromptUser: true

  - kind: AutomaticTaskInput
    propertyName: CustomerNumber
    description: >-
      El número de cliente en Business Central. Alternativa al email para
      identificar al cliente. Si el usuario proporciona un número de cliente,
      usar este campo en lugar del email.
    entity: StringPrebuiltEntity
    shouldPromptUser: false

modelDescription: >-
  Onboarding y bienvenida de cliente. Usa este topic cuando un nuevo usuario
  inicia conversación y necesita ser identificado como cliente en Business Central,
  o cuando el usuario pide ver su perfil de cliente, datos de cuenta, o información
  de contacto registrada.

beginDialog:
  kind: OnRecognizedIntent
  id: main
  intent:
    displayName: Onboarding de Cliente
    triggerQueries:
      - Soy cliente, ¿me reconoces?
      - Ver mi perfil de cliente
      - Datos de mi cuenta
      - Información de mi contacto
      - Quiero registrarme como cliente

  actions:
    # ── Customer Found Check ────────────────────────────
    - kind: ConditionGroup
      id: conditionGroup_REPLACE1
      conditions:
        - id: conditionItem_REPLACE2
          condition: =IsBlank(Topic.CustomerName)
          actions:
            - kind: SendActivity
              id: sendMessage_REPLACE3
              activity: |
                No hemos encontrado tu cuenta de cliente en nuestro sistema.

                **¿Qué puedes hacer?**
                - Verificar que el email o número de cliente sea correcto
                - Contactar con el equipo de ventas para crear tu cuenta

                Un representante de ventas puede ayudarte con el proceso de alta.

            - kind: SetVariable
              id: setVariable_REPLACE4
              variable: Topic.OnboardingStatus
              value: ="not-found"

            - kind: EndDialog
              id: endDialog_REPLACE5
              clearTopicQueue: true

    # ── Customer Profile Card ───────────────────────────
    - kind: AdaptiveCardPrompt
      id: adaptiveCardPrompt_REPLACE6
      card: |
        {
          "type": "AdaptiveCard",
          "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
          "version": "1.5",
          "body": [
            {
              "type": "TextBlock",
              "text": "👤 Perfil de Cliente",
              "weight": "Bolder",
              "size": "Large",
              "wrap": true
            },
            {
              "type": "ColumnSet",
              "columns": [
                {
                  "type": "Column",
                  "width": "stretch",
                  "items": [
                    {
                      "type": "TextBlock",
                      "text": "${CustomerName}",
                      "weight": "Bolder",
                      "size": "Medium",
                      "wrap": true
                    },
                    {
                      "type": "TextBlock",
                      "text": "Nº Cliente: ${CustomerNumber}",
                      "isSubtle": true,
                      "spacing": "None",
                      "wrap": true
                    }
                  ]
                }
              ]
            },
            {
              "type": "FactSet",
              "facts": [
                {
                  "title": "Email:",
                  "value": "${CustomerEmail}"
                },
                {
                  "title": "Teléfono:",
                  "value": "${CustomerPhone}"
                },
                {
                  "title": "Dirección:",
                  "value": "${CustomerAddress}"
                },
                {
                  "title": "Límite crédito:",
                  "value": "${CreditLimit}"
                },
                {
                  "title": "Saldo actual:",
                  "value": "${CurrentBalance}"
                }
              ]
            },
            {
              "type": "TextBlock",
              "text": "¡Bienvenido! Estoy listo para ayudarte con la gestión de tu cuenta.",
              "wrap": true,
              "spacing": "Medium"
            }
          ]
        }
      output:
        binding:
          acknowledged: Topic.Acknowledged
      outputType:
        properties:
          acknowledged:
            type: String

    - kind: SetVariable
      id: setVariable_REPLACE7
      variable: Topic.OnboardingStatus
      value: ="found"

inputType:
  properties:
    CustomerEmail:
      displayName: CustomerEmail
      description: Email del cliente
      type: String
    CustomerNumber:
      displayName: CustomerNumber
      description: Número de cliente en BC
      type: String
    CustomerName:
      displayName: CustomerName
      description: Nombre del cliente obtenido de BC
      type: String
    CustomerPhone:
      displayName: CustomerPhone
      description: Teléfono del cliente
      type: String
    CustomerAddress:
      displayName: CustomerAddress
      description: Dirección del cliente
      type: String
    CreditLimit:
      displayName: CreditLimit
      description: Límite de crédito del cliente
      type: String
    CurrentBalance:
      displayName: CurrentBalance
      description: Saldo actual del cliente
      type: String

outputType:
  properties:
    OnboardingStatus:
      displayName: OnboardingStatus
      description: >-
        Estado del onboarding: "found" si el cliente fue identificado,
        "not-found" si no se encontró en BC
      type: String
    CustomerName:
      displayName: CustomerName
      description: Nombre del cliente identificado
      type: String
    CustomerNumber:
      displayName: CustomerNumber
      description: Número del cliente identificado
      type: String
```

## JIT User Context Integration

If the agent uses the JIT user context pattern from `/copilot-studio:best-practices`:

1. `Global.UserEmail` is populated from the M365 profile on the first message
2. The orchestrator can use this email to auto-fill `CustomerEmail` without asking
3. Add to the agent instructions:
   ```
   Si Global.UserEmail está disponible, usar automáticamente para identificar
   al cliente en BC sin pedir el email al usuario.
   ```

This creates a seamless experience — the user starts chatting and the agent already knows who they are.

## Notes

- **Customer creation is out of scope**: If the customer isn't found, suggest contacting sales. Don't try to create the customer via chat — BC customer master data requires validation and approval workflows.
- **Profile card is display-only**: No submit button — this is an info card showing the customer's current data from BC.
- **Credit limit and balance** are shown for internal users only. If the agent is customer-facing, remove these fields from the card.
