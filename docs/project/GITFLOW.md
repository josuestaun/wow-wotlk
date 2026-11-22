# GitFlow ligero

Flujo pensado para un proyecto personal con releases ocasionales. Se mantiene simple: dos ramas
permanentes y ramas cortas de trabajo.

## Ramas permanentes

| Rama | Propósito | Recibe cambios de | Se integra en |
| --- | --- | --- | --- |
| `master` | Estado estable, releases y hotfixes | `develop` (release), `hotfix/*` | — |
| `develop` | Integración y trabajo diario | `feature/*`, `fix/*`, `master` | `master` (release) |

## Ramas de trabajo

| Rama | Nace de | Se integra en | Uso |
| --- | --- | --- | --- |
| `feature/<slug>` | `develop` | `develop` vía PR | Nueva funcionalidad o documentación |
| `fix/<slug>` | `develop` | `develop` vía PR | Corrección no urgente |
| `hotfix/<slug>` | `master` | `master` y `develop` | Corrección urgente de una release |

Nombres en minúsculas y kebab-case:

- `feature/project-docs`
- `feature/module-custom-npcs`
- `fix/quest-reward-1234`
- `hotfix/auth-crash`

## Ciclo de una feature o fix

```powershell
git switch develop
git pull --ff-only
git switch -c feature/mi-cambio
# trabajar y commitear
git push -u origin feature/mi-cambio
```

En GitHub, abre un PR contra `develop`. El PR debe:

- Tener un título en Conventional Commits (será el mensaje del squash).
- Describir qué cambia, por qué, cómo se probó y qué riesgos tiene.
- Pasar CI y no tener conflictos.
- Actualizar documentación y ADR si aplica.
- No incluir secretos, binarios, datos locales ni assets del cliente.

Al aprobar, se hace **squash merge** y se elimina la rama remota. No se hace merge local directo a
`develop` salvo la excepción de bootstrap descrita abajo.

## Excepción de bootstrap

El commit inicial de fundación documental puede ir directamente a `develop` porque todavía no hay
una base estable desde la que abrir un PR. A partir de ese commit, todo cambio funcional o de
documentación relevante pasa por rama y PR.

## Release

No hay rama `release/*`. El flujo es:

1. Verificar que `develop` está estable y probado.
2. Abrir PR de `develop` a `master`.
3. Hacer merge (merge commit o squash, según el historial que se quiera conservar).
4. Etiquetar `master` con `vX.Y.Z`.
5. Crear la release en GitHub con notas.
6. Volver a integrar `master` en `develop` si el merge no fue fast-forward.

```powershell
git switch master
git pull --ff-only
git tag -a v0.1.0 -m "v0.1.0"
git push origin master --tags
```

No se hace push sin autorización del usuario.

## Hotfix

Un hotfix corrige un problema en `master` sin esperar al siguiente ciclo:

```powershell
git switch master
git pull --ff-only
git switch -c hotfix/descripcion-corta
# corregir y commitear
git push -u origin hotfix/descripcion-corta
```

1. PR de `hotfix/*` a `master`.
2. Merge y tag de parche (`v0.1.1`).
3. PR de `master` (o del hotfix) a `develop` para no perder la corrección.

## Sincronización con AzerothCore

```powershell
git fetch upstream
git switch master
git merge upstream/master
git switch develop
git merge master
```

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

## Configuración recomendada en GitHub

- Proteger `master` y `develop`: prohibir force push y borrado.
- Requerir PR antes de mergear en ambas.
- Requerir CI en verde.
- Requerir conversaciones resueltas.
- Activar secret scanning y Dependabot alerts si el repositorio es público.
- Limitar quién puede hacer push directo.

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
