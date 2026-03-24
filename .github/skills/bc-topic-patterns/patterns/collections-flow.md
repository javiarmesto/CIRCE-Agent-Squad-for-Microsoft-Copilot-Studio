# Collections Flow Pattern

## Overview

Deterministic collections workflow with NON-NEGOTIABLE dispute protection. This pattern enforces the agent's protection rules as a ConditionGroup in the topic YAML — providing double protection (instructions + deterministic check) against sending external communications when disputes or inconsistencies exist.

## When to Use

- **Use this pattern** when you need the dispute protection rule enforced deterministically, not just by instruction
- **Don't use** for simple balance queries — let the orchestrator + MCP handle those
- **Don't use** for aging breakdowns without communication — format via instructions instead

## Flow

```
AutomaticTaskInput (customer name/number)
    ↓
Query customer balance (MCP action output)
    ↓
Check disputes (ConditionGroup — NON-NEGOTIABLE)
  ├─ Dispute active → Internal reconciliation report (SendActivity) → END
  ├─ Inconsistency >5% or >1.000€ → Internal reconciliation report → END
  └─ Clean →
        ↓
    Query overdue invoices (MCP action output)
        ↓
    Aging-based branching (ConditionGroup)
      ├─ >90d → Escalation template + schedule meeting
      ├─ 61-90d → Formal reminder + schedule call
      ├─ 31-60d → Standard reminder email
      ├─ 1-30d → Courtesy reminder
      └─ Current → No action needed
        ↓
    Output: reminder action + customer context
```

## Key Design Decisions

1. **Dispute check in topic, not just instructions**: Instructions tell the orchestrator not to send communications when disputes exist, but the orchestrator is probabilistic. The ConditionGroup check is deterministic — it physically blocks the flow from reaching communication nodes. This is double protection per CIRCE protection rules.

2. **Reminder messages as topic outputs**: The reminder text is output as a topic variable, not sent directly via SendActivity. This allows the orchestrator to compose it with other actions (e.g., send via Outlook MCP, schedule a follow-up meeting).

3. **AutomaticTaskInput for customer identification**: The orchestrator collects the customer identifier from conversation context. No explicit Question node — the `description` guides the orchestrator on what to collect.

4. **Aging buckets match agent instructions**: Current, 1-30d, 31-60d, 61-90d, >90d — matching the `agent.mcs.yml` KPI definitions.

## YAML Snippets

### Topic Structure with Dispute Check

```yaml
kind: AdaptiveDialog
inputs:
  - kind: AutomaticTaskInput
    propertyName: CustomerIdentifier
    description: >-
      El nombre o número del cliente para la gestión de cobros.
      Puede ser el nombre completo, número de cliente, o cualquier identificador.
    entity: StringPrebuiltEntity
    shouldPromptUser: true

modelDescription: >-
  Gestión de cobros con verificación de disputas. Usa este topic cuando el usuario
  pida enviar un recordatorio de pago, gestionar cobros de un cliente, o hacer
  seguimiento de facturas vencidas. Incluye verificación obligatoria de disputas
  y clasificación por antigüedad.

beginDialog:
  kind: OnRecognizedIntent
  id: main
  intent:
    displayName: Gestión de Cobros
    triggerQueries:
      - Envía recordatorio de pago al cliente
      - Gestiona cobros del cliente
      - Seguimiento de facturas vencidas
      - Recordatorio de cobro
      - Gestionar cartera del cliente

  actions:
    # ── Dispute Check (NON-NEGOTIABLE) ──────────────────
    # The orchestrator has already queried BC via MCP at this point.
    # This condition checks the dispute status from the MCP response.
    # If dispute data is not available as a topic variable, the orchestrator
    # will have included it in the conversation context.
    - kind: ConditionGroup
      id: conditionGroup_REPLACE1
      conditions:
        - id: conditionItem_REPLACE2
          condition: =Topic.HasDispute = true
          actions:
            - kind: SendActivity
              id: sendMessage_REPLACE3
              activity: |
                ⚠️ **Alerta: Disputa activa detectada**

                No se pueden enviar comunicaciones externas para este cliente.
                Motivo: existe una disputa activa registrada en Business Central.

                **Acciones recomendadas:**
                - Revisar el detalle de la disputa con finanzas
                - Consultar el historial de disputas del cliente
                - Considerar crear un caso de escalado interno

            - kind: EndDialog
              id: endDialog_REPLACE4
              clearTopicQueue: true

    # ── Aging-Based Branching ───────────────────────────
    - kind: ConditionGroup
      id: conditionGroup_REPLACE5
      conditions:
        - id: conditionItem_REPLACE6
          condition: =Topic.MaxOverdueDays > 90
          actions:
            - kind: SetVariable
              id: setVariable_REPLACE7
              variable: Topic.ReminderAction
              value: ="escalation"

            - kind: SetVariable
              id: setVariable_REPLACE8
              variable: Topic.ReminderMessage
              value: >-
                =Concatenate(
                  "Escalación urgente — Cliente: ", Topic.CustomerIdentifier,
                  ". Facturas vencidas >90 días. Se recomienda reunión de escalado."
                )

        - id: conditionItem_REPLACE9
          condition: =Topic.MaxOverdueDays > 60
          actions:
            - kind: SetVariable
              id: setVariable_REPLACE10
              variable: Topic.ReminderAction
              value: ="formal-reminder"

            - kind: SetVariable
              id: setVariable_REPLACE11
              variable: Topic.ReminderMessage
              value: >-
                =Concatenate(
                  "Recordatorio formal — Cliente: ", Topic.CustomerIdentifier,
                  ". Facturas vencidas 61-90 días. Se recomienda llamada de seguimiento."
                )

        - id: conditionItem_REPLACE12
          condition: =Topic.MaxOverdueDays > 30
          actions:
            - kind: SetVariable
              id: setVariable_REPLACE13
              variable: Topic.ReminderAction
              value: ="standard-reminder"

            - kind: SetVariable
              id: setVariable_REPLACE14
              variable: Topic.ReminderMessage
              value: >-
                =Concatenate(
                  "Recordatorio estándar — Cliente: ", Topic.CustomerIdentifier,
                  ". Facturas vencidas 31-60 días."
                )

        - id: conditionItem_REPLACE15
          condition: =Topic.MaxOverdueDays > 0
          actions:
            - kind: SetVariable
              id: setVariable_REPLACE16
              variable: Topic.ReminderAction
              value: ="courtesy-reminder"

            - kind: SetVariable
              id: setVariable_REPLACE17
              variable: Topic.ReminderMessage
              value: >-
                =Concatenate(
                  "Recordatorio de cortesía — Cliente: ", Topic.CustomerIdentifier,
                  ". Facturas vencidas 1-30 días."
                )

      elseActions:
        - kind: SetVariable
          id: setVariable_REPLACE18
          variable: Topic.ReminderAction
          value: ="none"

        - kind: SetVariable
          id: setVariable_REPLACE19
          variable: Topic.ReminderMessage
          value: ="No hay facturas vencidas para este cliente. No se requiere acción."

inputType:
  properties:
    CustomerIdentifier:
      displayName: CustomerIdentifier
      description: El nombre o número del cliente
      type: String
    HasDispute:
      displayName: HasDispute
      description: Si el cliente tiene una disputa activa en BC
      type: Boolean
    MaxOverdueDays:
      displayName: MaxOverdueDays
      description: Máximo número de días de vencimiento entre las facturas del cliente
      type: Number

outputType:
  properties:
    ReminderAction:
      displayName: ReminderAction
      description: >-
        Tipo de acción de recordatorio recomendada: escalation, formal-reminder,
        standard-reminder, courtesy-reminder, o none
      type: String
    ReminderMessage:
      displayName: ReminderMessage
      description: Mensaje de recordatorio generado para el cliente
      type: String
```

### Why Topic Outputs Instead of SendActivity

The `ReminderAction` and `ReminderMessage` outputs let the orchestrator decide what to do next:
- If `ReminderAction = "escalation"` → orchestrator may schedule a meeting via Outlook MCP
- If `ReminderAction = "standard-reminder"` → orchestrator may send an email via Outlook MCP
- If `ReminderAction = "none"` → orchestrator reports no action needed

This is the correct pattern under generative orchestration — the topic prepares data, the orchestrator composes the final action.

## Full Template

See [../templates/collections-reminder.topic.mcs.yml](../templates/collections-reminder.topic.mcs.yml) for the complete AdaptiveDialog YAML with all placeholder IDs.
