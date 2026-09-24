# [PR-A] Configuración portable y almacenamiento de credenciales

**Estado: especificación preparada; implementación pendiente.** Esta PR en borrador incorpora únicamente el documento de trabajo. No acredita correcciones aplicadas ni pruebas funcionales.

Por petición de Javier se abren los cuatro borradores ahora desde la misma base; la ejecución e integración mantienen el orden A → B → C → D. Se reutilizarán esta rama y esta PR, actualizando su base después de integrar su predecesora. No crear otra PR al ejecutar el prompt de la especificación.

Base de preparación: `4f0e0538616336d7641a35350691ac446fc72160`.

## Dependencia

Sin dependencia previa. Es la primera PR que debe implementarse.

## Reglas comunes de ejecución

1. Leer las instrucciones aplicables del repositorio y comprobar el estado de trabajo. Refrescar la base y revisar cambios posteriores al commit evaluado; adaptar el plan si el código ya resuelve un punto.
2. Trabajar únicamente en CIRCE. No modificar ALDC, DELFOS ni otros repositorios.
3. Mantener una fuente canónica para el contenido común y adaptadores por superficie/harness. No prometer equivalencia funcional por el mero hecho de que los archivos existan.
4. Completar cada PR con los cambios de implementación y una descripción que explique problema, solución, verificación y límites. Abrirla con la identidad de Javier; verificar la cuenta autenticada antes de crearla. No atribuirle una PR creada por otra identidad.
5. No mezclar el merge con la preparación: no fusionar ni publicar releases dentro de estos paquetes sin una instrucción posterior que lo autorice.
6. No introducir datos del tenant, credenciales o transcripciones sin sanear en commits o evidencias públicas. No borrar cachés locales de autenticación que pertenezcan al usuario.
7. Registrar resultados como `PASS`, `FAIL`, `BLOCKED` o `NOT RUN`, indicando versión y superficie. Un log breve suficiente es preferible a transcripciones extensas. Una comprobación estática no acredita ejecución.
8. No exigir una aprobación por cada lectura o edición local. Para operaciones sobre un tenant, comprobar el alcance ya autorizado y el destino concreto. Distinguir creación en Dataverse, push y publicación; no considerarlos todos operaciones locales.


## PR-A — Higiene, configuración portable y credenciales

### Problema

El árbol evaluado incluye una caché DPAPI versionada, reglas de autoaprobación con rutas y datos de un entorno concreto, y un almacén que sitúa archivos de caché dentro del proyecto. `.gitignore` es insuficiente. Las guías de clonación utilizan un nombre de repositorio distinto del actual.

### Alcance

- Retirar `.github/.token_cache_manage-agent.dpapi` del seguimiento de Git conservando, si existe en la máquina de trabajo, su copia local. No leer, descifrar ni mostrar el contenido.
- Actualizar `.gitignore` para cachés y configuración local; conservar las plantillas necesarias. Comprobar qué archivos ya están versionados, porque ignorarlos no los retira del historial ni del índice.
- Sustituir la configuración compartida de `.vscode/settings.json` por opciones portables. Retirar las autorizaciones ligadas a rutas/entornos personales y revisar la aprobación genérica de `ForEach-Object`.
- Cambiar `.github/scripts/src/credential-store.js` para que la ubicación de caché pertenezca al perfil de usuario, fuera del checkout. Definir un identificador estable de aplicación y una ruta por sistema operativo.
- Mantener los mecanismos de credenciales nativos. Evitar una degradación silenciosa a texto plano: si se conserva ese modo, que sea explícito, documentado y con permisos apropiados.
- Resolver las cachés heredadas mediante migración local controlada o una indicación de volver a autenticar. La instalación del framework no debe descifrar o exportar credenciales automáticamente.
- Recompilar los bundles afectados usando el proceso del repositorio. Verificar que el código fuente y el bundle usado por las skills contienen el mismo comportamiento.
- Corregir la URL de clonación en `README.md` y `docs/setup-guide.md`; revisar enlaces equivalentes en los quickstarts.
- Comprobar `INFORME-CRITICO.md`: figura ignorado pero también versionado. Tratar su exclusión de distribución según la intención documentada; no reescribir historial como parte de esta PR.

### Archivos existentes afectados

`.gitignore`, `.vscode/settings.json`, `.github/scripts/src/credential-store.js`, bundles que importen ese módulo, `README.md`, `docs/setup-guide.md` y quickstarts con enlaces obsoletos.

### Criterios de aceptación

- [ ] El diff no contiene valores de credenciales ni incorpora datos de entornos reales.
- [ ] La caché identificada deja de estar en el árbol versionado de la rama; se indica que el historial permanece.
- [ ] Las nuevas cachés se resuelven fuera del repositorio.
- [ ] Las rutas funcionan con espacios y caracteres acentuados.
- [ ] El bundle invocado por CIRCE refleja el cambio del almacén.
- [ ] Las guías clonan el repositorio correcto.
- [ ] Se declara qué comprobaciones de Windows/macOS/Linux se ejecutaron realmente.

Pruebas proporcionadas: resolución de rutas con directorios temporales, ausencia de escritura dentro del checkout y comportamiento ante fallo del almacén nativo. Usar datos ficticios. No hace falta conectarse al tenant para esta PR. La revisión de historial y una eventual revocación de sesiones son actuaciones separadas, según los hallazgos; no afirmar que la caché DPAPI demuestra exposición en claro.

### Prompt de ejecución

> Ejecuta únicamente PR-A del documento «CIRCE — Plan de ejecución de cuatro PRs» en javiarmesto/CIRCE-Agent-Squad-for-Microsoft-Copilot-Studio. Revisa primero main e instrucciones aplicables. Crea la rama fix/circe-portable-config desde la base actual y aplica higiene de configuración, almacenamiento de credenciales fuera del repositorio, bundles afectados y corrección de enlaces. Conserva las cachés locales del usuario; no las descifres ni las muestres. No reescribas historial ni accedas al tenant. Comprueba los criterios de aceptación con datos ficticios y abre una PR con la identidad autenticada de Javier, verificándola antes. Indica claramente cualquier prueba de plataforma no ejecutada. No hagas merge ni avances a PR-B.


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

