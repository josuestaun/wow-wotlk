# Instalación y build en Windows nativo

> Ruta alternativa. El entorno elegido del proyecto es Docker Desktop + WSL2; consulta
> [DOCKER.md](DOCKER.md) primero. Esta guía se mantiene para quien prefiera Windows nativo.

Esta guía cubre Windows 10/11 x64 con PowerShell 5.1, Visual Studio 2022 y VS Code. No usa Docker,
WSL2 ni Git Bash para compilar. Git Bash solo es necesario si se ejecuta el generador de módulos
`modules/create_module.sh`.

## 1. Requisitos

| Herramienta | Versión mínima | Notas |
| --- | --- | --- |
| Windows | 10 u 11 x64 | PowerShell 5.1 incluido |
| Visual Studio Community | 2022 (17.x) | Carga de trabajo "Desarrollo para el escritorio con C++" |
| CMake | 3.16 | Instalar la última versión estable x64 |
| MySQL Server | 8.4 LTS | **MySQL 26.x no está soportado** |
| OpenSSL | 3.x Win64 | Versión completa, no la "light" |
| Boost | >= 1.78 | Binarios precompilados para MSVC 14.3 x64 (1.87 recomendado) |
| Git | Actual | Incluye Git Bash opcional |
| VS Code | Actual | Extensiones C/C++ y CMake Tools recomendadas |
| Cliente WoW | 3.3.5a | Solo para extraer datos; nunca se versiona |

Opcional: HeidiSQL, DBeaver o MySQL CLI para administrar la base de datos.

## 2. Variables de entorno

Configura estas variables de usuario o sistema. Usa barras normales en las rutas de CMake y no
añadas una barra final.

```powershell
[Environment]::SetEnvironmentVariable("BOOST_ROOT", "C:/local/boost_1_87_0", "User")
[Environment]::SetEnvironmentVariable("OPENSSL_ROOT_DIR", "C:/OpenSSL-Win64", "User")
[Environment]::SetEnvironmentVariable("MYSQL_ROOT_DIR", "C:/Program Files/MySQL/MySQL Server 8.4", "User")
```

Añade también `C:\Program Files\MySQL\MySQL Server 8.4\bin` al `PATH` para poder usar `mysql` y
`mysqldump`. Cierra y vuelve a abrir la terminal después de cambiar variables.

Verificación rápida:

```powershell
cmake --version
git --version
mysql --version
$env:BOOST_ROOT
$env:OPENSSL_ROOT_DIR
```

## 3. Estructura de directorios

| Uso | Ruta de ejemplo | ¿En Git? |
| --- | --- | --- |
| Código | `E:\proyectos\WoW-WotLK` | Sí |
| Build | `E:\proyectos\WoW-WotLK\var\build` | No |
| Instalación | `E:\proyectos\WoW-WotLK\env\dist` | No |
| Datos del cliente | `E:\WoW-Data` | No |
| Configuración local | `E:\proyectos\WoW-WotLK\conf` | No |
| Backups | `E:\WoW-Backups` | No |

No instales el servidor ni la base de datos dentro del repositorio. Si necesitas datos dentro del
árbol, usa `Data/`, que está ignorado.

## 4. Clonar y preparar el repositorio

```powershell
git clone https://github.com/josuestaun/wow-wotlk.git E:\proyectos\WoW-WotLK
Set-Location E:\proyectos\WoW-WotLK
git remote add upstream https://github.com/azerothcore/azerothcore-wotlk.git
git fetch upstream
```

El remoto `origin` es el fork propio; `upstream` es AzerothCore. El agente puede hacer push de ramas
de trabajo a `origin`; los pushes a `master`, tags y releases requieren autorización.

## 5. Configurar con CMake

### Opción A: línea de comandos (recomendada para agentes)

```powershell
cmake -S . -B var/build -G "Visual Studio 17 2022" -A x64 `
  -DCMAKE_INSTALL_PREFIX="E:/proyectos/WoW-WotLK/env/dist" `
  -DTOOLS_BUILD=all `
  -DSCRIPTS=static `
  -DMODULES=static
```

`TOOLS_BUILD=all` compila los extractores. Si no los necesitas en este build, usa `none` y
reconfigura cuando vayas a extraer datos.

### Opción B: CMake GUI

1. Source: `E:\proyectos\WoW-WotLK`.
2. Build: `E:\proyectos\WoW-WotLK\var\build`.
3. Generador: Visual Studio 17 2022, plataforma `x64`.
4. Configura `CMAKE_INSTALL_PREFIX`, `TOOLS_BUILD=all`, `SCRIPTS=static`, `MODULES=static`.
5. `Configure`, revisa que no queden errores en rojo y pulsa `Generate`.

Si CMake no encuentra Boost, OpenSSL o MySQL, revisa las variables de entorno y borra la caché de
`var/build` antes de reconfigurar.

## 6. Compilar

```powershell
cmake --build var/build --config RelWithDebInfo --parallel
```

Para limitar el uso de CPU:

```powershell
cmake --build var/build --config RelWithDebInfo --parallel 8
```

Los binarios quedan en `var/build/bin/RelWithDebInfo/`. La instalación se hace con:

```powershell
cmake --install var/build --config RelWithDebInfo
```

Tras instalar, `authserver.exe`, `worldserver.exe`, las DLLs y la carpeta `configs/` quedan en
`env/dist/`. En builds multi-configuración, `CMAKE_BUILD_TYPE` se ignora; la configuración se elige
con `--config`.

## 7. DLLs necesarias

Copia junto a los ejecutables (o añade al `PATH`) las DLLs que no instala CMake:

| DLL | Origen |
| --- | --- |
| `libmysql.dll` | `C:\Program Files\MySQL\MySQL Server 8.4\lib` |
| `libcrypto-3-x64.dll` | `C:\OpenSSL-Win64\bin` |
| `libssl-3-x64.dll` | `C:\OpenSSL-Win64\bin` |
| `legacy.dll` | `C:\OpenSSL-Win64\bin` |

La versión de `libmysql.dll` debe coincidir con el servidor MySQL. Si actualizas MySQL, recompila o
copia la DLL nueva.

## 8. Extraer datos del cliente

Necesitas un cliente 3.3.5a legal. No copies archivos del cliente al repositorio.

1. Copia a la carpeta del cliente (`wow.exe`):
   - `map_extractor.exe`
   - `vmap4_extractor.exe`
   - `vmap4_assembler.exe`
   - `mmaps_generator.exe`
   - `mmaps-config.yaml`
   - `apps\extractor\extractor.bat`
2. Crea las carpetas `vmaps` y `mmaps` en la carpeta del cliente.
3. Ejecuta `extractor.bat` y extrae `dbc`, `maps`, `vmaps`, `mmaps` y `cameras`.
4. Mueve las carpetas resultantes a `E:\WoW-Data`.
5. Configura `DataDir` en `worldserver.conf`:

```ini
DataDir = "E:/WoW-Data"
```

No interrumpas la extracción de vmaps o mmaps. Si se interrumpe, borra `Buildings/` y vuelve a
empezar.

## 9. Crear la base de datos

Con MySQL en ejecución, ejecuta como `root` el script de creación:

```powershell
cmd /c "mysql -u root -p < data\sql\create\create_mysql.sql"
```

PowerShell 5.1 no admite `<` como redirección; por eso el comando se ejecuta con `cmd /c`.

Crea el usuario `acore` y las bases `acore_auth`, `acore_characters` y `acore_world`. Cambia la
contraseña por defecto si el servidor no es estrictamente local.

El actualizador automático de AzerothCore crea y actualiza las tablas al arrancar. También puedes
importar manualmente los SQL si lo prefieres; en ese caso sigue `data/sql/README.md` y no edites los
directorios inmutables.

## 10. Configurar el servidor

Copia las plantillas y renómbralas sin `.dist`:

```powershell
Copy-Item env\dist\configs\authserver.conf.dist env\dist\configs\authserver.conf
Copy-Item env\dist\configs\worldserver.conf.dist env\dist\configs\worldserver.conf
```

Ajusta en ambos archivos las cadenas de conexión:

```ini
LoginDatabaseInfo     = "127.0.0.1;3306;acore;acore;acore_auth"
WorldDatabaseInfo     = "127.0.0.1;3306;acore;acore;acore_world"
CharacterDatabaseInfo = "127.0.0.1;3306;acore;acore;acore_characters"
```

En `worldserver.conf` ajusta también `DataDir`. Los archivos `conf/*.conf` están ignorados por Git:
nunca los commitees.

## 11. Arrancar y verificar

1. Ejecuta `authserver.exe` y espera a que abra el puerto 3724.
2. Ejecuta `worldserver.exe`; la primera vez aplicará las actualizaciones de base de datos.
3. En la consola de `worldserver`, crea una cuenta:

```text
account create <usuario> <contraseña>
account set gmlevel <usuario> 3 -1
```

4. En el cliente, edita `realmlist.wtf`:

```text
set realmlist 127.0.0.1
```

5. Inicia sesión y crea un personaje.

Checklist mínimo:

- [ ] `authserver` y `worldserver` arrancan sin errores fatales.
- [ ] La base de datos se actualiza sin fallos.
- [ ] `DataDir` contiene `dbc`, `maps`, `vmaps` y `mmaps`.
- [ ] El cliente conecta a `127.0.0.1`.
- [ ] Se puede crear y entrar con un personaje.

## 12. Actualizar el servidor

```powershell
git fetch upstream
git switch master
git merge upstream/master
git switch develop
git merge master
```

Antes de actualizar, respalda `acore_characters`. Después, recompila y arranca para que el
actualizador aplique los cambios SQL. Si hay conflictos, resuélvelos en una rama de integración y
verifica el build antes de continuar.

## 13. Problemas frecuentes

| Síntoma | Causa probable | Solución |
| --- | --- | --- |
| CMake no encuentra Boost | `BOOST_ROOT` ausente o con barras invertidas | Usa `C:/local/boost_1_87_0` |
| Error de OpenSSL | Versión "light" o arquitectura incorrecta | Instala Win64 completo y define `OPENSSL_ROOT_DIR` |
| `libmysql.dll` no encontrada | MySQL no está en `PATH` | Copia la DLL o añade `lib` al `PATH` |
| El mundo no carga mapas | `DataDir` incorrecto o faltan `dbc`/`maps` | Revisa la ruta y vuelve a extraer |
| Puerto ocupado | Otro proceso usa 3724 u 8085 | Detén el proceso o cambia el puerto en la configuración |
| `opencode.ps1` bloqueado | Política de ejecución de PowerShell | Usa `opencode.cmd` (ver `OPENCODE.md`) |
| El build consume demasiada RAM | Paralelismo alto | Usa `--parallel 8` o menos |

## 14. Seguridad y datos

- No ejecutes el servidor como administrador ni MySQL como `root`.
- No abras los puertos a Internet para un servidor de pruebas.
- No commitees `conf/*.conf`, dumps, logs, datos del cliente ni binarios.
- Guarda los backups de `acore_characters` fuera del repositorio.
- Sigue `REPO_HYGIENE.md` antes de cada commit.
