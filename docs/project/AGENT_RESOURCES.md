# Recursos de agentes de AzerothCore y adaptación a OpenCode

Auditoría de las instrucciones y herramientas para agentes de IA que ya existían en el repositorio
original de AzerothCore, y de cómo se reutilizan en este fork con OpenCode.

- Estado: completada el 2026-11-23.
- Alcance: documentación y configuración de agentes. No se ha tocado la lógica del juego.
- Principio: una sola fuente de verdad (`.agents/` + `AGENTS.md`); OpenCode lee esos recursos en vez
  de duplicarlos.

## Inventario original

| Recurso | Para quién | Contenido | ¿Se conserva? |
| --- | --- | --- | --- |
| `AGENTS.md` | Cualquier agente | Reglas de ingeniería y contexto del fork | Sí, fuente de verdad |
| `CLAUDE.md` | Claude Code | Una línea: `@AGENTS.md` (import) | Sí |
| `.claude/skills/generate-pr-description` | Claude Code | Symlink a `.agents/skills/` | Sí |
| `.agents/docs/` | Cualquier agente | Guías técnicas por tarea (C++, SQL, scripts, e2e) | Sí |
| `.agents/docs/systems/` | Cualquier agente | Guías de sistemas (SmartAI, spells, IA, …) | Sí |
| `.agents/skills/generate-pr-description/SKILL.md` | Claude y compatibles | Skill para redactar un PR | Sí |
| `.agents/README.md` | Cualquier agente | Convenciones de skills y conexión | Sí |
| `.github/copilot-instructions.md` | GitHub Copilot | Revisar PR según `.agents/docs/code-review.md` | Sí |
| `.github/agents/pr-reviewer.md` | GitHub Copilot | Perfil de revisor que apunta a `AGENTS.md` | Sí |

No existen comandos de agente propios en el repositorio original (`.claude/commands/`,
`.agents/commands/`); solo hay una skill.

## Cómo los consume OpenCode

OpenCode 1.19.6 descubre automáticamente:

- `AGENTS.md` del proyecto como instrucciones base.
- Skills en `.agents/skills/` y `.claude/skills/` (compatibilidad), además de `.opencode/skills/`.
- Subagentes en `.opencode/agent/<nombre>.md`.

Verificado en este equipo:

- `generate-pr-description` aparece como skill disponible, resuelta desde
  `.agents/skills/generate-pr-description/SKILL.md`.
- `pr-reviewer` aparece en `opencode.cmd agent list` como `subagent` y aplica `edit: deny`.

## Recursos reutilizados sin cambios

- `AGENTS.md`: contiene las reglas siempre activas y el enrutado a `.agents/docs/`. OpenCode lo lee
  nativamente; no hace falta un archivo puente.
- `.agents/docs/` y `.agents/docs/systems/`: se leen bajo demanda según los triggers de `AGENTS.md`.
- `.agents/skills/generate-pr-description/`: se usa tal cual. Depende de `pull_request_template.md`
  (existe) y de `git diff origin/master...HEAD`, coherente con el flujo del fork.
- `CLAUDE.md`, `.claude/` y `.github/`: se conservan intactos para Claude Code y GitHub Copilot.

## Adaptaciones realizadas a OpenCode

1. `.opencode/agent/pr-reviewer.md` (nuevo): adaptación fina de `.github/agents/pr-reviewer.md`.
   OpenCode no lee los perfiles de Copilot, así que se crea un subagente equivalente que apunta a
   las mismas fuentes (`AGENTS.md`, `.agents/docs/code-review.md`, `.agents/docs/self-review-rules.md`)
   y prohíbe editar archivos. No restablece reglas: solo enruta.
2. `docs/project/OPENCODE.md`: se documenta el descubrimiento de skills y subagentes, la precedencia
   y la limitación de symlinks en Windows.
3. `docs/project/DECISIONS.md`: ADR-0014 registra la reutilización de `.agents/` como fuente de
   verdad y la política de punteros finos; ADR-0013 deja pendiente la decisión Windows/Docker.
4. `docs/project/README.md` y `AGENTS.md`: se ajusta el entorno para no afirmar que Docker está
   fuera de alcance mientras la decisión siga pendiente.

No se copian skills ni docs a `.opencode/`: duplicar crearía dos copias que se desincronizan. La
precedencia de OpenCode permite que `.opencode/` sobrescriba, pero solo debe usarse para recursos
propios de OpenCode.

## Incompatibilidades y limitaciones detectadas

### Symlink de `.claude/skills` en Windows

El repositorio versiona `.claude/skills/generate-pr-description` como symlink (modo `120000`), pero
en este equipo `core.symlinks=false`. Git lo descarga como archivo de texto de 44 bytes con el
contenido `../../.agents/skills/generate-pr-description`. Consecuencias:

- Claude Code no encuentra la skill desde `.claude/skills/` mientras `core.symlinks` siga en `false`.
- OpenCode no se ve afectado: descubre la skill real en `.agents/skills/`.
- No se modifica `.claude/` porque es un archivo original de AzerothCore. Si se necesita Claude
  Code, activar Developer Mode o una shell elevada y `git config core.symlinks true`, y volver a
  hacer checkout del enlace.

### Sintaxis específica de cada agente

- `CLAUDE.md` usa el import `@AGENTS.md`, que solo entiende Claude Code. OpenCode no lo necesita.
- `.github/copilot-instructions.md` y `.github/agents/` solo los lee GitHub Copilot. OpenCode los
  ignora; las reglas que referencian ya están enroutadas desde `AGENTS.md`.
- La skill `generate-pr-description` menciona la skill `use-conversational-language` de
  `agent-toolkit`. No está instalada; la propia skill indica que se puede obtener de su fuente
  remota. No se vendoriza para no añadir dependencias.

### Esquema V1 frente a V2

La documentación pública de OpenCode describe el esquema V2 (`permissions`, `.opencode/agents/`,
`agents`). La versión instalada, 1.19.6, usa V1: `permission`, `.opencode/agent/` (singular) y la
clave `agent` en la configuración. Los recursos añadidos siguen V1 para que se carguen realmente.

### Remoto `upstream` no configurado

ADR-0002 prevé un remoto `upstream` hacia `azerothcore/azerothcore-wotlk`, pero el clon solo tiene
`origin`. No se cambian remotos sin petición explícita; queda anotado para cuando se retome la
sincronización con upstream.

## Contradicciones detectadas

- Docker/WSL2: `AGENTS.md`, `docs/project/README.md` y `docs/project/ROADMAP.md` los daban por
  fuera de alcance, mientras la decisión de entorno sigue pendiente. Resuelto con ADR-0013: se deja
  pendiente y no se asume ni Windows nativo ni Docker.
- `INSTALL_WINDOWS.md` sigue siendo la guía de Windows nativo; es válida como opción provisional,
  no como decisión cerrada. Si se elige Docker, se añadirá una guía equivalente y se actualizará
  ADR-0013.

## Mantenimiento

- Añadir una skill nueva en `.agents/skills/<nombre>/SKILL.md`; OpenCode la descubre sin config.
- Añadir un subagente propio de OpenCode en `.opencode/agent/<nombre>.md` (V1), con `mode: subagent`
  y un `description` claro.
- Si una skill de `.agents/skills` necesita un override solo para OpenCode, crear la misma ID en
  `.opencode/skills/`; la precedencia de OpenCode hará que gane la de `.opencode/`.
- No duplicar reglas: los perfiles y skills deben enrutar a `AGENTS.md` y `.agents/docs/`.
