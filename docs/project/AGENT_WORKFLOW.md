# Flujo de trabajo del agente

Este documento define cómo debe trabajar un agente de IA en este repositorio. Las reglas de
AzerothCore de `AGENTS.md` y `.agents/docs/` siguen siendo obligatorias.

## Principios

1. **Inspeccionar antes de actuar**: lee `AGENTS.md`, el documento de `docs/project/` que
   corresponda y el código real antes de proponer cambios.
2. **Cambios pequeños y reversibles**: una tarea, una rama, un objetivo verificable.
3. **No sorprender**: no hagas push, no borres datos, no cambies remotos ni reescribas historia sin
   autorización.
4. **Verificar**: no declares una tarea terminada sin comprobar el resultado.
5. **Documentar**: las decisiones nuevas van a `DECISIONS.md`; los cambios de alcance, a
   `ROADMAP.md`.

## Autonomía

### Permitido sin pedir permiso

- Leer, buscar y analizar archivos.
- Crear y editar documentación, código y configuración dentro de una rama de trabajo.
- Crear ramas locales `feature/*`, `fix/*` o `hotfix/*`.
- Ejecutar comandos de solo lectura: `git status`, `git diff`, `git log`, `git show`, `git grep`.
- Ejecutar comprobaciones estáticas y de formato que no instalen dependencias.
- Preparar commits locales siguiendo `COMMITS.md`.
- Corregir errores propios y volver a verificar.

### Requiere autorización explícita

- `git push`, abrir o actualizar un PR, cambiar remotos o etiquetas remotas.
- Instalar dependencias del servidor o del sistema.
- Configurar o compilar el servidor (salvo que la tarea lo pida explícitamente).
- Ejecutar migraciones o cambios sobre una base de datos con datos reales.
- Modificar archivos de configuración locales con credenciales.
- Cambiar la lógica del juego o el comportamiento del núcleo.
- Eliminar ramas, archivos o datos persistentes.
- Tocar `data/sql/base/`, `data/sql/archive/` o `data/sql/updates/db_*/`.

### Prohibido

- `git reset --hard`, `git clean -fd`, `git clean -fdx` o cualquier limpieza destructiva.
- Force push o reescritura de historia publicada.
- Commits con secretos, credenciales, binarios, dumps, logs o datos del cliente.
- Bypass de hooks, permisos o reglas de CI.
- Editar SQL inmutable o introducir assets propietarios.
- Ejecutar `--auto` en OpenCode para saltar permisos.

## Ciclo de una tarea

1. **Contexto**: lee `AGENTS.md`, `docs/project/README.md` y el documento específico.
2. **Estado**: revisa `git status`, la rama actual y los cambios locales.
3. **Plan**: si hay 3 o más pasos, crea una lista de tareas y actualízala.
4. **Implementación**: haz el cambio mínimo que resuelve el problema.
5. **Verificación**: ejecuta las comprobaciones disponibles; si no puedes, dilo.
6. **Revisión**: lee el diff completo y busca secretos, artefactos y cambios accidentales.
7. **Commit**: usa Conventional Commits y una rama de trabajo.
8. **Resumen**: indica archivos, comprobaciones, decisiones y siguientes pasos.

## Verificación

Según el tipo de cambio:

| Cambio | Comprobación mínima |
| --- | --- |
| Documentación | Enlaces válidos, formato, `git diff --check` |
| C++ | Compilar si se pidió; leer `.agents/docs/cpp-guidelines.md`; codestyle |
| SQL | Leer `.agents/docs/sql-guidelines.md`; validar sintaxis; ruta correcta |
| Módulo | Compilar con `MODULES=static` si se pidió; probar carga y desactivación |
| Configuración | Revisar que no haya secretos ni rutas absolutas versionadas |
| Git | `git status`, `git diff --cached`, `git diff --check` |

No inventes resultados de build o tests. Si no se puede compilar, indica exactamente qué falta.

## Commits y ramas

- Trabaja en `feature/*`, `fix/*` o `hotfix/*`; no commitees directamente en `master`.
- La excepción de bootstrap está descrita en `GITFLOW.md`.
- Un commit por cambio lógico.
- Título en Conventional Commits; el cuerpo explica el porqué.
- No incluyas archivos generados ni cambios de formato ajenos.
- Acredita el código portado de otro proyecto.

## Pull Requests

Un PR debe incluir:

- Título Conventional Commits.
- Descripción del problema, la solución y el alcance.
- Evidencia de prueba.
- Impacto en base de datos, núcleo y actualizaciones.
- Checklist de `GITFLOW.md`.
- Sin secretos ni artefactos.

Si no hay autorización para push, deja la rama y los commits preparados y explica cómo abrir el PR.

## Cuándo detenerse y preguntar

- La tarea es ambigua y hay varias interpretaciones con impacto distinto.
- El cambio requiere modificar el núcleo o un directorio inmutable.
- Hay que borrar o migrar datos.
- Hay conflictos de merge no triviales.
- Se detecta un secreto o un archivo propietario.
- El build o los tests fallan por causas que no puedes resolver.
- La solución propuesta rompe compatibilidad con el upstream.

## Windows y PowerShell

- Shell principal: PowerShell 5.1.
- No uses `&&`; usa `;` o comandos separados.
- Usa `opencode.cmd` si `opencode.ps1` está bloqueado por la política de ejecución.
- Cita rutas con espacios.
- No asumas `bash`, `make`, `nproc` ni herramientas Unix.
- Para módulos, Git Bash es opcional y solo para scripts oficiales.

## Gestión de contexto

- No leas archivos enormes completos si puedes buscar la sección relevante.
- No repitas documentación upstream; enlázala.
- Mantén las respuestas y los resúmenes concisos.
- Si el contexto se llena, resume el estado y continúa con una lista de tareas.

## Formato del resumen final

Al terminar una tarea, informa de:

1. Archivos creados o modificados.
2. Comprobaciones realizadas y su resultado.
3. Decisiones tomadas y su justificación.
4. Riesgos o limitaciones.
5. Siguientes pasos concretos.

No incluyas relleno ni explicaciones largas que no aporten.
