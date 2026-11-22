# Conventional Commits

Formato:

```text
<tipo>(<scope>): <asunto>

<cuerpo opcional>

<footer opcional>
```

El asunto se escribe en imperativo, con mayúscula inicial y sin punto final. Apunta a 72 caracteres
como máximo. El cuerpo explica **por qué** y **qué**, no repite el diff.

## Tipos

| Tipo | Uso |
| --- | --- |
| `feat` | Nueva funcionalidad visible o nueva capacidad |
| `fix` | Corrección de un error |
| `docs` | Solo documentación |
| `style` | Formato, espacios, punto y coma; sin cambio de comportamiento |
| `refactor` | Reestructuración sin cambio funcional |
| `perf` | Mejora de rendimiento |
| `test` | Tests nuevos o modificados |
| `build` | CMake, dependencias, empaquetado |
| `ci` | Workflows y automatización |
| `chore` | Mantenimiento, scripts, configuración no funcional |
| `revert` | Reversión de un commit anterior |

## Scopes

Se usan scopes compatibles con AzerothCore y con este proyecto:

| Scope | Ámbito |
| --- | --- |
| `Core` | Código del núcleo C++ |
| `DB/World` | Base de datos de mundo |
| `DB/Auth` | Base de datos de autenticación |
| `DB/Chars` | Base de datos de personajes |
| `DB/SAI` | SmartAI |
| `Scripts/<Área>` | Scripts de contenido, por zona, clase o dominio |
| `Module/<nombre>` | Módulo propio |
| `Project` | Documentación, roadmap y flujo del proyecto |
| `Agents` | `AGENTS.md`, configuración de OpenCode, guías de agentes |
| `Build` | CMake, compilación, dependencias |
| `CI` | GitHub Actions y automatización |
| `Docs` | Documentación general |

El subscope es opcional. Si el cambio afecta a varias áreas, elige el scope principal y detalla el
resto en el cuerpo.

## Ejemplos

```text
feat(Module/CustomCore): Add configuration for custom NPCs
fix(DB/World): Correct quest 1234 objective text
docs(Project): Document Windows build and GitFlow
chore(Agents): Add OpenCode config and permission rules
refactor(Scripts/Northrend): Reuse helper for phase transitions
build(Build): Enable static modules in local preset
ci(CI): Run SQL codestyle on pending updates
```

Ejemplos con cuerpo:

```text
fix(DB/World): Correct Hogger charge behavior

Hogger did not charge players because the SmartAI event was missing a
target condition. Add the condition and the corresponding action.

Closes #123
```

## Footer

- `Closes #123` o `Fixes #123` para cerrar issues.
- `Refs: #456` para referencias que no cierran.
- `Co-authored-by: Nombre <correo>` para autoría compartida.
- `BREAKING CHANGE: <descripción>` para cambios incompatibles.

Para marcar un cambio incompatible en el asunto:

```text
feat(Core/API)!: Remove deprecated player hook
```

## Plantilla local

El repositorio incluye `.git_commit_template.txt`. Actívala solo en este repositorio:

```powershell
git config commit.template .git_commit_template.txt
```

La plantilla original de AzerothCore se conserva. Este documento añade los scopes y el flujo del
proyecto sin reemplazarla.

## Título de Pull Request

Como las ramas `feature/*` y `fix/*` se integran con squash (o fast-forward cuando no hay PR), el
título del PR o el mensaje del commit de integración se convierte en el commit final. Debe cumplir el
mismo formato:

```text
feat(Project): Add local server setup documentation
```

## Reglas

- Un commit debe poder revertirse sin arrastrar cambios no relacionados.
- No mezclar formato, refactor y funcionalidad.
- No incluir secretos, credenciales, dumps, binarios ni datos del cliente.
- No usar mensajes vacíos como `update`, `fix`, `changes` o `wip`.
- Acreditar el código portado de otro core o proyecto, incluso si se reescribe.
- Si un commit corrige una regresión, menciona el commit o PR que la introdujo.

## Verificación antes de commitear

```powershell
git status --short
git diff
git diff --cached
git diff --check
```

Revisa que no haya archivos ignorados añadidos por error y que el mensaje describa el cambio real.
