# Order Inquiry Pattern

## Overview

Sales order lookup with status-based response branching. In most cases, this is better handled by the orchestrator + MCP directly — only create a custom topic when you need specific formatting, conditional logic (e.g., different responses for shipped vs processing), or an AdaptiveCard display.

## When to Use

- **Use this pattern** when you need status-based branching with different response formats per status
- **Use this pattern** when you want to display order details in an AdaptiveCard
- **Don't use** for simple "what's the status of order X?" — the orchestrator + MCP handles this directly
- **Don't use** for order lists — format via instructions instead

## When Orchestrator Is Sufficient

The orchestrator + BC MCP action can handle order inquiries when:
1. The response format is the same regardless of status
2. No conditional logic is needed based on the order status
3. No AdaptiveCard is required

In these cases, add to the agent instructions:
```
Cuando el usuario pregunte por el estado de un pedido, consultar en BC el pedido
por número o cliente. Mostrar: nº pedido, cliente, fecha, estado, importe total.
```

## Flow (Custom Topic)

```
AutomaticTaskInput (order reference or customer)
    ↓
Query sales orders (MCP action — orchestrator provides context)
    ↓
Order found? (ConditionGroup)
  ├─ Not found → "No se encontró el pedido" message
  └─ Found →
        ↓
    Status branching (ConditionGroup)
      ├─ Shipped → Delivery tracking info + estimated arrival
      ├─ Released → Processing confirmation + expected ship date
      ├─ Open → Draft status + next steps
      └─ Other → General status report
        ↓
    Output: order details + status context
```

## Key Design Decisions

1. **AutomaticTaskInput for order reference**: The orchestrator collects the order number or customer name from context. The `description` tells the orchestrator what to look for.

2. **Topic outputs, not SendActivity**: Status-based response text is set as a topic output variable, allowing the orchestrator to incorporate it into a natural response or chain with other actions.

3. **Consider not creating this topic**: For most agents, the orchestrator handles order inquiries well from instructions alone. Only create this topic if you have specific formatting or branching requirements.

## YAML Snippet: Status Branching

```yaml
kind: AdaptiveDialog
inputs:
  - kind: AutomaticTaskInput
    propertyName: OrderReference
    description: >-
      El número de pedido o nombre del cliente para consultar pedidos de venta.
      Puede ser un número de pedido (ej. S-ORD-1001), nombre de cliente,
      o número de cliente.
    entity: StringPrebuiltEntity
    shouldPromptUser: true

modelDescription: >-
  Consulta de pedidos de venta en Business Central. Usa este topic cuando el
  usuario pregunte por el estado de un pedido, detalles de un pedido de venta,
  o pedidos pendientes de un cliente. Proporciona respuestas diferenciadas según
  el estado del pedido.

beginDialog:
  kind: OnRecognizedIntent
  id: main
  intent:
    displayName: Consulta de Pedidos
    triggerQueries:
      - Estado del pedido
      - ¿Cómo va mi pedido?
      - Consultar pedido de venta
      - Pedidos pendientes del cliente
      - Detalles del pedido

  actions:
    # ── Order Found Check ───────────────────────────────
    - kind: ConditionGroup
      id: conditionGroup_REPLACE1
      conditions:
        - id: conditionItem_REPLACE2
          condition: =IsBlank(Topic.OrderStatus)
          actions:
            - kind: SetVariable
              id: setVariable_REPLACE3
              variable: Topic.OrderResponse
              value: >-
                ="No se encontró el pedido solicitado. Verifica el número de pedido
                o el nombre del cliente e intenta de nuevo."

      elseActions:
        # ── Status Branching ────────────────────────────
        - kind: ConditionGroup
          id: conditionGroup_REPLACE4
          conditions:
            - id: conditionItem_REPLACE5
              condition: =Topic.OrderStatus = "Shipped"
              actions:
                - kind: SetVariable
                  id: setVariable_REPLACE6
                  variable: Topic.OrderResponse
                  value: >-
                    =Concatenate(
                      "Pedido ", Topic.OrderReference, " — Estado: Enviado. ",
                      "El pedido ha sido despachado y está en tránsito."
                    )

            - id: conditionItem_REPLACE7
              condition: =Topic.OrderStatus = "Released"
              actions:
                - kind: SetVariable
                  id: setVariable_REPLACE8
                  variable: Topic.OrderResponse
                  value: >-
                    =Concatenate(
                      "Pedido ", Topic.OrderReference, " — Estado: Liberado. ",
                      "El pedido está aprobado y en proceso de preparación."
                    )

            - id: conditionItem_REPLACE9
              condition: =Topic.OrderStatus = "Open"
              actions:
                - kind: SetVariable
                  id: setVariable_REPLACE10
                  variable: Topic.OrderResponse
                  value: >-
                    =Concatenate(
                      "Pedido ", Topic.OrderReference, " — Estado: Abierto (borrador). ",
                      "El pedido aún no ha sido liberado para procesamiento."
                    )

          elseActions:
            - kind: SetVariable
              id: setVariable_REPLACE11
              variable: Topic.OrderResponse
              value: >-
                =Concatenate(
                  "Pedido ", Topic.OrderReference, " — Estado: ", Topic.OrderStatus, "."
                )

inputType:
  properties:
    OrderReference:
      displayName: OrderReference
      description: Número de pedido o identificación del cliente
      type: String
    OrderStatus:
      displayName: OrderStatus
      description: Estado del pedido obtenido de BC
      type: String

outputType:
  properties:
    OrderResponse:
      displayName: OrderResponse
      description: Respuesta formateada con detalles del estado del pedido
      type: String
```

## Notes

- **No full template provided** — this pattern is simple enough to build inline using `/copilot-studio:new-topic` + the snippet above
- **If you add an AdaptiveCard** for order details, use `/copilot-studio:add-adaptive-card` with an info card type showing order number, customer, date, status, and total amount
- **Status values** depend on BC configuration — the values above (`Open`, `Released`, `Shipped`) are standard BC sales order statuses but may vary
