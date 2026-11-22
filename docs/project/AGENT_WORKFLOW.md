# Flujo de trabajo del agente

Este documento define cómo debe trabajar un agente de IA en este repositorio. Las reglas de
AzerothCore de `AGENTS.md` y `.agents/docs/` siguen siendo obligatorias.

## Principios

1. **Inspeccionar antes de actuar**: lee `AGENTS.md`, el documento de `docs/project/` que
   corresponda y el código real antes de proponer cambios.
2. **Cambios pequeños y reversibles**: una tarea, una rama, un objetivo verificable.
3. **No sorprender con operaciones destructivas**: el ciclo Git rutinario se hace de forma autónoma;
   force push, resets destructivos, borrado de trabajo no fusionado y cambios en `master` requieren
   autorización.
4. **Verificar**: no declares una tarea terminada sin comprobar el resultado.
5. **Documentar**: las decisiones nuevas van a `DECISIONS.md`; los cambios de alcance, a
   `ROADMAP.md`.

## Autonomía

### Git: permitido sin pedir permiso

- Consultar estado, ramas, remotos, diferencias e historial.
- Hacer `fetch` y `pull` no destructivo.
- Crear y cambiar ramas `feature/*`, `fix/*` o `hotfix/*`.
- Crear commits siguiendo `COMMITS.md`.
- Hacer push de ramas de trabajo a `origin`.
- Abrir o actualizar PRs cuando `gh` u otra herramienta esté disponible y autenticada.
- Fusionar `feature/*` y `fix/*` en `develop` cuando el cambio esté validado.
- Resolver conflictos simples sin riesgo de perder trabajo.
- Eliminar ramas temporales ya fusionadas.
- Sincronizar `develop` con `origin` después de integrar.

### Git: requiere autorización explícita

- Force push o reescritura de historia publicada.
- `reset` destructivo o descarte de cambios no guardados.
- Borrado de ramas con trabajo no fusionado.
- Cualquier modificación o merge a `master`.
- Borrado o reinicialización de bases de datos, personajes o datos persistentes.
- Sobrescritura de recursos originales sin respaldo.

### Otros límites del proyecto

- Instalar dependencias del servidor o del sistema.
- Configurar o compilar el servidor (salvo que la tarea lo pida explícitamente).
- Ejecutar migraciones o cambios sobre una base de datos con datos reales.
- Modificar archivos de configuración locales con credenciales.
- Cambiar la URL o los remotos configurados sin una petición explícita.
- Cambiar la lógica del juego o el comportamiento del núcleo.
- Tocar `data/sql/base/`, `data/sql/archive/` o `data/sql/updates/db_*/`.

### Prohibido

- Commits con secretos, credenciales, binarios, dumps, logs o datos del cliente.
- Bypass de hooks, permisos o reglas de CI.
- Editar SQL inmutable o introducir assets propietarios.
- Ejecutar `--auto` en OpenCode para saltar permisos.

## Ciclo de una tarea

1. **Contexto**: lee `AGENTS.md`, `docs/project/README.md` y el documento específico.
2. **Estado**: revisa `git status`, la rama actual y los cambios locales; preserva trabajo existente.
3. **Sincronización**: `git fetch origin` y `git pull --ff-only` en `develop`; no uses operaciones
   destructivas.
4. **Rama**: crea `feature/*` o `fix/*` desde `develop`; `hotfix/*` desde `master` solo con
   autorización.
5. **Implementación**: haz el cambio mínimo que resuelve el problema.
6. **Verificación**: ejecuta las comprobaciones disponibles; si no puedes, dilo.
7. **Revisión**: lee el diff completo y busca secretos, artefactos y cambios accidentales.
8. **Commit**: usa Conventional Commits.
9. **Publicación**: sube la rama de trabajo a `origin`.
10. **Integración**: abre PR si hay herramienta; si no, haz merge local validado en `develop`,
    push de `develop` y limpia la rama fusionada.
11. **Resumen**: indica rama, commits, push, PR, merge, comprobaciones y validación pendiente.

## Verificación

Según el tipo de cambio:

| Cambio | Comprobación mínima |
| --- | --- |
| Documentación | Enlaces válidos, formato, `git diff --check` |
| C++ | Compilar si se pidió; leer `.agents/docs/cpp-guidelines.md`; codestyle |
| SQL | Leer `.agents/docs/sql-guidelines.md`; validar sintaxis; ruta correcta |
| Módulo | Compilar con `MODULES=static` si se pidió; probar carga y desactivación |
| Gameplay | Build si se pidió; probar en el cliente y esperar confirmación antes de integrar |
| Configuración | Revisar que no haya secretos ni rutas absolutas versionadas |
| Git | `git status`, `git diff --cached`, `git diff --check` |

No inventes resultados de build o tests. Si no se puede compilar, indica exactamente qué falta. Una
compilación correcta no valida una funcionalidad que necesita prueba en el cliente.

## Commits y ramas

- Trabaja en `feature/*`, `fix/*` o `hotfix/*`; no commitees directamente en `master`.
- La excepción de bootstrap está descrita en `GITFLOW.md`.
- Un commit por cambio lógico.
- Título en Conventional Commits; el cuerpo explica el porqué.
- No incluyas archivos generados ni cambios de formato ajenos.
- Acredita el código portado de otro proyecto.
- Puedes hacer push de la rama y abrir el PR sin autorización.
- Puedes integrar en `develop` cuando el cambio esté validado; no necesitas autorización para ese
  merge.
- Integrar en `master`, crear tags o publicar releases requiere autorización.

## Pull Requests

Un PR debe incluir:

- Título Conventional Commits.
- Descripción del problema, la solución y el alcance.
- Evidencia de prueba.
- Impacto en base de datos, núcleo y actualizaciones.
- Checklist de `GITFLOW.md`.
- Sin secretos ni artefactos.

Si `gh` no está instalado o no hay autenticación, no inventes un PR: integra localmente el cambio
validado en `develop`, push de `develop` y explica que no se pudo abrir el PR. Un PR contra `master`
requiere autorización.

## Cuándo detenerse y preguntar

- La tarea es ambigua y hay varias interpretaciones con impacto distinto.
- El cambio requiere modificar `master`, el núcleo o un directorio inmutable.
- Hay que hacer force push, un reset destructivo o borrar una rama no fusionada.
- Hay que borrar o migrar datos.
- Hay conflictos de merge no triviales.
- Se detecta un secreto o un archivo propietario.
- El build o los tests fallan por causas que no puedes resolver.
- La validación en el cliente está pendiente y el cambio aún no debe integrarse.
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

1. Rama y commits creados.
2. Archivos creados o modificados.
3. Comprobaciones realizadas y su resultado.
4. Push, PR y merge realizados, o el motivo por el que no se hicieron.
5. Validación pendiente en el cliente, si aplica.
6. Decisiones tomadas y su justificación.
7. Riesgos o limitaciones.
8. Siguientes pasos concretos.

No incluyas relleno ni explicaciones largas que no aporten.
