---
user-invocable: false
name: bc-mcp-setup
description: Guide users through the complete Business Central MCP Server setup — from enabling the feature in BC to connecting it in Copilot Studio and pulling locally.
argument-hint: <optional: sandbox or production, agent purpose>
---

# BC MCP Server Setup (Guide)

This skill guides users through the end-to-end process of configuring the Business Central MCP Server and connecting it to a Copilot Studio agent. **It does NOT write YAML directly** — MCP connections must be created through the BC admin and Copilot Studio portals.

## Why This Is a Guide, Not a Generator

The BC MCP connection requires:
1. **Feature enablement** in Business Central's Feature Management
2. **MCP Server Configuration** — created in the BC web client (page 8351)
3. **Connection reference** — authenticated link created in Copilot Studio
4. Once the connection is added via the UI and pulled locally, the action YAML can be edited with `/copilot-studio:configure-mcp-action`

## Reference

For detailed field-by-field reference, Dynamic Tool Mode comparison, tool naming conventions, and security recommendations, see:

```
Read: ${CLAUDE_SKILL_DIR}/mcp-config-guide.md
```

## Instructions

### Phase 1 — Prerequisites Check

Before starting, verify the user has:

> **Verificación de requisitos previos:**
>
> | Requisito | Detalle |
> |-----------|---------|
> | BC version | **27+** (2025 Wave 2 minimum — MCP requires this version) |
> | Copilot Studio license | Con capacidad de Copilot Credits disponible |
> | Permission set | **MCP - ADMIN** asignado al usuario en BC |
> | Entorno | ¿Vas a configurar en **sandbox** o **producción**? |
>
> ¿Cumples con todos estos requisitos?

If the user is unsure about the BC version, guide them:
- BC admin center → Environments → check the version column
- Version 27.x = 2025 Wave 2, version 26.x = 2025 Wave 1 (NOT supported)

If the user doesn't have MCP-ADMIN:
- BC → Users → select the user → Permission Sets → add `MCP - ADMIN`

### Phase 2 — Enable MCP in Business Central

Walk the user through enabling the MCP feature:

> **Habilitar MCP Server en Business Central:**
>
> 1. Abre Business Central
> 2. Busca y abre la página **Feature Management** (página 2610)
>    - O navega directamente: `https://businesscentral.dynamics.com/?page=2610`
> 3. Busca: **"Feature: Enable MCP Server access"**
> 4. Cambia el estado a **Enabled** (for all users)
> 5. Si se requiere, reinicia la sesión de BC
>
> ⚠️ Una vez habilitado, el MCP server proporciona acceso de **solo lectura** a todas las API pages expuestas por defecto. Para habilitar operaciones de escritura, necesitas crear una configuración específica (siguiente paso).

### Phase 3 — Configure MCP Server in Business Central

Guide the user through creating an MCP Server configuration:

> **Crear configuración del MCP Server:**
>
> 1. Busca y abre **"Model Context Protocol (MCP) Server Configurations"** (página 8351)
>    - O navega: `https://businesscentral.dynamics.com/?page=8351`
> 2. Haz clic en **New**
> 3. Configura los campos generales:

Present the general fields:

> | Campo | Valor recomendado | Descripción |
> |-------|-------------------|-------------|
> | **Name** | Nombre descriptivo que coincida con el propósito del agente (e.g., `CIRCE`, `COLLECTIONS-AGENT`) | Este nombre aparece en Copilot Studio al asignar la configuración |
> | **Description** | Breve descripción del propósito | e.g., "Agente de cobros y pagos" |
> | **Active** | ✅ On | La configuración debe estar activa para que los agentes la usen |
> | **Dynamic Tool Mode** | Ver recomendación abajo | Controla cómo se exponen las herramientas |
> | **Unblock Edit Tools** | ✅ On (si necesitas escritura) | Habilita operaciones de creación, modificación y eliminación |

#### Dynamic Tool Mode Decision

Ask the user about their scenario:

> **¿Habilitar Dynamic Tool Mode?**
>
> | | Dynamic Tool Mode **ON** | Dynamic Tool Mode **OFF** |
> |---|---|---|
> | **Uso ideal** | Prototipado, desarrollo, muchas APIs | Producción, control preciso |
> | **Cómo funciona** | Las herramientas se descubren dinámicamente en runtime (`bc_actions_search`, `bc_actions_describe`, `bc_actions_invoke`) | Las herramientas se listan explícitamente en Copilot Studio |
> | **Límite de 70 tools** | No aplica — discovery dinámico | Aplica — solo las primeras 70 herramientas están disponibles |
> | **Ventaja** | Acceso a todas las APIs sin límite | Control granular, rendimiento predecible |
> | **Desventaja** | Menos control, más tokens consumidos en discovery | Límite de 70 tools, configuración manual |
>
> **Recomendación para Circe**: OFF en producción (control preciso de las APIs expuestas), ON durante prototipado.
>
> Si activas Dynamic Tool Mode, el campo **Discover Additional Objects** permite dar acceso de lectura a API pages no incluidas explícitamente.

#### Add API Page Objects (Tools)

Based on the agent's purpose, recommend specific API pages:

> **Selección de API pages según el tipo de agente:**

**Collections / Finance agent (like Circe):**
| API Page | Page ID | Allow Read | Allow Create | Allow Modify | Allow Delete |
|----------|---------|:---:|:---:|:---:|:---:|
| APIV2 - Customers | 30009 | ✅ | ❌ | ❌ | ❌ |
| APIV2 - Customer Ledger Entries | — | ✅ | ❌ | ❌ | ❌ |
| APIV2 - Sales Invoices | 30023 | ✅ | ❌ | ❌ | ❌ |
| APIV2 - Customer Payment Journals | 30028 | ✅ | ✅ | ✅ | ✅ |
| APIV2 - Aged Accounts Receivable | 30037 | ✅ | ❌ | ❌ | ❌ |

**Sales agent:**
| API Page | Allow Read | Allow Create | Allow Modify | Allow Delete |
|----------|:---:|:---:|:---:|:---:|
| APIV2 - Customers | ✅ | ✅ | ✅ | ❌ |
| APIV2 - Items | ✅ | ❌ | ❌ | ❌ |
| APIV2 - Sales Orders | ✅ | ✅ | ✅ | ✅ |
| APIV2 - Sales Quotes | ✅ | ✅ | ✅ | ✅ |
| APIV2 - Sales Invoices | ✅ | ❌ | ❌ | ❌ |

**Support agent:**
| API Page | Allow Read | Allow Create | Allow Modify | Allow Delete |
|----------|:---:|:---:|:---:|:---:|
| APIV2 - Customers | ✅ | ❌ | ❌ | ❌ |
| APIV2 - Sales Orders | ✅ | ❌ | ❌ | ❌ |
| APIV2 - Items | ✅ | ❌ | ❌ | ❌ |
| Custom incident/support APIs | ✅ | ✅ | ✅ | ❌ |

> ⚠️ **Buena práctica de seguridad**: NO uses "Add All Standard APIs" en producción. Crea API pages específicas para tu agente que limiten los campos expuestos. Principio de mínimo privilegio — expón solo lo que el agente necesita.

Ask the user which APIs to add:

> ¿Qué tipo de agente estás construyendo? Te recomendaré las API pages apropiadas. También puedes indicarme API pages personalizadas que hayas creado en tu extensión AL.

### Phase 4 — Connect in Copilot Studio

Walk the user through the Copilot Studio connection:

> **Conectar el MCP Server en Copilot Studio:**
>
> 1. Abre [Copilot Studio](https://copilotstudio.microsoft.com)
> 2. Navega a tu agente
> 3. Ve a la pestaña **Tools** (Herramientas)
> 4. Haz clic en **+ Add a tool**
> 5. Busca **"Dynamics 365 Business Central MCP Server (Preview)"**
> 6. Si aparece "Not connected", haz clic y selecciona **Create new connection**
> 7. Autentícate con tu cuenta de BC
> 8. Selecciona **Add and configure**
> 9. En la sección **Inputs**, configura:
>
> | Campo | Valor |
> |-------|-------|
> | **Environment** | Selecciona tu entorno BC (e.g., `SANDBOX_US`, `PRODUCTION`) |
> | **Company** | Selecciona la empresa (e.g., `CRONUS USA, Inc.`) |
> | **MCP Server Configuration** | Nombre de la configuración creada en el paso anterior (e.g., `CIRCE`) — déjalo vacío para acceso de solo lectura a todas las APIs |
>
> 10. Revisa la sección **Tools** para verificar las herramientas disponibles:
>     - Con Dynamic Tool Mode OFF: verás herramientas individuales como `ListAPIV2 - Customer_PAG30009`
>     - Con Dynamic Tool Mode ON: verás `bc_actions_search`, `bc_actions_describe`, `bc_actions_invoke`
> 11. Haz clic en **Save**

#### Authentication Mode

> **Modo de autenticación:**
>
> | Modo | Cuándo usar |
> |------|------------|
> | **Invoker** (delegado) | ✅ Recomendado para producción. Usa los permisos del usuario que interactúa con el agente |
> | **Maker** (service-to-service) | Para agentes del sistema que operan sin contexto de usuario |
>
> **Recomendación**: Usa **Invoker** para respetar los permisos de BC del usuario final.

### Phase 5 — Validate

Guide the user through validation:

> **Validar la conexión:**
>
> 1. En Copilot Studio, haz clic en **Test** (esquina superior derecha)
> 2. Escribe: **"Muéstrame el cliente 10000"** (o "Show me customer 10000")
> 3. Verifica que:
>    - El agente invoca la herramienta MCP correcta
>    - Se devuelven datos del cliente
>    - Los campos visibles corresponden a la configuración de API pages
>
> **Pruebas adicionales sugeridas:**
> - "¿Cuántos clientes tengo?" (verifica acceso de lectura)
> - "Muéstrame las facturas pendientes del cliente 10000" (verifica API de facturas)
> - Si habilitaste escritura: "Crea un pago de 100€ para el cliente 10000" (verifica operaciones de escritura)

If validation fails, consult the troubleshooting table at the end of this skill.

### Phase 6 — Pull to Local

After successful validation, pull the agent files locally:

> **Sincronizar archivos localmente:**
>
> 1. En VS Code, usa la extensión de Copilot Studio: **Source Control → Pull**
> 2. Verifica que `connectionreferences.mcs.yml` incluye la conexión BC MCP
> 3. Verifica que existe un archivo de acción BC en `actions/`:
>    ```
>    Glob: **/actions/*BusinessCentral*.mcs.yml
>    ```
> 4. Si necesitas modificar la configuración del acción MCP, usa `/copilot-studio:configure-mcp-action`

After the user confirms the pull:
- Read `connectionreferences.mcs.yml` to verify the BC connection reference exists
- Read the BC action file to verify MCP configuration (environment, company, configurationName)
- Suggest updating `circe-memory.md` with the MCP config details using `/copilot-studio:update-memory`

## Troubleshooting

| Síntoma | Causa probable | Solución |
|---------|---------------|----------|
| BC no aparece en la búsqueda de herramientas MCP | Feature no habilitada o entorno incompatible | Verificar Feature Management → "Enable MCP Server access" está Enabled. Verificar BC version ≥ 27 |
| No hay herramientas disponibles después de conectar | Configuración MCP no activa o sin API pages | Abrir página 8351 en BC → verificar que **Active** está ON y que hay API pages añadidas |
| Error de autenticación al conectar | Credenciales inválidas o permisos insuficientes | Verificar que el usuario tiene MCP-ADMIN y licencia válida de BC |
| No se devuelven datos | Company o Environment incorrectos | Verificar que los valores de Environment y Company coinciden exactamente con BC admin center |
| Datos parciales o campos faltantes | API page no expone esos campos | Revisar la API page en BC — los campos deben estar definidos en el objeto de API. Crear API page personalizada si es necesario |
| Error "Company mismatch" | El nombre de empresa no coincide | Usar el nombre exacto de la empresa incluyendo mayúsculas y puntuación (e.g., "CRONUS USA, Inc." no "Cronus USA") |
| Operaciones de escritura fallan | Unblock Edit Tools desactivado | En BC página 8351 → activar **Unblock Edit Tools** y los permisos Allow Create/Modify/Delete en cada API page |
| Más de 70 APIs necesarias | Límite de herramientas de Copilot Studio | Activar **Dynamic Tool Mode** ON para discovery dinámico sin límite |
| Herramientas aparecen pero el agente no las usa | modelDescription insuficiente | Editar el modelDescription de la acción MCP con `/copilot-studio:configure-mcp-action` para incluir descripción detallada de las operaciones disponibles |
| El agente usa la API equivocada | Descripciones ambiguas entre APIs | Personalizar las API page descriptions en BC o editar modelDescription en la acción MCP |

## After Completion

After the setup is complete:

1. Offer to configure the MCP action YAML:
   > ¿Quieres personalizar la configuración de la acción MCP? Puedo modificar el modelDescription, los inputs, o el modo de conexión con `/copilot-studio:configure-mcp-action`.

2. Offer to update agent instructions:
   > ¿Quieres actualizar las instrucciones del agente para incluir guías sobre cómo usar las herramientas de BC? Puedo hacerlo con `/copilot-studio:edit-agent`.

3. Update memory:
   > Voy a actualizar `circe-memory.md` con los detalles de la configuración MCP.

## References

- [Configure Business Central MCP Server](https://learn.microsoft.com/en-us/dynamics365/business-central/dev-itpro/ai/configure-mcp-server)
- [Create agents in Copilot Studio that connect to Business Central](https://learn.microsoft.com/en-us/dynamics365/business-central/dev-itpro/ai/create-agent-in-copilot-studio)
