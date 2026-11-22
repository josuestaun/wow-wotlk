# Higiene y seguridad del repositorio

El repositorio contiene código, documentación y módulos propios. No contiene secretos, binarios,
bases de datos locales, datos persistentes, archivos temporales ni recursos propietarios del
cliente.

## Qué no se versiona

### Credenciales y secretos

- `.env`, `.env.*` y cualquier variante local.
- `conf/*.conf` (solo se versionan las plantillas `conf/dist/`).
- Contraseñas de MySQL, tokens de API, claves SSH, certificados privados.
- `*.key`, `*.pem`, `*.pfx`, `*.p12`.
- Cualquier archivo con `password`, `secret`, `token` o `api_key` en el nombre.

Las credenciales de OpenCode, si se usan, se guardan en la configuración global del usuario, nunca
en este repositorio.

### Binarios y artefactos de build

- `build/`, `build-*/`, `out/`, `var/`, `env/dist/`, `env/user/`.
- `*.exe`, `*.dll`, `*.pdb`, `*.lib`, `*.obj`, `*.ilk`, `*.exp`.
- `compile_commands.json` generado, cachés de CMake y ccache.
- Logs y crash dumps.

Los binarios necesarios se copian desde MySQL/OpenSSL o se generan con el build; no se commitean.

### Bases de datos y datos persistentes

- Dumps `*.sql.gz`, `*.sql.bz2`, `*.sql.zip`, `*.dump`.
- Copias `*.bak` de bases de datos.
- Directorios de datos de MySQL.
- `acore_characters` es estado vivo: se respalda fuera del repo antes de actualizar.
- `data/sql/custom/` está reservado para datos locales y está ignorado.
- `src/server/scripts/Custom/` está reservado para scripts locales y está ignorado.

### Datos del cliente y recursos propietarios

- `Data/` y cualquier carpeta con `dbc`, `maps`, `vmaps`, `mmaps`, `cameras`.
- `*.mpq`, `*.MPQ`, `*.dbc`, `*.db2`, `*.wdt`, `*.adt`, `*.wmo`, `*.m2`, `*.skin`, `*.anim`, `*.blp`.
- Binarios del cliente, addons copiados y capturas con contenido propietario.
- Cualquier asset extraído de World of Warcraft.

Usa un cliente 3.3.5a legal y guarda los datos extraídos fuera del repositorio, por ejemplo en
`E:\WoW-Data`.

### Temporales y ruido

- `*.tmp`, `*.swp`, `*.bak`, `*.orig`, `*.rej`, `*~`.
- `.cache`, `__pycache__`, `node_modules`, `.idea`, `.vs`, `cmake-build-*`.
- Archivos de sistema y de editor que no aportan al proyecto.

## Reglas de `.gitignore`

- No se debilita ni se elimina una regla upstream sin justificarlo.
- Las reglas nuevas van en la sección `CUSTOM` al final de `.gitignore`.
- Los módulos propios se re-incluyen con `!/modules/mod-*/`; solo se versiona su código fuente,
  configuración de ejemplo y SQL, nunca su build.
- Si una ruta local debe estar dentro del árbol, se ignora explícitamente y se documenta.
- `.gitkeep` se usa para conservar directorios vacíos necesarios.

## Comprobaciones antes de commitear

```powershell
git status --short
git diff --cached --stat
git diff --check
git ls-files | Select-String -Pattern '\.(exe|dll|pdb|lib|mpq|MPQ|dbc|sql\.gz|dump)$'
git grep -n -I -E "password|passwd|secret|api[_-]?key|token" -- "*.conf" "*.json" "*.yml"
```

Revisa el resultado del último comando: puede haber falsos positivos en documentación o plantillas,
pero no debe haber credenciales reales.

## Si se filtra un secreto

1. Detén el trabajo y no hagas push adicional.
2. Rota la credencial en el servicio correspondiente.
3. Elimina el secreto del árbol de trabajo y del índice.
4. Si ya está en el historial, planifica una limpieza con `git filter-repo` o BFG **con backup** y
   autorización explícita; un force push solo se hace con permiso del usuario.
5. Registra el incidente y la medida tomada en `DECISIONS.md` si afecta al flujo.

## Backups

- Respalda `acore_characters` antes de actualizar el núcleo, cambiar SQL o experimentar.
- Guarda los backups fuera del repositorio, por ejemplo `E:\WoW-Backups`.
- Verifica que el backup se puede restaurar, no solo que se creó.
- No commitees el backup ni lo subas a servicios públicos.

## Archivos grandes

No se usa Git LFS. Si un archivo supera unos pocos MB, pregunta si realmente debe versionarse. Los
datos del cliente, bases de datos y binarios nunca deben entrar al historial.

## Configuración local

- `conf/authserver.conf` y `conf/worldserver.conf` son locales.
- `DataDir` apunta fuera del repo.
- Las rutas absolutas de tu máquina no se commitean.
- Si necesitas compartir una configuración, comparte una plantilla sin secretos en `docs/project/` o
  en `conf/dist/`, nunca tu archivo local.

## Checklist rápido

- [ ] `git status` solo muestra archivos intencionados.
- [ ] No hay secretos ni credenciales.
- [ ] No hay binarios, dumps, logs ni datos del cliente.
- [ ] `git diff --check` no reporta espacios ni finales de línea.
- [ ] Los archivos nuevos están en la ruta correcta y con el nombre correcto.
- [ ] El commit sigue `COMMITS.md`.
