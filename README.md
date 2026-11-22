# WoW WotLK 3.3.5a — Proyecto personal AzerothCore

Fork personal de [AzerothCore](https://github.com/azerothcore/azerothcore-wotlk) para World of
Warcraft 3.3.5a (WotLK). El objetivo es ejecutar un servidor local en Windows nativo y, más adelante,
personalizar mecánicas, NPC, habilidades, misiones y otros sistemas mediante módulos y SQL.

- Repositorio: `josuestaun/wow-wotlk`
- Entorno: Windows 10/11, PowerShell 5.1, Visual Studio 2022, VS Code, OpenCode
- Estado: fundación documental y de agentes; servidor aún no compilado
- Fuera de alcance: Docker, WSL2, servidores públicos y assets propietarios del cliente

## Empezar

1. Lee `docs/project/README.md`.
2. Revisa `docs/project/ROADMAP.md` para la fase actual.
3. Sigue `docs/project/INSTALL_WINDOWS.md` cuando toque instalar o compilar.
4. Antes de commitear, consulta `docs/project/COMMITS.md` y `docs/project/REPO_HYGIENE.md`.

## Documentación

- [Índice del proyecto](docs/project/README.md)
- [Arquitectura](docs/project/ARCHITECTURE.md)
- [Instalación en Windows](docs/project/INSTALL_WINDOWS.md)
- [Roadmap](docs/project/ROADMAP.md)
- [Decisiones técnicas](docs/project/DECISIONS.md)
- [GitFlow](docs/project/GITFLOW.md)
- [Conventional Commits](docs/project/COMMITS.md)
- [Módulos](docs/project/MODULES.md)
- [Higiene del repositorio](docs/project/REPO_HYGIENE.md)
- [OpenCode](docs/project/OPENCODE.md)
- [Flujo del agente](docs/project/AGENT_WORKFLOW.md)

Las reglas para agentes de IA están en [AGENTS.md](AGENTS.md) y en `.agents/docs/`.

## Licencia y atribución

El núcleo deriva de AzerothCore y conserva su licencia GPL v2. Consulta `LICENSE` y `AUTHORS`. El
código portado de otros proyectos se acredita en el commit correspondiente. Este repositorio no
distribuye archivos del cliente de World of Warcraft.
