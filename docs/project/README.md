# Proyecto WoW WotLK 3.3.5a

Repositorio personal basado en un fork de [AzerothCore](https://github.com/azerothcore/azerothcore-wotlk)
para World of Warcraft 3.3.5a (WotLK). El objetivo inicial es compilar y ejecutar un servidor local
funcional en Windows nativo; el objetivo a medio plazo es personalizar mecánicas, NPC, habilidades,
misiones y otros sistemas sin romper la compatibilidad con el núcleo original.

- Repositorio: `josuestaun/wow-wotlk`
- Upstream: `azerothcore/azerothcore-wotlk`
- Entorno: Windows 10/11 x64, PowerShell 5.1, VS Code, OpenCode
- Fuera de alcance: Docker, WSL2, servidores públicos, assets propietarios del cliente
- Licencia del núcleo: GPL v2 (ver `LICENSE` y `AUTHORS`)

## Estado actual

| Fase | Estado |
| --- | --- |
| 0. Fundación documental y de agentes | Completada (publicada en `origin/develop`) |
| 1. Servidor local MVP | Pendiente |
| 2. Base operativa (backups, actualizaciones) | Pendiente |
| 3. Módulo propio base | Pendiente |
| 4. Primer vertical de gameplay | Pendiente |

No se ha instalado ninguna dependencia ni compilado el servidor todavía. No se ha modificado la
lógica del juego.

El agente gestiona el ciclo Git rutinario (ramas, commits, push, PR y merge a `develop`) sin
aprobación individual; las operaciones destructivas y `master` requieren autorización. Ver
[GITFLOW.md](GITFLOW.md) y [AGENT_WORKFLOW.md](AGENT_WORKFLOW.md).

## Documentación

| Documento | Cuándo leerlo |
| --- | --- |
| [ARCHITECTURE.md](ARCHITECTURE.md) | Antes de tocar código, módulos, base de datos o configuración |
| [INSTALL_WINDOWS.md](INSTALL_WINDOWS.md) | Antes de instalar dependencias, compilar o ejecutar el servidor |
| [ROADMAP.md](ROADMAP.md) | Para elegir la siguiente tarea o revisar el alcance |
| [DECISIONS.md](DECISIONS.md) | Antes de tomar una decisión técnica o cambiar una ya tomada |
| [GITFLOW.md](GITFLOW.md) | Antes de crear una rama, abrir un PR o preparar una release |
| [COMMITS.md](COMMITS.md) | Antes de escribir un mensaje de commit o el título de un PR |
| [MODULES.md](MODULES.md) | Antes de crear o modificar un módulo, script o SQL de contenido |
| [REPO_HYGIENE.md](REPO_HYGIENE.md) | Antes de hacer `git add`, limpiar el árbol o añadir datos locales |
| [OPENCODE.md](OPENCODE.md) | Para configurar o usar OpenCode en este repositorio |
| [AGENT_WORKFLOW.md](AGENT_WORKFLOW.md) | Para entender la autonomía, verificación y límites del agente |

Las reglas de código de AzerothCore siguen viviendo en `.agents/docs/` y se enrutan desde
`AGENTS.md`. No se duplican aquí: este directorio documenta las decisiones y el flujo del proyecto,
no las convenciones internas del núcleo.

## Mapa del repositorio

| Ruta | Contenido | ¿Se versiona? |
| --- | --- | --- |
| `src/` | Núcleo C++ de AzerothCore | Sí, se modifica lo mínimo |
| `modules/` | Módulos externos; los propios empiezan por `mod-` | Solo código fuente propio |
| `data/sql/` | Esquema y actualizaciones de base de datos | Solo `updates/pending_db_*/` |
| `conf/` | Configuración local (`*.conf`) y plantillas (`conf/dist`) | Solo plantillas |
| `var/`, `build/`, `out/` | Build, logs y estado local | No |
| `env/` | Entorno de instalación/ejecución local | Solo `env/dist/.gitkeep` |
| `docs/project/` | Documentación de este proyecto | Sí |
| `.agents/` | Guías de agentes de AzerothCore | Sí, sin cambios innecesarios |
| `.github/` | CI y plantillas de GitHub | Sí |

## Convenciones documentales

- Idioma: español para la documentación de proyecto; inglés para las reglas de agentes y el código,
  siguiendo la convención de AzerothCore.
- Codificación UTF-8, finales de línea LF, sin espacios al final, máximo 120 columnas.
- Una decisión técnica relevante se registra como ADR en `DECISIONS.md` antes de implementarla.
- Los cambios de alcance se reflejan en `ROADMAP.md`.
- Los enlaces a documentación oficial deben apuntar a `azerothcore.org/wiki` o a la documentación
  de OpenCode, no a copias locales desactualizadas.

## Jerarquía de fuentes de verdad

1. El código y los datos del repositorio, para el estado real.
2. `AGENTS.md` y `.agents/docs/`, para las reglas de ingeniería del núcleo.
3. `docs/project/`, para las decisiones y el flujo propios de este fork.
4. La wiki oficial de AzerothCore, para procedimientos de instalación y referencia general.

Si hay conflicto entre una guía antigua y el código actual, gana el código; actualiza la guía en el
mismo cambio.

## Atribución y assets

Este proyecto no distribuye ni versiona archivos del cliente de World of Warcraft. Los datos
extraídos (`dbc`, `maps`, `vmaps`, `mmaps`, `cameras`) y cualquier MPQ pertenecen al cliente y se
guardan fuera del repositorio. El código derivado de AzerothCore conserva su licencia y autoría;
cualquier código portado de otro proyecto se acredita en el commit correspondiente.
