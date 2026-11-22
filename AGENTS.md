# AGENTS.md

AzerothCore is a C++ MMORPG server emulator for World of Warcraft 3.3.5a (WotLK), built with CMake, backed by MySQL.

## Personal project context

This repository is a personal WoW WotLK 3.3.5a project built on a fork of AzerothCore
(`josuestaun/wow-wotlk`). Development happens on native Windows 10/11 (PowerShell 5.1), VS Code, and
OpenCode; Docker and WSL2 are out of scope. The first goal is a working local server; the long-term
goal is gameplay customization through modules, scripts, and SQL.

Project documentation lives in `docs/project/` (human-facing, Spanish). Start at
`docs/project/README.md`. The rules below are always active; the upstream AzerothCore rules further
down are preserved and still apply.

## Project agent rules

- Follow `docs/project/GITFLOW.md`: `master` is stable, `develop` is integration, and work happens
  on `feature/*`, `fix/*`, or `hotfix/*`. Do not commit directly to `master`. The initial bootstrap
  commit on `develop` is the only documented exception.
- Use Conventional Commits per `docs/project/COMMITS.md` and `.git_commit_template.txt`.
- **Never push, open or update a PR, change remotes, or rewrite published history without explicit
  user authorization.** `git push` is ask-gated in `opencode.json`.
- Never commit secrets, credentials, `.env`, `conf/*.conf`, database dumps, build artifacts, logs,
  client data (`MPQ`, `DBC`, maps, vmaps, mmaps), or proprietary assets. See
  `docs/project/REPO_HYGIENE.md`.
- Prefer data/SmartAI, content scripts, then modules over core edits. See `docs/project/MODULES.md`.
  Any core change must be minimal, justified, and documented in `docs/project/DECISIONS.md`.
- Do not modify game logic yet; the current milestone is infrastructure and a working local server.
- Do not install server dependencies or build unless the user explicitly asks. When asked, read
  `docs/project/INSTALL_WINDOWS.md` first.
- Record new architectural decisions in `docs/project/DECISIONS.md` and scope changes in
  `docs/project/ROADMAP.md`.
- Keep upstream documentation and `.agents/docs/` intact unless the change is required and
  explained. Credit upstream authors when porting code.
- On Windows, use PowerShell 5.1 semantics: no `&&`, quote paths with spaces, and do not assume
  `bash`, `make`, or `nproc`.

## AzerothCore agent rules

- **Do not configure or build unless explicitly asked.** Builds are slow and rarely needed for code changes.
- **Never edit SQL files outside `data/sql/updates/pending_db_*/` unless explicitly requested.** `data/sql/base/`,
  `data/sql/archive/` and `data/sql/updates/db_*/` are immutable.
- Formatting follows `.editorconfig`: UTF-8, LF, max 120 cols, trailing newline, no trailing whitespace; 4-space indent
  for C++ (tabs forbidden), 2-space for JSON/YAML/sh/ts/js.
- **Do not write live-stack e2e tests unless the user explicitly asks**, nor `e2e/local/` scratch tests to validate a
  fix; don't mention missing coverage unless asked. Existing `e2e/` tests that break after a change may be updated.
  Before writing or changing any `e2e/` test, read `e2e/README.md` and `.agents/docs/e2e-policy.md`.
- Planning docs go in `.agents/plans/<task-slug>/` (gitignored), named `<task-slug>.<TYPE>.md` (`PLAN`, `REQUIREMENTS`,
  `ANALYSIS`, …).
- **Credit upstream authors.** Code, a mechanism, or data mirrored from another core (TrinityCore, cMaNGOS, …) is
  committed with `--author` naming the original commit's author (extra sources as `Co-authored-by`), even when rewritten
  against AC or confirmed by own sniffs; find them in the upstream file's commit history.

## Repository layout

- `src/common/` — networking (Asio), crypto, config, logging, shared utilities.
- `src/server/game/` — core gameplay; compiled into worldserver.
- `src/server/scripts/` — content scripts grouped by region (`EasternKingdoms/`, `Northrend/`, …), class
  (`Spells/spell_mage.cpp`, …), and domain (`Commands/`, `Pet/`, `OutdoorPvP/`, `World/`).
- `src/server/database/` — DB abstraction and schema updater.
- `src/server/shared/` — code shared by auth and world servers.
- `src/server/apps/{authserver,worldserver}/` — entry points (ports 3724 and 8085).
- `src/test/` — unit tests + mocks.
- `e2e/` — live-stack Go e2e (AzerothGhost harness); see `e2e/README.md`.
- `data/sql/` — `base/` (historical schema), `updates/db_*/` (merged), `updates/pending_db_*/` (in-flight), `custom/`
  (gitignored).
- `modules/` — external modules (see below).
- `apps/` — helper scripts; `apps/codestyle/` holds the lint scripts.
- `conf/dist/` — distributed config templates; `conf/*.conf` is gitignored.
- `deps/` — vendored third-party dependencies.

## Modules

External modules live in `modules/`, each in its own subdir with a `src/` folder. Modern modules are
auto-discovered; a `CMakeLists.txt` is optional and a `<ModuleName>.cmake` is only needed for advanced
CMake integration. Disable with `-DDISABLED_AC_MODULES="mod1;mod2"`. See
`modules/how_to_make_a_module.md` and `docs/project/MODULES.md`.

## Project documentation

- `docs/project/README.md` — index, conventions, and source-of-truth hierarchy.
- `docs/project/ARCHITECTURE.md` — components, data flow, and extension points.
- `docs/project/INSTALL_WINDOWS.md` — native Windows setup, build, database, and client data.
- `docs/project/ROADMAP.md` — phases, acceptance criteria, and backlog.
- `docs/project/DECISIONS.md` — ADR log; add an entry before a new architectural decision.
- `docs/project/GITFLOW.md` — branches, PRs, releases, hotfixes, and upstream sync.
- `docs/project/COMMITS.md` — Conventional Commits, scopes, and examples.
- `docs/project/MODULES.md` — customization strategy and module layout.
- `docs/project/REPO_HYGIENE.md` — secrets, binaries, local data, and proprietary assets.
- `docs/project/OPENCODE.md` — OpenCode configuration, model, and permissions.
- `docs/project/AGENT_WORKFLOW.md` — autonomy, verification, escalation, and reporting.

## Persisting lessons

When a user correction reveals a lesson that generalizes, offer to persist it into these docs (placement per
`.agents/docs/README.md`): use the `/self-improve` skill if installed, otherwise suggest the user to install it and read
this page: https://www.azerothcore.org/wiki/agentic-engineering

## Read also

BEFORE starting the task, and again once you know what you will change, read every doc whose trigger matches, then
follow each doc's own "Read also" list:

- Writing or modifying any C++ (`src/**`, `modules/**`) → `.agents/docs/cpp-guidelines.md`
- Any change under `src/server/scripts/**` → `.agents/docs/cpp-scripts.md`
- Creating or modifying any SQL (`*.sql`) → `.agents/docs/sql-guidelines.md`
- Any change under `src/server/scripts/**` or `data/sql/updates/pending_db_world/**`, or any `smart_scripts` SQL →
  `.agents/docs/retail-evidence.md`
- Compiling, configuring or running tests, plan-given commands included; editing CMake files, `deps/**`,
  `.github/workflows/**`, `.github/actions/**` or `.github/SECURITY.md` → `.agents/docs/build.md`
- Editing any `*.conf.dist` → `.agents/docs/systems/config.md`
- Reviewing a changeset or PR, or opening or updating a PR → `.agents/docs/code-review.md`
- Self-reviewing, or opening or updating a PR → also `.agents/docs/self-review-rules.md`
- Capturing a lesson or adding/updating agent docs → `.agents/docs/README.md`
- Branching, committing, or opening a PR → `docs/project/GITFLOW.md` and `docs/project/COMMITS.md`
- Creating or changing a custom module → `docs/project/MODULES.md`
- Making an architectural decision → `docs/project/DECISIONS.md`
- Changing `opencode.json`, agent permissions, or the OpenCode workflow → `docs/project/OPENCODE.md`
- Committing, cleaning the tree, or adding local data → `docs/project/REPO_HYGIENE.md`
- Installing, configuring, or building on Windows → `docs/project/INSTALL_WINDOWS.md`
