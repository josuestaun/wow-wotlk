# Arquitectura

## Vista general

AzerothCore separa la autenticación del mundo del juego en dos procesos y tres bases de datos
MySQL. El cliente 3.3.5a habla con `authserver` para autenticarse y con `worldserver` para jugar.

```text
Cliente WoW 3.3.5a
        |
        | 3724 (auth)
        v
  +-------------+        +------------------+
  | authserver  | -----> | acore_auth       |
  +-------------+        +------------------+
        |
        | 8085 (world)
        v
  +-------------+        +------------------+      +----------------------+
  | worldserver | -----> | acore_world      |      | acore_characters     |
  +-------------+        +------------------+      +----------------------+
        |                       ^
        |                       |
        +--> DataDir -----------+  dbc / maps / vmaps / mmaps / cameras
        |
        +--> Scripts estáticos + Modules (static o dynamic)
```

## Componentes

| Componente | Responsabilidad | Código |
| --- | --- | --- |
| `authserver` | Login, realmlist, sesión inicial | `src/server/apps/authserver` |
| `worldserver` | Mundo, jugadores, criaturas, spells, quests, IA | `src/server/apps/worldserver` |
| `acore_auth` | Cuentas, realmlist, sesiones | `data/sql/base/db_auth`, `data/sql/updates` |
| `acore_world` | Contenido estático: NPC, quests, spells, loot | `data/sql/base/db_world`, `data/sql/updates` |
| `acore_characters` | Estado persistente de personajes e inventario | `data/sql/base/db_characters` |
| Scripts | Contenido y comportamiento del mundo | `src/server/scripts/` |
| Modules | Extensiones autocontenidas | `modules/` |
| Herramientas | Extractores de datos del cliente | `src/tools/`, `apps/extractor/` |
| CMake | Configuración, build e instalación | `CMakeLists.txt`, `src/cmake/` |

## Flujo de login y entrada al mundo

1. El cliente envía credenciales a `authserver` en el puerto 3724.
2. `authserver` valida contra `acore_auth` y devuelve la lista de reinos (`realmlist`).
3. El cliente conecta con `worldserver` en el puerto 8085.
4. `worldserver` valida la sesión, carga el personaje desde `acore_characters` y el contenido desde
   `acore_world`.
5. El mundo usa `DataDir` para DBC, mapas, vmaps, mmaps y cámaras. Sin `dbc` y `maps` el servidor no
   funciona; sin `vmaps`/`mmaps` el movimiento y la línea de visión fallan de forma sutil.

En local, `realmlist` apunta a `127.0.0.1` y el cliente usa `set realmlist 127.0.0.1`.

## Build

- CMake >= 3.16, C++20, MSVC 2022 en Windows x64.
- Build fuera del árbol: `var/build` o `build/`, nunca dentro de `src/`.
- `SCRIPTS=static` y `MODULES=static` para un servidor local simple.
- `TOOLS_BUILD=all` solo cuando se necesiten los extractores.
- `RelWithDebInfo` para desarrollo: mantiene símbolos y permite crash logs útiles.
- `compile_commands.json` se exporta automáticamente para VS Code / clangd.

## Directorios en Windows

| Uso | Ruta recomendada | Versionado |
| --- | --- | --- |
| Fuente | `E:\proyectos\WoW-WotLK` | Sí |
| Build | `E:\proyectos\WoW-WotLK\var\build` | No |
| Instalación | `E:\proyectos\WoW-WotLK\env\dist` | No |
| Datos del cliente | `E:\WoW-Data` (fuera del repo) | No |
| Configuración local | `conf\*.conf` | No |
| Base de datos MySQL | `C:\ProgramData\MySQL\...` | No |
| Backups | `E:\WoW-Backups` (fuera del repo) | No |

`DataDir` en `worldserver.conf` debe apuntar a la carpeta que contiene `dbc`, `maps`, `vmaps`,
`mmaps` y `cameras`. Usa barras normales o barras escapadas en la configuración.

## Puntos de extensión

Orden de preferencia para personalizar el juego, de menor a mayor impacto en el núcleo:

1. **Datos y SmartAI**: `data/sql/updates/pending_db_world/` para NPC, loot, quests, condiciones,
   eventos y comportamiento declarativo.
2. **Scripts de contenido**: `src/server/scripts/` para lógica específica de una zona, clase o
   instancia, siguiendo las guías de `.agents/docs/cpp-scripts.md`.
3. **Módulos**: `modules/mod-<tema>/` para sistemas propios, con configuración, hooks y SQL
   encapsulados. Es la vía preferida para mecánicas nuevas.
4. **Cambio mínimo en el núcleo**: solo si no existe hook, API o evento. Debe justificarse en un ADR,
   documentarse y mantener el diff pequeño para facilitar futuras actualizaciones.

No se aceptan parches grandes al núcleo ni reescrituras de sistemas existentes sin una razón
arquitectónica documentada.

## Persistencia y límites

- `acore_characters` es estado vivo: se respalda antes de actualizar, migrar o experimentar.
- `acore_world` y `acore_auth` se pueden recrear desde SQL, pero un backup acelera la recuperación.
- Las configuraciones locales y los datos del cliente nunca se versionan.
- Los módulos propios se versionan como código fuente; sus bases de datos locales no.
- Los binarios, DLLs, PDBs y artefactos de build quedan fuera del repositorio.

## Actualizaciones del upstream

El fork mantiene `origin` como repositorio propio y un remoto `upstream` hacia AzerothCore. La
estrategia es integrar `upstream/master` en `master` y luego `master` en `develop`, resolviendo
conflictos en una rama de integración y verificando el build antes de continuar. Nunca se reescribe
historia publicada.
