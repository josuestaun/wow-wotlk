# Decisiones técnicas (ADR)

Registro de decisiones de arquitectura y flujo de trabajo. Cada entrada es breve y estable: si una
decisión cambia, se añade una nueva entrada que reemplaza a la anterior en lugar de reescribir la
historia.

## Cómo añadir una decisión

Copia esta plantilla al final del archivo y usa el siguiente número libre:

```markdown
## ADR-0000: Título corto

- Estado: Propuesta | Aceptada | Reemplazada por ADR-0000 | Obsoleta
- Fecha: YYYY-MM-DD
- Contexto: qué problema o fuerza motiva la decisión.
- Decisión: qué se decide hacer.
- Consecuencias: qué mejora, qué empeora y qué queda pendiente.
- Alternativas: opciones descartadas y por qué.
```

## ADR-0001: Windows nativo como entorno principal

- Estado: Reemplazada por ADR-0013
- Fecha: 2026-11-22
- Contexto: el proyecto es personal, el equipo es una sola persona y se quiere evitar la complejidad
  de contenedores o subsistemas.
- Decisión: desarrollar, compilar y ejecutar en Windows 10/11 x64 con PowerShell 5.1, Visual Studio
  2022 y VS Code. Docker y WSL2 no forman parte del flujo principal.
- Consecuencias: se usan las instrucciones oficiales de AzerothCore para Windows; los scripts deben
  ser compatibles con PowerShell; no se asume `bash` salvo para herramientas opcionales como
  `create_module.sh` mediante Git Bash.
- Alternativas: Docker (más reproducible, pero añade capas y requisitos), WSL2 (build Linux, pero
  duplica el entorno).

## ADR-0002: Fork con upstream separado

- Estado: Aceptada
- Fecha: 2026-11-22
- Contexto: el proyecto parte de AzerothCore y necesita recibir actualizaciones sin perder las
  personalizaciones.
- Decisión: `origin` apunta al fork `josuestaun/wow-wotlk`; `upstream` apunta a
  `azerothcore/azerothcore-wotlk`. Las actualizaciones se integran desde `upstream/master` hacia
  `master` y luego hacia `develop`.
- Consecuencias: hay dos remotos y un flujo de sincronización; los conflictos se resuelven en ramas
  de integración. No se reescribe historia publicada.
- Alternativas: trabajar directamente sobre el fork sin upstream (pierde actualizaciones), o
  reescribir el fork (rompe la trazabilidad).

## ADR-0003: Documentación de proyecto en `docs/project/`

- Estado: Aceptada
- Fecha: 2026-11-22
- Contexto: la documentación de AzerothCore es extensa y no debe mezclarse con las decisiones
  personales del fork.
- Decisión: la documentación humana del proyecto vive en `docs/project/`. `.agents/docs/` se reserva
  para las reglas de ingeniería del núcleo y `doc/` para la documentación upstream existente.
- Consecuencias: los enlaces y la navegación son claros; las actualizaciones del upstream no pisan
  la documentación propia.
- Alternativas: un único `README.md` monolítico (difícil de mantener), o editar la wiki upstream
  (no aplica al fork).

## ADR-0004: GitFlow ligero

- Estado: Aceptada
- Fecha: 2026-11-22
- Contexto: se necesita un flujo simple para una persona, con ramas de integración y releases
  ocasionales.
- Decisión: `master` estable, `develop` integración, `feature/*`, `fix/*` y `hotfix/*`. No se crea
  una rama `release/*`; la release se prepara en `develop` y se integra en `master` con tag.
- Consecuencias: menos ramas y menos ceremonia; los hotfix nacen de `master` y se vuelven a integrar
  en `develop`.
- Alternativas: trunk-based (más simple, pero menos aislamiento para un proyecto de aprendizaje),
  GitFlow completo con `release/*` (demasiada ceremonia para una persona).

## ADR-0005: Conventional Commits y Pull Requests

- Estado: Aceptada
- Fecha: 2026-11-22
- Contexto: se quiere un historial legible, automatizable y compatible con las convenciones de
  AzerothCore.
- Decisión: usar Conventional Commits con scopes del proyecto (`Core`, `DB/*`, `Scripts/*`,
  `Module/*`, `Project`, `Agents`, `Docs`). Las ramas `feature/*` y `fix/*` se integran en `develop`
  mediante PR y squash merge. Los hotfix se integran en `master` y se vuelven a integrar en
  `develop`.
- Consecuencias: el título del PR se convierte en el mensaje del commit squash; se requiere una
  revisión mínima y CI en verde.
- Alternativas: mensajes libres (peor trazabilidad), merge commits para todo (historial más
  ruidoso).

## ADR-0006: Personalización mediante módulos primero

- Estado: Aceptada
- Fecha: 2026-11-22
- Contexto: modificar el núcleo complica las actualizaciones y mezcla código propio con upstream.
- Decisión: el orden de preferencia es datos/SmartAI, scripts de contenido, módulos propios y, solo
  como último recurso, un cambio mínimo en el núcleo. Todo cambio de núcleo requiere justificación,
  documentación y un diff pequeño.
- Consecuencias: más trabajo inicial en módulos, pero actualizaciones más limpias y menor riesgo de
  regresiones.
- Alternativas: parchear el núcleo directamente (rápido al principio, costoso al actualizar).

## ADR-0007: SQL solo en `pending_db_world` (y equivalentes)

- Estado: Aceptada
- Fecha: 2026-11-22
- Contexto: AzerothCore considera inmutables los directorios base, archive y las actualizaciones ya
  mergeadas.
- Decisión: los cambios de datos propios se añaden exclusivamente en
  `data/sql/updates/pending_db_*/`. Los SQL de módulos viven dentro del módulo. No se editan
  `data/sql/base/`, `data/sql/archive/` ni `data/sql/updates/db_*/`.
- Consecuencias: los cambios son reproducibles y el actualizador los aplica en orden; se evita
  romper la trazabilidad upstream.
- Alternativas: editar directamente la base de datos (no reproducible), tocar `base/` (rompe
  actualizaciones).

## ADR-0008: OpenCode con modelo gratuito y permisos restrictivos

- Estado: Reemplazada parcialmente por ADR-0012
- Fecha: 2026-11-22
- Contexto: se usa OpenCode como agente principal y no se quieren credenciales ni servicios de pago
  en el repositorio.
- Decisión: configuración de proyecto en `opencode.json` con el modelo
  `opencode/deepseek-v4.1-flash-free`. Los comandos `git push*` piden autorización; `git reset
  --hard`, `git clean -f*` y los force push se deniegan. La versión instalada (1.19.6) usa el
  esquema V1 (`permission`/`bash`); el esquema V2 (`permissions`/`shell`) no se usa porque no valida
  en esta versión.
- Consecuencias: el agente puede trabajar de forma autónoma en cambios reversibles, pero no puede
  publicar ni destruir historia sin intervención humana. Las credenciales, si alguna vez se usan,
  se guardan en la configuración global de OpenCode, nunca en el repo.
- Alternativas: modelo de pago (no deseado), configuración global única (menos reproducible para el
  proyecto).
- Nota: ADR-0012 sustituye la política de push; el modelo gratuito, el esquema V1 y el bloqueo de
  operaciones destructivas siguen vigentes.

## ADR-0009: Repositorio solo para código y documentación

- Estado: Aceptada
- Fecha: 2026-11-22
- Contexto: el repositorio debe ser ligero, seguro y reproducible; los datos vivos y los binarios no
  aportan valor al historial.
- Decisión: no versionar secretos, configuraciones locales, binarios, builds, bases de datos,
  backups, logs, datos extraídos del cliente ni assets propietarios. Se amplía `.gitignore` y se
  documenta en `REPO_HYGIENE.md`.
- Consecuencias: clones más rápidos y menos riesgo de filtrar información; cada entorno reconstruye
  sus datos locales.
- Alternativas: usar Git LFS para binarios (no se necesita), versionar configuraciones (riesgo de
  secretos).

## ADR-0010: Build fuera del árbol y datos fuera del repo

- Estado: Aceptada
- Fecha: 2026-11-22
- Contexto: CMake exige build out-of-source y los datos del cliente son grandes y no versionables.
- Decisión: build en `var/build` o `build/`; instalación en `env/dist`; datos del cliente en una
  ruta externa como `E:\WoW-Data`; backups en `E:\WoW-Backups`.
- Consecuencias: el árbol de trabajo queda limpio; las rutas se configuran en `worldserver.conf` y
  no se commitean.
- Alternativas: build dentro de `src/` (bloqueado por CMake), datos dentro del repo (descartado por
  tamaño y licencia).

## ADR-0011: Documentación de proyecto en español; reglas de agente en inglés

- Estado: Aceptada
- Fecha: 2026-11-22
- Contexto: el autor trabaja en español, pero las reglas de AzerothCore y las convenciones de código
  están en inglés.
- Decisión: `docs/project/` se escribe en español; `AGENTS.md` y `.agents/docs/` se mantienen en
  inglés para no divergir del upstream.
- Consecuencias: la documentación humana es natural para el autor y las reglas de ingeniería siguen
  siendo compatibles con AzerothCore.
- Alternativas: todo en inglés (menos natural para el autor), todo en español (dificulta futuras
  comparaciones con upstream).

## ADR-0012: Autonomía Git supervisada y validación antes de integrar

- Estado: Aceptada
- Fecha: 2026-11-22
- Contexto: el autor quiere centrarse en definir y validar gameplay sin aprobar cada operación Git
  rutinaria. La política anterior exigía autorización para cada push y PR, lo que interrumpía el
  trabajo.
- Decisión: el agente puede consultar estado y remotos, hacer fetch y pull seguro, crear y cambiar
  ramas, commitear, hacer push de ramas de trabajo a `origin`, abrir PRs si hay herramienta,
  fusionar `feature/*` y `fix/*` validados en `develop`, resolver conflictos simples y borrar ramas
  fusionadas. Requiere autorización para force push o reescritura de historia, reset destructivo o
  descarte de cambios, borrado de ramas no fusionadas, `master`, bases de datos o datos persistentes,
  y sobrescritura de recursos originales sin respaldo. Una compilación correcta no valida gameplay:
  si hace falta probar en el cliente, se espera confirmación del usuario antes de integrar.
- Consecuencias: menos interrupciones y más velocidad; la seguridad depende de los permisos, la
  verificación y la disciplina del agente. `master`, los datos persistentes y las operaciones
  destructivas siguen siendo manuales.
- Alternativas: mantener autorización para cada push (más control, más fricción), autonomía total
  sin límites (riesgo de pérdida de trabajo o datos).

## ADR-0013: Entorno de ejecución pendiente (Windows nativo o Docker)

- Estado: Reemplazada por ADR-0015
- Fecha: 2026-11-23
- Contexto: ADR-0001 fijó Windows nativo, pero el autor todavía no ha decidido si usará Windows
  nativo o Docker. El objetivo inmediato es la fundación documental y de agentes, no compilar ni
  instalar dependencias.
- Decisión: dejar la decisión pendiente. No se configura ni se documenta un build definitivo;
  `INSTALL_WINDOWS.md` queda como opción provisional, no como decisión cerrada. Docker y WSL2 no se
  adoptan ni se prohíben. Cuando se decida, se añadirá un ADR que la cierre y una guía del entorno
  elegido.
- Consecuencias: los documentos no deben asumir un entorno de ejecución; el trabajo se centra en
  código, datos y agentes. La fase 1 del roadmap no empieza hasta cerrar esta decisión.
- Alternativas: Windows nativo (ADR-0001, ahora provisional), Docker (más reproducible, añade
  capas), WSL2 (no se considera como entorno principal).

## ADR-0014: Reutilizar los recursos de agentes de AzerothCore en OpenCode

- Estado: Aceptada
- Fecha: 2026-11-23
- Contexto: AzerothCore ya incluye `AGENTS.md`, `.agents/docs/`, `.agents/skills/` y perfiles para
  Claude Code y GitHub Copilot. Copiarlos a `.opencode/` crearía dos copias que se desincronizan.
- Decisión: OpenCode lee `AGENTS.md` y descubre las skills en `.agents/skills/`. Solo se añaden
  recursos propios de OpenCode cuando no hay equivalente, como el subagente
  `.opencode/agent/pr-reviewer.md`, que enruta a las mismas fuentes. Se conservan `CLAUDE.md`,
  `.claude/` y `.github/` sin cambios. La configuración usa el esquema V1 de OpenCode 1.19.6
  (`permission`, `.opencode/agent/` en singular).
- Consecuencias: una sola copia de las reglas; las actualizaciones de upstream se conservan. El
  symlink de `.claude/skills` no se resuelve en Windows con `core.symlinks=false`, pero OpenCode usa
  la skill real de `.agents/skills/`.
- Alternativas: copiar todo a `.opencode/` (duplicación), depender solo de Claude/Copilot (no
  aplica), reescribir las guías en formato OpenCode (pierde compatibilidad con upstream).

## ADR-0015: Docker Desktop + WSL2 como entorno de ejecución

- Estado: Aceptada
- Fecha: 2026-11-23
- Contexto: el autor ha elegido Docker para el servidor local. El equipo es Windows 11 Home, con
  Ryzen 7 3700X y 16 GB de RAM; la virtualización por hardware está disponible pero deshabilitada en
  la BIOS. Windows Home solo permite contenedores Linux, y el backend recomendado por Docker es
  WSL2. AzerothCore mantiene una infraestructura Docker oficial en el propio repositorio.
- Decisión: usar Docker Desktop con backend WSL2 y los archivos Docker oficiales de AzerothCore
  (`docker-compose.yml`, `apps/docker/*`). El servidor se construye desde este checkout con
  `docker compose build` y se opera con `docker compose up -d`. Los puertos se publican solo en
  `127.0.0.1`; no se abre el router ni se expone a Internet.
- Consecuencias: se necesita habilitar `SVM Mode` en la BIOS y reiniciar, instalar WSL2 y Docker
  Desktop, y reservar recursos (8 CPUs, 12 GB RAM, 4 GB swap). El build inicial es lento, pero el
  entorno es reproducible y las actualizaciones de código se prueban reconstruyendo imágenes. La
  instalación nativa de Windows queda como alternativa, no como ruta principal.
- Alternativas: Windows nativo (ADR-0001/ADR-0013, más pasos manuales de dependencias), WSL2 sin
  Docker (menos reproducible), Docker en otra máquina Linux (fuera del alcance local).

## ADR-0016: Almacenamiento en E: y build reproducible con imágenes

- Estado: Aceptada
- Fecha: 2026-11-23
- Contexto: C: tiene ~303 GB libres y E: ~745 GB; las imágenes, volúmenes, bases y datos de cliente
  son grandes y conviene mantenerlos fuera de C:. El flujo de desarrollo exige recompilar C++,
  módulos y SQL con frecuencia.
- Decisión: instalar Docker Desktop con `--wsl-default-data-root=E:\Docker\wsl` y fijar el disk image
  location en E:. Usar volúmenes nombrados para `ac-database` y `ac-client-data` (quedan dentro del
  data-root en E:) y bind mounts en `env/dist/etc` y `env/dist/logs` del repo en E:. Compilar las
  imágenes desde el checkout con `docker compose build` y etiquetarlas como `local`; no depender de
  imágenes precompiladas de AzerothCore. Los datos de cliente se descargan con el contenedor oficial
  desde `wowgaming/client-data` v20.0. El build usa MySQL 8.4 oficial y no la imagen de base
  preimportada, para evitar desajustes de esquema con este checkout.
- Consecuencias: C: no crece con imágenes ni bases; el entorno es reproducible y actualizable. El
  primer build y la primera importación tardan; los builds posteriores reutilizan caché. El
  desarrollo intensivo con bind mounts sobre E: puede ser lento, por lo que se documenta la opción de
  clonar en WSL2 si hace falta.
- Alternativas: data-root en C: (llena el disco del sistema), bind mounts para bases y datos (más
  lentos y frágiles en Windows), imágenes precompiladas (riesgo de esquema desalineado), volúmenes
  anónimos (difíciles de respaldar).
