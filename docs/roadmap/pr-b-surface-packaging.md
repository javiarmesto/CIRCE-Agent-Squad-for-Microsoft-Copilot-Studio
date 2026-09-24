# [PR-B] Empaquetado CIRCE para VS Code y Claude Code

**Estado: especificación preparada; implementación pendiente.** Esta PR en borrador incorpora únicamente el documento de trabajo. No acredita correcciones aplicadas ni pruebas funcionales.

Por petición de Javier se abren los cuatro borradores ahora desde la misma base; la ejecución e integración mantienen el orden A → B → C → D. Se reutilizarán esta rama y esta PR, actualizando su base después de integrar su predecesora. No crear otra PR al ejecutar el prompt de la especificación.

Base de preparación: `4f0e0538616336d7641a35350691ac446fc72160`.

## Dependencia

Implementar después de integrar https://github.com/javiarmesto/CIRCE-Agent-Squad-for-Microsoft-Copilot-Studio/pull/2. Actualizar esta rama desde main antes de iniciar la implementación.

## Reglas comunes de ejecución

1. Leer las instrucciones aplicables del repositorio y comprobar el estado de trabajo. Refrescar la base y revisar cambios posteriores al commit evaluado; adaptar el plan si el código ya resuelve un punto.
2. Trabajar únicamente en CIRCE. No modificar ALDC, DELFOS ni otros repositorios.
3. Mantener una fuente canónica para el contenido común y adaptadores por superficie/harness. No prometer equivalencia funcional por el mero hecho de que los archivos existan.
4. Completar cada PR con los cambios de implementación y una descripción que explique problema, solución, verificación y límites. Abrirla con la identidad de Javier; verificar la cuenta autenticada antes de crearla. No atribuirle una PR creada por otra identidad.
5. No mezclar el merge con la preparación: no fusionar ni publicar releases dentro de estos paquetes sin una instrucción posterior que lo autorice.
6. No introducir datos del tenant, credenciales o transcripciones sin sanear en commits o evidencias públicas. No borrar cachés locales de autenticación que pertenezcan al usuario.
7. Registrar resultados como `PASS`, `FAIL`, `BLOCKED` o `NOT RUN`, indicando versión y superficie. Un log breve suficiente es preferible a transcripciones extensas. Una comprobación estática no acredita ejecución.
8. No exigir una aprobación por cada lectura o edición local. Para operaciones sobre un tenant, comprobar el alcance ya autorizado y el destino concreto. Distinguir creación en Dataverse, push y publicación; no considerarlos todos operaciones locales.


## PR-B — Distribución real para VS Code y Claude Code

### Problema

CIRCE contiene agentes y skills en `.github/`, un `CLAUDE.md` y hooks, pero no una distribución de plugin nativo con manifiestos. La presencia de instrucciones no garantiza descubrimiento, delegación o ejecución de hooks en cada cliente.

### Alcance

- Inventariar los cinco roles actuales: Author, Manage, Test, Troubleshoot y Conductor; conservar sus responsabilidades salvo una incompatibilidad demostrada.
- Elegir y documentar la fuente canónica del núcleo. Mantener CIRCE (convenciones, memoria, decisiones y BC) separado de la infraestructura heredada de Microsoft.
- Añadir manifiestos de plugin y catálogo instalable para Claude Code, con nombres propios de CIRCE que eviten colisiones con `copilot-studio` y `mcs-assistant`.
- Generar o mantener adaptadores explícitos para VS Code/GitHub Copilot y Claude Code: descubrimiento de agentes, skills, hooks y rutas a scripts.
- Revisar las ubicaciones y esquemas de hooks según la documentación vigente del cliente. No copiar `.github/hooks/hooks.json` a otra superficie sin adaptación.
- Resolver rutas a recursos desde la raíz instalada del plugin, no desde la raíz del proyecto de negocio ni desde rutas del equipo del autor.
- Registrar commit/versionado de las dependencias de Microsoft y qué contenido se importa o modifica. No sustituir en bloque el upstream encima de las aportaciones de CIRCE.
- Añadir instrucciones separadas de instalación, actualización, desactivación y desinstalación. La instalación no debe sobrescribir instrucciones del proyecto consumidor.
- Definir detección de instalaciones duplicadas y convivencia con plugins de Microsoft; evitar que dos hooks impongan flujos incompatibles.

### Archivos

Revisar `.github/agents/`, `.github/skills/`, `.github/hooks/hooks.json`, `.github/copilot-instructions.md`, `CLAUDE.md`, `CONTRIBUTING.md` y `docs/setup-guide.md`.

Propuestos, sujetos a la estructura nativa verificada: `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, carpetas nativas `agents/`, `skills/`, `hooks/`, un generador de adaptadores y `docs/compatibility.md`. No crear dos fuentes manuales divergentes para el mismo contenido.

### Criterios de aceptación

- [ ] Los manifiestos se validan y todas las rutas de recursos existen.
- [ ] Instalar desde un clon limpio no altera archivos ajenos a CIRCE.
- [ ] VS Code descubre los roles y skills previstos y carga el hook adaptado.
- [ ] Claude Code descubre el plugin, roles y skills, y carga sus hooks.
- [ ] Una petición sencilla llega al especialista correcto en cada cliente probado.
- [ ] El Conductor puede entregar una tarea y recibir el resultado sin bucles.
- [ ] Actualizar o desinstalar no borra artefactos del agente de negocio.
- [ ] Cada resultado distingue validación de archivos de ejecución en el cliente.

El empaquetado puede revisarse sin tenant. Si falta un cliente, dejar su prueba como pendiente y mantener la PR en borrador hasta disponer de evidencia suficiente. No anunciar soporte validado por la simple validación del manifiesto.

### Prompt de ejecución

> Ejecuta únicamente PR-B del plan CIRCE, partiendo de main con PR-A integrada. Crea feat/circe-surface-packaging. Implementa una distribución instalable para Claude Code y un adaptador verificado para VS Code/GitHub Copilot, con núcleo común y nombres CIRCE. Conserva la semántica del harness estándar. Resuelve recursos desde la instalación, adapta hooks y evita colisiones con los plugins Microsoft. Documenta instalación y compatibilidad. Ejecuta comprobaciones de empaquetado y descubrimiento disponibles; proporciona una prueba breve para cualquier cliente ausente. Abre la PR con la identidad verificada de Javier. No hagas merge, no despliegues al tenant y no implementes PR-C.


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

