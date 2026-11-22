# GitFlow ligero

Flujo pensado para un proyecto personal con releases ocasionales. Se mantiene simple: dos ramas
permanentes y ramas cortas de trabajo.

## Autonomía del agente

El agente gestiona el ciclo rutinario sin pedir confirmación individual:

- Consultar estado, ramas, remotos y diferencias.
- Hacer `fetch` y `pull` no destructivo.
- Crear y cambiar ramas `feature/*`, `fix/*` o `hotfix/*`.
- Crear commits con Conventional Commits.
- Hacer push de ramas de trabajo a `origin`.
- Abrir PRs cuando `gh` u otra herramienta esté disponible y autenticada.
- Fusionar `feature/*` y `fix/*` en `develop` cuando el cambio esté validado.
- Resolver conflictos simples sin riesgo de perder trabajo.
- Eliminar ramas temporales ya fusionadas.

Requiere autorización explícita:

- Force push o reescritura de historia publicada.
- `reset` destructivo o descarte de cambios no guardados.
- Borrado de ramas con trabajo no fusionado.
- Cualquier modificación o merge a `master`.
- Borrado o reinicialización de bases de datos, personajes o datos persistentes.
- Sobrescritura de recursos originales sin respaldo.

## Ramas permanentes

| Rama | Propósito | Recibe cambios de | Se integra en |
| --- | --- | --- | --- |
| `master` | Estado estable, releases y hotfixes | `develop` (release), `hotfix/*` | — |
| `develop` | Integración y trabajo diario | `feature/*`, `fix/*`, `master` | `master` (release) |

## Ramas de trabajo

| Rama | Nace de | Se integra en | Uso |
| --- | --- | --- | --- |
| `feature/<slug>` | `develop` | `develop` vía PR o merge validado | Nueva funcionalidad o documentación |
| `fix/<slug>` | `develop` | `develop` vía PR o merge validado | Corrección no urgente |
| `hotfix/<slug>` | `master` | `master` y `develop` (con autorización) | Corrección urgente de una release |

Nombres en minúsculas y kebab-case:

- `feature/project-docs`
- `feature/module-custom-npcs`
- `fix/quest-reward-1234`
- `hotfix/auth-crash`

## Ciclo de una feature o fix

1. Sincronizar `develop` sin operaciones destructivas.
2. Crear la rama de trabajo.
3. Implementar y verificar.
4. Commitear con Conventional Commits.
5. Subir la rama a `origin`.
6. Abrir PR contra `develop` si hay herramienta; si no, dejar constancia.
7. Integrar en `develop` solo cuando el cambio esté validado.
8. Sincronizar `develop` y limpiar la rama.

```powershell
git fetch origin
git switch develop
git pull --ff-only
git switch -c feature/mi-cambio
# trabajar, verificar y commitear
git push -u origin feature/mi-cambio
```

El PR, si existe, debe:

- Tener un título en Conventional Commits (será el mensaje del squash).
- Describir qué cambia, por qué, cómo se probó y qué riesgos tiene.
- Pasar CI y no tener conflictos.
- Actualizar documentación y ADR si aplica.
- No incluir secretos, binarios, datos locales ni assets del cliente.

Integración:

- Con `gh` disponible y autenticado: crear el PR, comprobar CI y hacer squash merge en `develop`.
- Sin herramienta de PR: hacer merge local validado (`--ff-only` si `develop` no divergió, o
  `--squash`) y push de `develop`; indicarlo en el resumen.
- Si el cambio necesita prueba en el cliente, esperar la confirmación del usuario antes de integrar.
- No se hace merge local directo a `develop` fuera de este flujo; la excepción de bootstrap está
  descrita abajo.

## Excepción de bootstrap

El commit inicial de fundación documental puede ir directamente a `develop` porque todavía no hay
una base estable desde la que abrir un PR. A partir de ese commit, todo cambio funcional o de
documentación relevante pasa por rama y PR, o por merge local validado cuando no haya herramienta de
PR.

## Validación antes de integrar

Una compilación correcta no valida una funcionalidad. Antes de integrar en `develop`:

- Ejecutar las comprobaciones disponibles (build, tests, lint, validación de SQL o documentación).
- Si el cambio afecta al gameplay y requiere el cliente de WoW, esperar a que el usuario confirme el
  resultado en el juego.
- Si no se puede verificar, dejarlo indicado y no presentar el cambio como validado.

## Limpieza de ramas

- Borrar ramas locales y remotas solo después de fusionarlas.
- `git branch -d` es seguro porque Git rechaza ramas no fusionadas; `git branch -D` requiere
  autorización.
- Para ramas remotas, borrar solo `feature/*` o `fix/*` ya fusionadas. Cualquier otro borrado remoto
  requiere autorización.

## Release

No hay rama `release/*`. El flujo es:

1. Verificar que `develop` está estable y probado.
2. Pedir autorización y abrir PR de `develop` a `master`.
3. Hacer merge (merge commit o squash, según el historial que se quiera conservar).
4. Etiquetar `master` con `vX.Y.Z`.
5. Crear la release en GitHub con notas.
6. Volver a integrar `master` en `develop` si el merge no fue fast-forward.

```powershell
# Solo con autorización explícita para tocar master y publicar la release
git switch master
git pull --ff-only
git tag -a v0.1.0 -m "v0.1.0"
git push origin master --tags
```

Los pushes de ramas de trabajo a `origin` están permitidos. Los pushes a `master`, la creación de
tags y las releases requieren autorización.

## Hotfix

Un hotfix corrige un problema en `master` sin esperar al siguiente ciclo. Requiere autorización
porque modifica `master`:

```powershell
git switch master
git pull --ff-only
git switch -c hotfix/descripcion-corta
# corregir y commitear
git push -u origin hotfix/descripcion-corta
```

1. PR de `hotfix/*` a `master` (con autorización).
2. Merge y tag de parche (`v0.1.1`).
3. PR de `master` (o del hotfix) a `develop` para no perder la corrección.

## Sincronización con AzerothCore

`upstream` apunta a AzerothCore. Actualizar `master` requiere autorización. Para preparar la
integración sin tocar `master`:

```powershell
git fetch upstream
git switch develop
git switch -c chore/upstream-sync
git merge upstream/master
# resolver conflictos, compilar y probar
```

Con autorización, integra el resultado en `master`, etiqueta si corresponde y vuelve a integrar
`master` en `develop`. El orden final se mantiene: `upstream/master` → `master` → `develop`.

Reglas:

- Resolver conflictos en una rama de integración, no directamente en `develop` si el conflicto es
  grande.
- Compilar y probar antes de continuar.
- No reescribir historia publicada.
- Si el merge de upstream trae cambios incompatibles con módulos propios, registrar la decisión en
  `DECISIONS.md`.

## Pull Requests

Plantilla: `pull_request_template.md` en la raíz. Cubre cambios, issues, fuente, pruebas y pasos de
verificación. Además, el PR debe incluir:

- [ ] Título Conventional Commits.
- [ ] Rama base correcta (`develop`, o `master` para hotfix/release).
- [ ] Descripción del problema y de la solución.
- [ ] Evidencia de prueba (build, log, captura o pasos manuales).
- [ ] Impacto en base de datos y plan de rollback.
- [ ] Impacto en el núcleo y justificación si lo hay.
- [ ] Documentación actualizada.
- [ ] Sin secretos, binarios, datos locales ni assets propietarios.

Para una persona sola, el PR sigue siendo útil: obliga a revisar el diff, deja registro y permite
volver atrás con facilidad.

Si `gh` no está instalado o no hay autenticación, el agente integra localmente y lo indica; no
inventa un PR. Un PR contra `master` requiere autorización.

## Configuración recomendada en GitHub

- Proteger `master`: requerir PR, prohibir force push y borrado.
- Proteger `develop`: prohibir force push y borrado; permitir los pushes del agente o exigir PR si
  la herramienta de PR está disponible.
- Requerir CI en verde cuando exista.
- Requerir conversaciones resueltas.
- Activar secret scanning y Dependabot alerts si el repositorio es público.

## Commits

- Un commit = un cambio lógico.
- No mezclar refactors, formato y funcionalidad en el mismo commit.
- No commitear trabajo a medias (`WIP`) salvo que sea un commit temporal en una rama que se va a
  squash.
- Seguir `COMMITS.md`.
- Acreditar código portado de otro proyecto con `Co-authored-by` o `--author`.

## Comandos útiles

```powershell
git status --short
git branch --show-current
git log --oneline --graph --decorate -20
git diff
git diff --cached
git diff --check
```

En PowerShell 5.1 no uses `&&`; usa `;` o ejecuta los comandos por separado.
