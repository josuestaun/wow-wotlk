# OpenCode en este proyecto

OpenCode es el agente principal del repositorio. La configuración de proyecto vive en
`opencode.json`, en la raíz. No contiene credenciales ni servicios de pago.

## Versión y modelo

- Versión instalada: `1.19.6`.
- Modelo por defecto: `opencode/deepseek-v4.1-flash-free`.
- El modelo es gratuito y se selecciona con el formato `proveedor/modelo`.
- Para ver los modelos disponibles:

```powershell
opencode.cmd models
```

Si el modelo gratuito no está disponible, elige otro de la lista `opencode/*-free` y actualiza
`opencode.json` y este documento en el mismo cambio.

## Configuración

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "opencode/deepseek-v4.1-flash-free",
  "default_agent": "build",
  "permission": {
    "bash": {
      "*": "allow",
      "git push": "ask",
      "git pull": "ask",
      "git pull origin master*": "ask",
      "git pull * master*": "ask",
      "git switch master": "ask",
      "git switch master *": "ask",
      "git checkout master": "ask",
      "git checkout master *": "ask",
      "git push origin master": "ask",
      "git push origin master *": "ask",
      "git push * master": "ask",
      "git push * master *": "ask",
      "git push --tags*": "ask",
      "git push * --tags*": "ask",
      "git push --follow-tags*": "ask",
      "git push * --follow-tags*": "ask",
      "git tag *": "ask",
      "git tag -l*": "allow",
      "git tag --list*": "allow",
      "git branch -D*": "ask",
      "git push * --delete*": "ask",
      "git push origin --delete feature/*": "allow",
      "git push origin --delete fix/*": "allow",
      "git restore *": "ask",
      "git checkout -- *": "ask",
      "git checkout .": "ask",
      "git stash drop*": "ask",
      "git stash clear*": "ask",
      "gh release *": "ask",
      "git push --force*": "deny",
      "git push -f*": "deny",
      "git push * --force*": "deny",
      "git push * -f*": "deny",
      "git push * +*": "deny",
      "git push --mirror*": "deny",
      "git reset --hard*": "deny",
      "git clean -f*": "deny",
      "git clean -*f*": "deny"
    }
  },
  "watcher": {
    "ignore": [
      "build/**",
      "out/**",
      "var/**",
      "env/dist/**",
      "env/user/**",
      "Data/**",
      "backups/**"
    ]
  }
}
```

Notas:

- OpenCode **1.x** usa `permission` (singular) y la clave `bash`. El formato `permissions` con
  `action`/`resource`/`effect` y `shell` es de OpenCode 2 y **no valida** en la versión instalada.
- Las reglas se evalúan en orden y **la última que coincide gana**. Por eso `"*": "allow"` va
  primero y las excepciones después.
- `git push` sin argumentos pide confirmación para evitar publicar la rama equivocada. Los pushes
  explícitos de ramas de trabajo (`git push -u origin feature/...`) están permitidos.
- Los pushes a `master`, la creación de tags y las releases piden confirmación.
- `git pull` sin argumentos pide confirmación; usa `git pull --ff-only origin develop`.
- Force push, `git reset --hard`, `git clean -f*` y `git clean -*f*` se bloquean.
- `git branch -D*` pide confirmación; `git branch -d` (solo ramas fusionadas) está permitido.
- El borrado remoto de `feature/*` y `fix/*` está permitido; otros borrados remotos piden
  confirmación.
- `git restore *`, `git checkout -- *`, `git checkout .` y `git stash drop/clear` piden confirmación
  para no perder cambios.
- `git merge` está permitido para integrar `feature/*`/`fix/*` en `develop`; no lo ejecutes estando
  en `master`.
- `gh pr create`, `gh pr view`, `gh pr checks` y `gh pr merge` están permitidos si `gh` existe y está
  autenticado. Un PR contra `master` requiere autorización.
- El resto de operaciones usan la política por defecto de OpenCode (permitidas), salvo lectura de
  `.env`, que pide confirmación.
- No uses `--auto` en este repositorio: saltaría las confirmaciones que protegen las operaciones
  destructivas.
- Valida siempre los cambios con `opencode.cmd debug config`.

La configuración global del usuario está en `%USERPROFILE%\.config\opencode\opencode.jsonc`. No
guardes secretos en la configuración del proyecto.

## Comandos útiles

```powershell
# Abrir OpenCode en el repositorio
opencode.cmd

# Ver la configuración resuelta (incluye modelo y permisos)
opencode.cmd debug config

# Ver agentes disponibles
opencode.cmd agent list

# Ver modelos disponibles
opencode.cmd models

# Ejecutar una instrucción sin abrir la interfaz
opencode.cmd run "Revisa el estado del repositorio"

# Logs de diagnóstico
opencode.cmd --print-logs --log-level DEBUG
```

En PowerShell 5.1, `opencode` puede resolverse al script `opencode.ps1` y fallar por la política de
ejecución. Usa `opencode.cmd` o ejecuta:

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

Este último comando cambia la política del usuario; ejecútalo solo si lo entiendes y lo autorizas.

## Cómo se cargan las instrucciones

- OpenCode descubre y carga `AGENTS.md` automáticamente.
- `AGENTS.md` enruta a `.agents/docs/` para las reglas de código de AzerothCore.
- `docs/project/` es documentación humana; el agente la lee cuando la tarea lo requiere.
- `AGENTS.md` contiene las reglas siempre activas; no dupliques reglas en varios sitios.
- Para instrucciones adicionales, OpenCode soporta la clave `instructions` con archivos o globs. No
  se usa por defecto para mantener el contexto ligero.

## Agentes y contexto

- `build`: agente principal para implementar cambios.
- `plan`: útil para diseñar antes de tocar código; no edita archivos.
- `explore`: búsquedas rápidas en el repositorio.
- `general`: tareas de investigación y ejecución multi-paso.

Buenas prácticas:

- Empieza por `AGENTS.md` y el documento de `docs/project/` que corresponda.
- Usa búsquedas (`glob`, `grep`) antes de leer archivos grandes.
- No pegues secretos, volcados de base de datos ni datos del cliente en la conversación.
- Revisa siempre el diff antes de aceptar un commit.
- Para cambios de varios pasos, usa una lista de tareas y actualízala.

## Skills y comandos

OpenCode descubre skills en `.opencode/skills/` y comandos en `.opencode/commands/`. El repositorio
incluye skills de AzerothCore en `.agents/skills/`; no las dupliques sin necesidad. Si añades una
skill propia, documenta su propósito y su trigger en `AGENTS.md` o en `.agents/docs/README.md` según
corresponda.

## Seguridad

- No introduzcas claves, tokens ni contraseñas en `opencode.json`, `AGENTS.md` ni en la documentación.
- No uses `--auto` para saltar permisos.
- No ejecutes comandos destructivos (`git reset --hard`, `git clean -f*`, force push) aunque el
  agente los proponga; están denegados por configuración.
- Puedes hacer push de ramas de trabajo, abrir PRs y fusionar cambios validados en `develop` sin
  autorización.
- Pide autorización para force push o reescritura de historia, resets destructivos o descarte de
  cambios, borrado de ramas no fusionadas, `master`, bases de datos o datos persistentes, y
  sobrescritura de recursos originales sin respaldo.
- No cambies los remotos configurados salvo que la tarea lo pida.
- Si un comando falla por permisos, no rodees la regla: explica el bloqueo y pide autorización.

## Solución de problemas

| Problema | Solución |
| --- | --- |
| `opencode.ps1` bloqueado | Usa `opencode.cmd` o ajusta la política de ejecución |
| Modelo no disponible | Ejecuta `opencode.cmd models` y elige otro `*-free` |
| Configuración ignorada | Ejecuta `opencode.cmd debug config` y revisa la ruta |
| Permiso denegado inesperado | Revisa el orden de `permission.bash`; la última regla gana |
| El agente no lee una guía | Enlázala desde `AGENTS.md` o ábrela explícitamente en la tarea |
| Respuestas con contexto insuficiente | Divide la tarea y referencia los archivos concretos |
