# [PR-D] Escenarios de aceptación y quickstarts

**Estado: especificación preparada; implementación pendiente.** Esta PR en borrador incorpora únicamente el documento de trabajo. No acredita correcciones aplicadas ni pruebas funcionales.

Por petición de Javier se abren los cuatro borradores ahora desde la misma base; la ejecución e integración mantienen el orden A → B → C → D. Se reutilizarán esta rama y esta PR, actualizando su base después de integrar su predecesora. No crear otra PR al ejecutar el prompt de la especificación.

Base de preparación: `4f0e0538616336d7641a35350691ac446fc72160`.

## Dependencia

Implementar después de integrar https://github.com/javiarmesto/CIRCE-Agent-Squad-for-Microsoft-Copilot-Studio/pull/4. Actualizar esta rama desde main antes de iniciar la implementación.

## Reglas comunes de ejecución

1. Leer las instrucciones aplicables del repositorio y comprobar el estado de trabajo. Refrescar la base y revisar cambios posteriores al commit evaluado; adaptar el plan si el código ya resuelve un punto.
2. Trabajar únicamente en CIRCE. No modificar ALDC, DELFOS ni otros repositorios.
3. Mantener una fuente canónica para el contenido común y adaptadores por superficie/harness. No prometer equivalencia funcional por el mero hecho de que los archivos existan.
4. Completar cada PR con los cambios de implementación y una descripción que explique problema, solución, verificación y límites. Abrirla con la identidad de Javier; verificar la cuenta autenticada antes de crearla. No atribuirle una PR creada por otra identidad.
5. No mezclar el merge con la preparación: no fusionar ni publicar releases dentro de estos paquetes sin una instrucción posterior que lo autorice.
6. No introducir datos del tenant, credenciales o transcripciones sin sanear en commits o evidencias públicas. No borrar cachés locales de autenticación que pertenezcan al usuario.
7. Registrar resultados como `PASS`, `FAIL`, `BLOCKED` o `NOT RUN`, indicando versión y superficie. Un log breve suficiente es preferible a transcripciones extensas. Una comprobación estática no acredita ejecución.
8. No exigir una aprobación por cada lectura o edición local. Para operaciones sobre un tenant, comprobar el alcance ya autorizado y el destino concreto. Distinguir creación en Dataverse, push y publicación; no considerarlos todos operaciones locales.


## PR-D — Escenarios de aceptación y guía de uso

### Objetivo

Probar que CIRCE sirve para construir y mantener agentes desde ambas superficies. Preparar recorridos breves reutilizables, con pocas evidencias y resultados verificables.

### Matriz objetivo

| Cliente de desarrollo | Harness estándar | Harness GitHub Copilot |
|---|---|---|
| VS Code + GitHub Copilot | KB y BC | KB y BC, según disponibilidad |
| Claude Code | KB y BC | KB y BC, según disponibilidad |

Todas las celdas comienzan como NOT RUN. No transferir un PASS de una superficie a otra.

### Escenario 1: agente de conocimiento

1. Crear un proyecto o clonar un agente de prueba según la autorización disponible.
2. Añadir instrucciones y una fuente accesible con hechos concretos comprobables.
3. Editar un componente, validar con el mecanismo del perfil y revisar el diff.
4. Sincronizar y, si forma parte del alcance autorizado, publicar.
5. Probar una pregunta cuya respuesta existe en la fuente, una que no existe y una petición fuera de alcance.
6. Modificar una instrucción, repetir el ciclo y comprobar el nuevo comportamiento.

Resultado: respuesta fundamentada, reconocimiento de información ausente y ausencia de afirmaciones inventadas. No basta con «respondió».

### Escenario 2: Business Central de solo lectura

1. Obtener tenant, entorno, empresa, configuración MCP e identidad de conexión del entorno de prueba; no usar valores históricos por defecto.
2. Descubrir herramientas reales y seleccionar una consulta permitida.
3. Configurar la integración con conexiones existentes/autorizadas y validar su representación por harness.
4. Consultar un registro identificable y contrastarlo con el resultado real de la herramienta.
5. Probar identificación ambigua o registro inexistente; comprobar que solicita precisión o informa correctamente.
6. Confirmar que el escenario no requiere permisos de administración para el usuario final ni añade operaciones de escritura.

Resultado: la respuesta coincide con los datos obtenidos, respeta el ámbito de empresa y no presenta fallos de autenticación como ausencia de registros.

### Comprobaciones transversales mínimas

- Cambio remoto después de una edición local: se detecta el conflicto y no se sobrescribe silenciosamente.
- Autenticación o permiso insuficiente: se comunica el error concreto y no se atribuye al modelo.
- Actualización/reinstalación del plugin: se conserva el proyecto de negocio.
- Delegación: un especialista recibe la tarea y devuelve el resultado al Conductor en cada cliente.
- Publicación: la evidencia acredita éxito real, no solo salida de proceso satisfactoria.

### Entregables propuestos

`docs/validation/acceptance-plan.md`, `docs/validation/results-template.md`, `docs/quickstart-vscode.md`, `docs/quickstart-claude-code.md` y fixtures/tests que verifiquen riesgos concretos. Reutilizar los quickstarts existentes cuando sea posible.

Formato de evidencia por caso: identificador, fecha, commit CIRCE, versiones relevantes, cliente, harness, operación, resultado esperado, resultado observado, estado y limitación. Usar identificadores de entorno saneados en material público. La confirmación concreta de Javier sobre una prueba local se registra como evidencia aportada por el usuario, sin atribuirla a una ejecución del asistente.

### Criterios de aceptación

- [ ] Las guías son reproducibles desde una instalación limpia.
- [ ] La matriz contiene resultados observados y pendientes honestos.
- [ ] Existen resultados para KB y BC en cada combinación que se anuncie como soportada.
- [ ] Los límites de disponibilidad del nuevo harness están documentados.
- [ ] No se exige adjuntar transcripciones completas cuando basta una salida breve o confirmación concreta.
- [ ] README distingue compatibilidad implementada, validada y experimental.

### Prompt de ejecución

> Ejecuta PR-D del plan CIRCE sobre main con PR-C integrada, en test/circe-acceptance-scenarios. Prepara los escenarios KB y BC de solo lectura y los quickstarts para VS Code/GitHub Copilot y Claude Code. Ejecuta lo que permitan las herramientas y autorizaciones disponibles; deja prompts cortos para las pruebas locales de Javier. Usa PASS/FAIL/BLOCKED/NOT RUN y no extrapoles evidencias entre superficies o harnesses. No publiques ni crees recursos remotos sin un alcance autorizado y un destino concreto. Abre la PR con la identidad verificada de Javier. Si falta evidencia obligatoria, mantenla en borrador. No hagas merge ni publiques release.


## Fuentes técnicas de partida

Consultadas durante la evaluación del 24 de septiembre de 2026; contrastar su estado al ejecutar cada paquete.

- Extensión VS Code: https://learn.microsoft.com/en-us/microsoft-copilot-studio/visual-studio-code-extension-overview
- Plugin del harness estándar: https://github.com/microsoft/skills-for-copilot-studio
- Nuevo plugin: https://github.com/microsoft/copilot-studio-plugin
- Arquitectura del nuevo plugin: https://github.com/microsoft/copilot-studio-plugin/blob/main/agents/copilot-studio-architect.md
- PAC y ciclo local: https://learn.microsoft.com/en-us/power-platform/developer/cli/reference/copilot
- MCP integrado en PAC: https://learn.microsoft.com/en-us/power-platform/developer/howto/use-mcp
- Estructura de plugins Claude Code: https://code.claude.com/docs/en/plugins-reference
- Conexión MCP del agente estándar: https://learn.microsoft.com/en-us/microsoft-copilot-studio/mcp-add-existing-server-to-agent

Se priorizan herramientas Microsoft para gestión y autoría. El servidor comunitario jgt87/copilot-studio-mcp queda fuera de las dependencias iniciales; puede evaluarse después si resuelve una necesidad concreta no cubierta.

