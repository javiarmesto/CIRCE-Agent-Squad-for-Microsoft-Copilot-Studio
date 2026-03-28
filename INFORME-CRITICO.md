# CIRCE — Informe Crítico

> **Fecha**: Marzo 2026  
> **Repositorio analizado**: `javiarmesto/CIRCE---Agent-Architecture-Framework-for-Microsoft-Copilot-Studio`  
> **Base sobre la que se construye**: [`microsoft/skills-for-copilot-studio`](https://github.com/microsoft/skills-for-copilot-studio) (Giorgio Ughini / Power CAT team)

---

## 1. Puntos Fuertes

### 1.1 El BC Extension Pack es la pieza más valiosa del repositorio

Las skills de Business Central (bc-mcp-setup, bc-instructions-patterns, bc-action-templates, bc-topic-patterns, bc-agent-blueprints) son conocimiento operacional concreto y difícil de encontrar en otro sitio: qué API pages abrir, qué permisos asignar, qué campos exponer, cómo estructurar el modelDescription para que el orquestador enrute correctamente. Eso tiene valor real para alguien que construye agentes BC sin haberlo hecho antes.

### 1.2 Los patrones de gobernanza son genuinamente útiles

Cross-session memory (circe-memory.md), decision records ligeros, HITL gates antes de cada push, skills evidencing: estos patrones no son decoración. Son la diferencia entre un proyecto que puede mantener una tercera persona y uno que solo entiende su autor original. Para equipos que van a construir agentes BC en producción, esto tiene peso real.

### 1.3 Los scripts CLI son funcionales y no requieren instalación

Los 5 scripts (schema-lookup, manage-agent, chat-with-agent, directline-chat, connector-lookup) están pre-bundleados con esbuild. No hay npm install. Eso elimina una fuente frecuente de fricción en proyectos de este tipo.

### 1.4 El quickstart está bien articulado

Los tres prompts del quickstart (clonar → construir → revisar/desplegar/probar) siguen una lógica clara. Las checklists de validación por fase son el mejor indicador de madurez del diseño: el autor sabe qué puede fallar.

### 1.5 Arquitectura de agentes especializada

La separación Author / Manage / Test / Troubleshoot / Conductor es coherente. No hay un agente que haga todo, lo que evita el problema habitual de contextos demasiado grandes y respuestas genéricas.

---

## 2. Puntos Débiles

### 2.1 La implementación de referencia prometida no existe

El README declara:

> *"3. Medea — A reference implementation (collections agent) with its own README"*

No hay ningún directorio `Medea/` en el repositorio. Es la diferencia entre un framework que demuestra que funciona y uno que dice que podría funcionar. Sin Medea, CIRCE es documentación sin evidencia.

### 2.2 Rutas de ficheros incorrectas en el README

El README enlaza a `docs/quickstart-order-tracker.md`. El fichero está en la raíz (`quickstart-order-tracker.md`). El README también referencia `order-tracker-requirements.md` como si estuviera en el directorio del agente, pero está en la raíz del repositorio. Son errores menores pero indican que el README se escribió antes de que la estructura final estuviera decidida.

### 2.3 Tensión no resuelta entre Claude Code y VS Code

El plugin base (`microsoft/skills-for-copilot-studio`) fue diseñado originalmente para Claude Code / GitHub Copilot CLI. El CHANGELOG del 2026-03-21 documenta la migración de `skills:` a `tools:` para adaptarlo a VS Code. Pero los ficheros de skill siguen referenciando `${CLAUDE_SKILL_DIR}` (sintaxis de Claude Code, no de VS Code). Si el target principal es VS Code, esto es un bug que afecta a la navegación relativa de los skills que referencian ficheros hermanos.

**Ejemplo concreto**: `bc-mcp-setup/SKILL.md` contiene:
```
Read: ${CLAUDE_SKILL_DIR}/mcp-config-guide.md
```
Esta variable no existe en VS Code Copilot. El fichero `mcp-config-guide.md` nunca se cargará desde VS Code a menos que el usuario lo lea manualmente.

### 2.4 El repo tiene 4 días de vida y 0 estrellas

Esto no es un juicio de valor, es contexto necesario. El framework no tiene validación externa todavía. No sabemos si los prompts del quickstart producen resultados útiles o alucinaciones en la práctica, porque no hay registros de nadie que lo haya ejecutado end-to-end independientemente.

### 2.5 Barrera de entrada alta

Para ejecutar el quickstart desde cero necesitas:
- VS Code + extensión Copilot Studio
- Licencia GitHub Copilot (Pro/Business/Enterprise)
- Licencia Copilot Studio con Copilot Credits
- BC 27+ (2025 Wave 2) con MCP Server habilitado — en Preview, no GA
- Permission set MCP-ADMIN en BC
- Acceso a un entorno Power Platform

Son 5+ productos de Microsoft en capas. Cualquier problema en cualquier capa rompe el flujo completo. El quickstart de 30 minutos asume que todas las capas están operativas, lo cual es el escenario más optimista posible.

### 2.6 El clone URL del setup-guide es incorrecto

El `docs/setup-guide.md` indica:
```bash
git clone https://github.com/javiarmesto/circe.git
```
El repositorio se llama `CIRCE---Agent-Architecture-Framework-for-Microsoft-Copilot-Studio`. La URL corta `circe.git` no existe. Cualquier persona que intente seguir el setup-guide literalmente fallará en el primer paso.

### 2.7 Sin CI/CD, sin tests propios del framework

No hay workflows de GitHub Actions. No hay validación automática de los ficheros YAML de ejemplo ni de los skill files. El único mecanismo de calidad es la validación manual con schema-lookup.bundle.js.

### 2.8 "Delfos" no es un repositorio público

Se menciona en el issue como repositorio propio para comparar con CIRCE. No existe ningún repositorio `Delfos` ni similar en la cuenta `javiarmesto`. O es privado, o está planificado, o el nombre ha cambiado. No es posible hacer comparativa sin acceso a él.

---

## 3. Grado de Originalidad y Oportunidad

### 3.1 Qué aporta el plugin base

`microsoft/skills-for-copilot-studio` proporciona:
- Los 5 agentes con sus herramientas (Author, Manage, Test, Troubleshoot + Conductor)
- Los scripts CLI (schema-lookup, manage-agent, chat-with-agent, directline-chat, connector-lookup)
- La mayoría de las skills core (add-action, add-node, new-topic, validate, edit-agent, etc.)
- El esquema de validación YAML
- El sistema de templates

### 3.2 Qué aporta CIRCE sobre el base

Las contribuciones originales son:
- **BC Extension Pack completo** (bc-mcp-setup, bc-instructions-patterns, bc-action-templates, bc-topic-patterns, bc-agent-blueprints): conocimiento de dominio BC que no existe en el base plugin
- **circe-conventions.md** con reglas siempre activas vía hooks.json
- **Sistema de memoria (circe-memory.md)** y decision records como patrón de trabajo
- **Skill de pre-push review** con HITL gate explícito
- **Conductor agent** (el base plugin tiene 4 agentes; CIRCE añade el Conductor como orquestador de workflows multi-agente)
- **Blueprints** de agentes específicos (Collections, Sales Assistant, Customer Support)
- **Adaptación para VS Code** (el base plugin está principalmente orientado a Claude Code / Copilot CLI)

### 3.3 Valoración de originalidad

CIRCE no es original en su arquitectura base — reconoce explícitamente y correctamente que construye sobre trabajo ajeno. La originalidad real está en dos lugares:

1. **La capa de gobernanza** (memoria, evidencias, HITL, decision records) aplicada a agentes de Copilot Studio: ese patrón combinado no existe en el base plugin ni en ningún otro repositorio público conocido.
2. **El conocimiento específico de BC+MCP+Copilot Studio**: la combinación de las tres tecnologías en un framework coherente tiene escasa competencia pública directa.

### 3.4 Oportunidad

El timing es interesante: BC 27+ MCP está en Preview (desde octubre 2025), Copilot Studio tiene extensión VS Code GA desde finales de 2025, y el base plugin de Microsoft es "experimental research project, not an officially supported Microsoft product". Hay una ventana real. Pero es una ventana de corto plazo: cuando Microsoft consolide su propia toolchain oficial, los frameworks de terceros como este tendrán que pivotar o volverse innecesarios.

---

## 4. Posicionamiento vs. Repositorios Propios

### 4.1 ALDC (AL Development Collection for GitHub Copilot)

| Dimensión | ALDC | CIRCE |
|-----------|------|-------|
| **Objetivo** | Desarrollo de extensiones AL para BC | Construcción de agentes Copilot Studio |
| **Target** | Desarrollador AL | Creador de agentes / consultor |
| **Estado** | v3.2.0, 45 estrellas, extensión en VS Code Marketplace, workshops | 4 días, 0 estrellas, sin extensión |
| **Madurez** | Alta — tiene versionado semántico, CHANGELOG detallado, validador propio | Baja — prometedora pero sin validación real aún |
| **Instalación** | `code --install-extension JavierArmesto.aldc-al-development-collection` | git clone + configuración manual |
| **Relación** | — | Complementario: ALDC construye el código BC, CIRCE construye el agente que lo consume |

ALDC y CIRCE **no compiten, se complementan**. El flujo natural sería: ALDC desarrolla las API pages BC personalizadas → CIRCE expone esas APIs a través de MCP en un agente Copilot Studio. Hay una narrativa de ecosystem aquí que no está explicitada todavía.

### 4.2 Delfos

No se encuentra ningún repositorio público con ese nombre en la cuenta `javiarmesto`. Si existe como repo privado o está planificado, sería el tercer pilar del ecosystem:

```
ALDC          → Construye extensiones AL
CIRCE         → Construye agentes Copilot Studio
Delfos (?)    → ¿Arquitectura de datos / reporting / BI? ¿Workshop combinado?
```

Sin acceso al repo no es posible hacer una comparativa.

---

## 5. Viabilidad del Quickstart

### 5.1 El diseño del quickstart es sólido

La estructura en 3 fases con checklists de validación por fase, los prompts concretos para copiar/pegar, y la tabla de troubleshooting son buenas prácticas. El orden (clonar → construir → revisar/desplegar/probar) es el flujo correcto.

### 5.2 Pero tiene dependencias frágiles

**Dependencia 1 — BC 27+ con MCP en Preview**: El quickstart requiere una feature que según la propia documentación es Public Preview ("Las funcionalidades pueden cambiar antes de General Availability"). Si Microsoft cambia el comportamiento del MCP Server BC, el quickstart queda roto.

**Dependencia 2 — El agente en blanco debe crearse antes**: El quickstart exige crear un agente en Copilot Studio UI antes de empezar. Si falla la autenticación de la extensión VS Code, si los tokens expiran, o si el entorno Power Platform tiene restricciones, el flujo se rompe antes de escribir una línea de YAML.

**Dependencia 3 — Medea no existe**: El README referencia Medea como "reference implementation" pero no está en el repo. El quickstart debería ser la referencia operativa; en su ausencia, un usuario no puede verificar si lo que genera CIRCE se parece a lo que debería generar.

**Dependencia 4 — La URL de clone es incorrecta**: `github.com/javiarmesto/circe.git` no existe. El repositorio tiene un nombre largo.

**Dependencia 5 — ${CLAUDE_SKILL_DIR} en VS Code**: Varios skills cargan ficheros complementarios usando esta variable. En VS Code no funciona. El agente IA no cargará esos ficheros automáticamente, lo que puede producir resultados incompletos.

### 5.3 Estimación realista de tiempo

"~30 min (prerequisites already installed)" es correcto si:
- Tienes BC 27+ con MCP habilitado y configurado
- Tienes Copilot Studio con licencia activa
- La extensión VS Code de Copilot Studio está autenticada
- Ya sabes cómo funciona YAML-first development con estas herramientas

Para alguien que no cumple alguna de estas condiciones, el quickstart es un proceso de 1-2 días que incluye solicitar licencias, configurar entornos, y depurar autenticación.

---

## 6. Roadmap Propuesto

Ordenado por impacto/esfuerzo.

### Fase 0 — Correcciones inmediatas (1-2 días)

| # | Problema | Acción |
|---|----------|--------|
| 1 | URL incorrecta en setup-guide.md | Corregir `github.com/javiarmesto/circe.git` con la URL real |
| 2 | Ruta incorrecta del quickstart en README | Mover a `docs/` o corregir el enlace |
| 3 | `${CLAUDE_SKILL_DIR}` en skills para VS Code | Reemplazar por rutas relativas explícitas o instrucción de leer manualmente |
| 4 | Medea/ prometida pero inexistente | Quitar la referencia del README hasta que exista, o crear la estructura mínima |

### Fase 1 — Validación real del framework (2-4 semanas)

| # | Acción |
|---|--------|
| 1 | **Ejecutar el quickstart completo end-to-end** y documentar exactamente dónde falla |
| 2 | **Crear Medea**: un directorio con el agente Collections real, con todos sus YAMLs, memory, decision records y evidencias como resultado de un run real del quickstart |
| 3 | Añadir GitHub Actions para validar YAML de Medea con schema-lookup en cada PR |

### Fase 2 — Reducir barreras de entrada (1-2 meses)

| # | Acción |
|---|--------|
| 1 | **Crear un quickstart alternativo sin BC**: agente de Knowledge Base sobre SharePoint, que valida el framework sin requerir BC 27+ MCP Preview |
| 2 | **Resolver la dualidad Claude Code / VS Code**: documentar explícitamente qué funciona en cada entorno y qué no; o comprometerse con uno solo |
| 3 | Crear un script de validación del entorno (`node .github/scripts/check-prereqs.js`) que verifique antes de empezar |

### Fase 3 — Madurez del ecosistema (2-4 meses)

| # | Acción |
|---|--------|
| 1 | **"Delfos" como workshop hands-on** del ecosystem ALDC+CIRCE: un repo equivalente a ALDC-Workshop-BC-Winter-Fest pero para CIRCE, con un agente funcional de cobros construido step-by-step |
| 2 | **Publicar VS Code extension** equivalente a lo que ALDC tiene para ALDC, que instale el framework en cualquier workspace de Copilot Studio con un comando |
| 3 | **Explicitar la narrativa ALDC → CIRCE**: cómo pasar de una extensión AL construida con ALDC a un agente Copilot Studio construido con CIRCE que la consume vía MCP |
| 4 | Añadir versioning semántico y CHANGELOG estructurado (seguir el modelo de ALDC) |

### Fase 4 — Posicionamiento a medio plazo (4-6 meses)

| # | Acción |
|---|--------|
| 1 | Monitorizar si Microsoft publica toolchain oficial para VS Code + Copilot Studio — si ocurre, redefinir el scope de CIRCE a la capa BC/governance que Microsoft no cubre |
| 2 | Construir el skill `bc-custom-apis` para conectar con extensiones AL desarrolladas con ALDC, cerrando el ciclo del ecosystem |
| 3 | Contribuir al repo base `microsoft/skills-for-copilot-studio` con las mejoras de VS Code compatibility (eliminar `${CLAUDE_SKILL_DIR}`, etc.) |

---

## Resumen ejecutivo

CIRCE tiene valor real en su BC Extension Pack y en sus patrones de gobernanza. La arquitectura es coherente y la intención es legítima. Pero en su estado actual (4 días de vida, Medea inexistente, URLs rotas, dualidad Claude/VS Code no resuelta, 0 validación externa) es un framework que promete más de lo que demuestra.

La comparación honesta con ALDC es desfavorable en madurez, pero tiene sentido si se lee como el primer release de un nuevo producto complementario, no como competidor. La oportunidad existe, la ventana es corta, y la Fase 0 del roadmap se puede ejecutar en un fin de semana.

La pregunta no resuelta más importante: ¿cuántos usuarios reales van a construir agentes Copilot Studio para BC en VS Code vs. en el portal web directamente? Si esa audiencia no existe en volumen suficiente, el framework resuelve un problema de alta sofisticación técnica para una audiencia muy pequeña.
