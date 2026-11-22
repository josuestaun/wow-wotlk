# Estrategia de módulos y personalización

El objetivo es personalizar el juego sin convertir el fork en un núcleo irreconocible. Cada cambio
debe poder actualizarse desde AzerothCore con el mínimo conflicto posible.

## Orden de preferencia

1. **Datos y SmartAI** (`data/sql/updates/pending_db_world/`): NPC, loot, quests, condiciones,
   eventos, textos y comportamiento declarativo.
2. **Scripts de contenido** (`src/server/scripts/`): lógica de una zona, clase, instancia o spell que
   ya tiene un punto de extensión claro.
3. **Módulo propio** (`modules/mod-<tema>/`): sistemas nuevos, configuración propia, comandos,
   hooks y SQL encapsulado.
4. **Cambio mínimo en el núcleo**: solo si no existe hook, evento, API o dato equivalente. Requiere
   ADR, justificación y un diff pequeño y revisable.

Si dudas entre dos niveles, elige el de menor impacto. Un módulo puede crecer; un parche al núcleo
se paga en cada actualización.

## Anatomía de un módulo moderno

En este fork los módulos se autodescubren: cualquier directorio bajo `modules/` que tenga una
subcarpeta `src/` se compila. No es obligatorio un `CMakeLists.txt`; solo se usa un archivo
`<Modulo>.cmake` para configuraciones avanzadas.

```text
modules/mod-wowwotlk-ejemplo/
  README.md
  conf/
    mod_wowwotlk_ejemplo.conf.dist
  data/
    sql/
      world/
        2026_11_22_00_mod_wowwotlk_ejemplo.sql
  src/
    mod_wowwotlk_ejemplo.cpp
    mod_wowwotlk_ejemplo_loader.cpp
```

Convenciones:

- Nombre de directorio: `mod-wowwotlk-<tema>` en kebab-case.
- Prefijo de archivos y configuración: `mod_wowwotlk_<tema>`.
- La configuración se instala en `configs/modules/` y se lee con `sConfigMgr`.
- El SQL del módulo vive dentro del módulo, no en `data/sql/base/`.
- El módulo debe poder desactivarse con `-DDISABLED_AC_MODULES="mod-wowwotlk-ejemplo"`.
- El README del módulo explica qué hace, cómo se configura, cómo se prueba y cómo se desactiva.

## Crear un módulo

### Opción A: script oficial

```powershell
# Requiere Git Bash, incluido con Git for Windows
& "C:\Program Files\Git\bin\bash.exe" modules/create_module.sh
```

El script crea la estructura base, limpia el historial y configura el repositorio local del módulo.

### Opción B: manual

1. Crea `modules/mod-wowwotlk-<tema>/`.
2. Añade `src/`, `conf/`, `data/sql/world/` y `README.md`.
3. Copia la estructura de `https://github.com/azerothcore/skeleton-module`.
4. Añade el módulo a `.gitignore` solo si no debe versionarse; para módulos propios ya existe la
   excepción `!/modules/mod-*`.
5. Configura y compila con `MODULES=static` para local.

## Versionado de módulos

| Opción | Cuándo usarla | Ventaja | Riesgo |
| --- | --- | --- | --- |
| Repo principal | Módulos pequeños y propios | Un clon, commits atómicos | Mezcla historial |
| Repo separado | Módulos grandes o reutilizables | Historia y releases propias | Clonar en modules/ |

Para este proyecto se prefiere el repo principal mientras el módulo sea personal y pequeño. Si un
módulo se vuelve reutilizable, se extrae a su propio repositorio y se documenta la dependencia.

## Cuándo tocar el núcleo

Un cambio de núcleo es aceptable solo si cumple todo esto:

- No existe un hook, evento, script base o dato que resuelva el caso.
- El cambio es pequeño, localizado y con tests o prueba manual clara.
- Está justificado en `DECISIONS.md`.
- Se documenta el motivo y el impacto en actualizaciones.
- Se mantiene el estilo y las guías de `.agents/docs/cpp-guidelines.md`.

Si el cambio es grande, primero se propone un módulo o se busca una solución upstream.

## SQL de módulos y contenido

- Cambios de mundo propios: `data/sql/updates/pending_db_world/`.
- SQL de un módulo: `modules/<modulo>/data/sql/world/` o el directorio que documente su README.
- Nunca editar `data/sql/base/`, `data/sql/archive/` ni `data/sql/updates/db_*/`.
- Un archivo SQL debe ser idempotente cuando sea posible y llevar fecha/nombre descriptivo.
- Antes de commitear, ejecutar la validación de SQL de AzerothCore si está disponible.

## Pruebas de un módulo

1. Compilar con `MODULES=static`.
2. Copiar `conf/mod_*.conf.dist` a `configs/modules/` y renombrar sin `.dist`.
3. Arrancar `worldserver` y revisar el log de carga del módulo.
4. Probar el comportamiento en local.
5. Desactivar el módulo y verificar que el servidor sigue arrancando.
6. Documentar los pasos en el README del módulo.

## Checklist antes de integrar

- [ ] El módulo no requiere cambios en el núcleo, o están justificados en un ADR.
- [ ] La configuración tiene valores por defecto seguros.
- [ ] El SQL está en el directorio correcto y es reproducible.
- [ ] El módulo se puede desactivar sin romper el arranque.
- [ ] README actualizado.
- [ ] Probado en local con `RelWithDebInfo`.
- [ ] Sin secretos, binarios ni datos del cliente.
