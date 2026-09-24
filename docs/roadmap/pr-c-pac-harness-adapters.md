# [PR-C] PAC CLI, MCP y autoría por harness

**Estado: especificación preparada; implementación pendiente.** Esta PR en borrador incorpora únicamente el documento de trabajo. No acredita correcciones aplicadas ni pruebas funcionales.

Por petición de Javier se abren los cuatro borradores ahora desde la misma base; la ejecución e integración mantienen el orden A → B → C → D. Se reutilizarán esta rama y esta PR, actualizando su base después de integrar su predecesora. No crear otra PR al ejecutar el prompt de la especificación.

Base de preparación: `4f0e0538616336d7641a35350691ac446fc72160`.

## Dependencia

Implementar después de integrar https://github.com/javiarmesto/CIRCE-Agent-Squad-for-Microsoft-Copilot-Studio/pull/3. Actualizar esta rama desde main antes de iniciar la implementación.

## Reglas comunes de ejecución

1. Leer las instrucciones aplicables del repositorio y comprobar el estado de trabajo. Refrescar la base y revisar cambios posteriores al commit evaluado; adaptar el plan si el código ya resuelve un punto.
2. Trabajar únicamente en CIRCE. No modificar ALDC, DELFOS ni otros repositorios.
3. Mantener una fuente canónica para el contenido común y adaptadores por superficie/harness. No prometer equivalencia funcional por el mero hecho de que los archivos existan.
4. Completar cada PR con los cambios de implementación y una descripción que explique problema, solución, verificación y límites. Abrirla con la identidad de Javier; verificar la cuenta autenticada antes de crearla. No atribuirle una PR creada por otra identidad.
5. No mezclar el merge con la preparación: no fusionar ni publicar releases dentro de estos paquetes sin una instrucción posterior que lo autorice.
6. No introducir datos del tenant, credenciales o transcripciones sin sanear en commits o evidencias públicas. No borrar cachés locales de autenticación que pertenezcan al usuario.
7. Registrar resultados como `PASS`, `FAIL`, `BLOCKED` o `NOT RUN`, indicando versión y superficie. Un log breve suficiente es preferible a transcripciones extensas. Una comprobación estática no acredita ejecución.
8. No exigir una aprobación por cada lectura o edición local. Para operaciones sobre un tenant, comprobar el alcance ya autorizado y el destino concreto. Distinguir creación en Dataverse, push y publicación; no considerarlos todos operaciones locales.


## PR-C — PAC CLI, MCP y autoría por harness

### Problema

La gestión depende de scripts y supuestos anteriores; las plantillas estándar no son intercambiables con el modelo del nuevo harness. Además, la configuración MCP genérica contiene detalles específicos de Business Central.

### Subpaso C1: herramientas de gestión

- Añadir una comprobación inicial de versiones y capacidades: PAC, runtime, cliente, plugins, proyecto, autenticación y destino. Un dato desconocido debe permanecer desconocido.
- Incorporar PAC para las operaciones disponibles de init/clone/pull/push/pack/publish. Verificar opciones con la versión instalada.
- Distinguir `init` local de `init --environment`, que crea recursos en Dataverse. No tratar bootstrap como un simple scaffold local.
- Mantener separado push de publish; detectar errores de negocio en la salida, sin basarse únicamente en el código de salida.
- Inventariar capacidades de `manage-agent.bundle.js` antes de reemplazarlo. No perder diff o validación por asumir que existe una equivalencia PAC.
- Ofrecer MCP oficial de PAC como integración opcional, con configuraciones propias de VS Code y Claude Code. No inventar nombres de herramientas: descubrir el catálogo y registrar lo realmente disponible.
- Mantener una ruta PAC directa funcional cuando MCP no esté configurado.

### Subpaso C2: perfiles de harness

- Seleccionar perfil mediante metadatos y estructura verificables; no inferirlo del modelo de IA elegido por el desarrollador. Si hay contradicción, detener la generación afectada y pedir el dato necesario.
- Perfil estándar: conservar topics, variables, acciones y su validación correspondiente; revisar cambios útiles del upstream actual.
- Perfil GitHub Copilot: utilizar el plugin Microsoft y sus referencias compatibles; contemplar `settings.mcs.yml`, `behaviors/`, `capabilities/` e `infrastructure/` según lo que realmente genere PAC.
- Evitar usar el esquema estándar para declarar válido un proyecto del nuevo harness.
- Distinguir skills de autoría de CIRCE de skills que se empaquetan como comportamiento del agente publicado.
- Adaptar el BC Extension Pack: separar MCP genérico de inputs, descubrimiento y permisos de BC. No inventar operationId, connection references, herramientas o esquemas.
- Añadir una evaluación de migración que identifique equivalencias y pérdidas. Preservar el origen; cualquier migración efectiva se hará sobre un destino separado y con alcance autorizado.
- Reubicar las reglas deterministas críticas en herramientas/flujos/código cuando el harness de destino no permita topics. No declarar que una instrucción proporciona la misma protección.

### Archivos

Agentes Manage, Author, Conductor y Troubleshoot; skills de gestión, scaffold, validación, descubrimiento del proyecto, MCP y BC; nuevas referencias por harness y ejemplos de configuración MCP; matriz de compatibilidad y guías.

### Criterios de aceptación

- [ ] Distingue un proyecto estándar, uno del nuevo harness y uno ambiguo mediante fixtures basadas en formatos reales.
- [ ] Un proyecto desconocido no recibe plantillas de un harness elegido silenciosamente.
- [ ] PAC directo funciona para el recorrido disponible; MCP opcional registra su catálogo real.
- [ ] La validación utiliza el mecanismo apropiado al tipo de proyecto y declara sus límites.
- [ ] No se reutilizan IDs ni conexiones entre ejemplos o tenants.
- [ ] Se conserva y prueba al menos un escenario estándar antes de aceptar el nuevo perfil.
- [ ] Crear, modificar, sincronizar y publicar quedan diferenciados en permisos, mensajes y evidencias.

Primero comprobar C1 con fixtures y comandos sin efectos remotos. Después validar C2. Si la disponibilidad del nuevo harness bloquea las pruebas reales, mantener ese perfil experimental y dejar el estado explícito. No afirmar paridad entre perfiles ni activar una migración automática.

### Prompt de ejecución

> Ejecuta únicamente PR-C del plan CIRCE, con PR-B integrada, en feat/circe-pac-harness-adapters. Implementa primero C1 (PAC y MCP opcional) y después C2 (selección de harness y referencias compatibles). Contrasta la documentación y las versiones instaladas antes de generar comandos o YAML. Conserva el recorrido estándar, separa MCP genérico de BC y conserva validaciones sin sustitución equivalente. Usa fixtures sin datos reales para lo offline; no ejecutes operaciones en Dataverse hasta tener un destino y un alcance autorizados. Abre la PR con la identidad verificada de Javier, indicando pruebas realizadas, pendientes y limitaciones del nuevo perfil. No hagas merge ni comiences PR-D.


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

