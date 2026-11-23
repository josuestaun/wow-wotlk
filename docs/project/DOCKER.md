# Servidor local con Docker Desktop + WSL2

Guía de instalación, configuración y operación del servidor AzerothCore de este fork usando la
infraestructura Docker oficial del repositorio. Sustituye a `INSTALL_WINDOWS.md` como ruta principal;
la instalación nativa queda como alternativa.

- Estado: entorno preparado, pendiente de habilitar la virtualización en la BIOS y reiniciar.
- Decisión: ADR-0015 (Docker Desktop + WSL2) y ADR-0016 (almacenamiento en E: y build reproducible).
- Alcance: servidor local, sin exposición a Internet.

## Arquitectura

- Docker Desktop con backend WSL2 (Windows 11 Home solo admite contenedores Linux).
- `docker-compose.yml` y `apps/docker/*` oficiales de AzerothCore, sin imágenes de terceros.
- Imágenes construidas desde este repositorio con `docker compose build`.
- MySQL 8.4 oficial (`mysql:8.4`) y el importador `ac-db-import`.
- Datos de cliente descargados por el contenedor oficial `ac-client-data-init` desde la fuente que
  usa el instalador de AzerothCore: `wowgaming/client-data` v20.0.
- Puertos publicados solo en `127.0.0.1`; no se abre el router ni se expone el servidor.

## Requisitos y estado del equipo

| Elemento | Valor |
| --- | --- |
| Sistema | Windows 11 Home build 26200 x64 |
| CPU | AMD Ryzen 7 3700X (8 núcleos, 16 hilos, AMD-V/SLAT disponibles) |
| RAM | 16 GB |
| Placa | MSI B450-A PRO (MS-7B86), BIOS AMI A.G3 |
| Disco del proyecto | `E:` con ~745 GB libres |
| Cliente | `C:\Users\josue\Desktop\UltimoWoW esES` (3.3.5a build 12340) |
| Virtualización en firmware | **Deshabilitada**: hay que activar `SVM Mode` en la BIOS |
| WSL2 | No instalado |
| Docker Desktop | No instalado; instalador descargado en `E:\Docker\installer` |

Docker Desktop exige virtualización por hardware. La CPU la soporta, pero la BIOS la tiene apagada.

## 1. Habilitar la virtualización en la BIOS (intervención manual)

Placa MSI B450-A PRO:

1. Reinicia y pulsa `Supr`/`Del` repetidamente para entrar en la BIOS.
2. Pulsa `F7` para pasar a modo avanzado si aparece el modo EZ.
3. Ve a `OC` > `Advanced CPU Configuration` > `SVM Mode` y ponlo en `Enabled`.
4. Si no aparece ahí, busca `Settings` > `Advanced` > `AMD CBS` > `CPU Common Options` >
   `SVM Mode` > `Enabled`.
5. Guarda y sal con `F10` y confirma.

Tras arrancar Windows, verifica:

```powershell
(Get-CimInstance Win32_Processor).VirtualizationFirmwareEnabled
```

Debe devolver `True`. También puedes comprobarlo en Administrador de tareas > Rendimiento > CPU >
Virtualización: `Habilitada`.

## 2. Instalar WSL2 (una vez, requiere administrador)

Abre PowerShell como administrador y ejecuta:

```powershell
wsl --install --no-distribution
wsl --update
wsl --version
```

Si pide reiniciar, reinicia y repite `wsl --update` y `wsl --version`. Si la Microsoft Store está
bloqueada, instala el MSI oficial desde https://github.com/microsoft/WSL/releases.

`wsl --version` debe mostrar una versión 2.x. Docker Desktop requiere WSL 2.1.5 o superior.

## 3. Instalar Docker Desktop (instalación por usuario, sin administrador)

El instalador oficial está descargado en:

```text
E:\Docker\installer\DockerDesktopInstaller.exe
```

Instalación por usuario con el disco de WSL en E::

```powershell
Start-Process "E:\Docker\installer\DockerDesktopInstaller.exe" -Wait -ArgumentList `
  'install','--user','--quiet','--accept-license','--backend=wsl-2',`
  '--wsl-default-data-root=E:\Docker\wsl'
```

Notas:

- `--user` instala en `%LOCALAPPDATA%\Programs\DockerDesktop` y no requiere administrador. Habilitar
  WSL2 por primera vez sí requiere administrador, por eso el paso 2 va antes.
- `--wsl-default-data-root=E:\Docker\wsl` deja el disco de WSL/Docker en E:.
- `--accept-license` evita el diálogo inicial. Revisa la licencia de Docker Desktop; es gratuita para
  uso personal.

Abre Docker Desktop, omite el inicio de sesión si lo permite y espera a que el motor arranque.
Después ajusta en `Settings` > `Resources` > `Advanced`:

| Recurso | Valor recomendado |
| --- | --- |
| CPUs | 8 |
| Memory | 12 GB |
| Swap | 4 GB |
| Disk image location | `E:\Docker\wsl` |

Con 16 GB de RAM, 12 GB para Docker deja margen para Windows. El build usa `nproc + 1` procesos; si
se queda sin memoria, baja las CPUs a 6 o cierra aplicaciones.

Verifica:

```powershell
docker --version
docker compose version
docker info --format '{{.ServerVersion}}'
```

## 4. Configuración local del proyecto (`.env`)

El repositorio ya incluye un `.env` local, ignorado por Git (`/.env*`). No se commitea. Contiene la
contraseña local de MySQL. Plantilla sin secretos:

```dotenv
DOCKER_AC_ENV_FILE=conf/dist/env.ac

DOCKER_VOL_ETC=./env/dist/etc
DOCKER_VOL_LOGS=./env/dist/logs
DOCKER_VOL_DATA=ac-client-data

DOCKER_AUTH_EXTERNAL_PORT=127.0.0.1:3724
DOCKER_WORLD_EXTERNAL_PORT=127.0.0.1:8085
DOCKER_SOAP_EXTERNAL_PORT=127.0.0.1:7878
DOCKER_DB_EXTERNAL_PORT=127.0.0.1:3306

DOCKER_DB_ROOT_PASSWORD=<contraseña local, solo en .env>
DOCKER_DB_IMAGE=mysql:8.4
DOCKER_IMAGE_TAG=local

DOCKER_USER=acore
DOCKER_USER_ID=1000
DOCKER_GROUP_ID=1000
```

Claves:

- Los puertos con prefijo `127.0.0.1:` evitan publicar en la red local. El cliente WoW se conecta a
  `127.0.0.1`.
- `DOCKER_IMAGE_TAG=local` evita usar imágenes precompiladas de AzerothCore por accidente; se usan
  las construidas desde este checkout.
- `DOCKER_DB_IMAGE=mysql:8.4` es la imagen oficial. No se usa la imagen preimportada
  `acore/ac-wotlk-db` para evitar desajustes de esquema con este checkout.
- `DOCKER_VOL_ETC` y `DOCKER_VOL_LOGS` son bind mounts dentro del repo en E:; las bases y los datos
  de cliente son volúmenes nombrados dentro del data-root de Docker en E:.

No uses `DOCKER_USER=root` salvo que aparezca un problema de permisos en los bind mounts.

## 5. Construir y arrancar

Desde `E:\proyectos\WoW-WotLK`:

```powershell
docker compose build
docker compose up -d
```

La primera vez:

- El build compila AzerothCore dentro de Linux; puede tardar entre 20 y 60 minutos.
- `ac-client-data-init` descarga ~1,14 GB de datos de cliente y los extrae en el volumen.
- `ac-db-import` crea e importa `acore_auth`, `acore_world` y `acore_characters` (10-15 minutos).
- Después arrancan `ac-authserver` y `ac-worldserver`.

Sigue el progreso:

```powershell
docker compose ps
docker compose logs -f ac-db-import
docker compose logs -f ac-client-data-init
docker compose logs -f ac-worldserver
```

## 6. Verificación

```powershell
docker compose ps
docker compose logs --tail=100 ac-authserver
docker compose logs --tail=200 ac-worldserver
Test-NetConnection 127.0.0.1 -Port 3724
Test-NetConnection 127.0.0.1 -Port 8085
```

Señales correctas:

- `ac-database` en `healthy`.
- `ac-db-import` y `ac-client-data-init` terminan con código 0.
- `ac-authserver` y `ac-worldserver` en ejecución.
- El log del worldserver muestra la carga de mapas y `World initialized`.
- Los puertos 3724 (auth) y 8085 (world) escuchan en localhost.

Comprueba el realm:

```powershell
docker compose exec ac-database mysql -uroot -p `
  -e "USE acore_auth; SELECT id,name,address,localAddress,port FROM realmlist;"
```

Debe mostrar `address` y `localAddress` en `127.0.0.1` y `port` 8085. Si no:

```sql
USE acore_auth;
UPDATE realmlist SET address='127.0.0.1', localAddress='127.0.0.1', port=8085;
```

## 7. Crear la cuenta de juego

Adjunta la consola del worldserver:

```powershell
docker attach ac-worldserver
```

Dentro:

```text
account create <usuario> <contraseña>
account set gmlevel <usuario> 3 -1
```

Para salir de la consola sin matar el proceso: `Ctrl+P` y luego `Ctrl+Q`. No uses `Ctrl+C`.

Si `docker attach` falla por no ser una TTY, usa Windows Terminal o, en Git Bash,
`winpty docker attach ac-worldserver`. Como alternativa no interactiva:

```powershell
docker compose exec -T ac-worldserver bash -c 'echo "account create <usuario> <contraseña>" > /proc/1/fd/0'
```

## 8. Conectar el cliente WoW 3.3.5a

Cliente detectado: `C:\Users\josue\Desktop\UltimoWoW esES` (build 12340). No se modifica ningún
archivo del cliente sin autorización. Solo se necesita `realmlist.wtf` en la raíz del cliente:

```text
set realmlist 127.0.0.1
```

Procedimiento seguro (haz backup si ya existe):

```powershell
$client = "C:\Users\josue\Desktop\UltimoWoW esES"
Copy-Item "$client\realmlist.wtf" "$client\realmlist.wtf.bak" -ErrorAction SilentlyContinue
Set-Content -LiteralPath "$client\realmlist.wtf" -Value "set realmlist 127.0.0.1" -Encoding ASCII
```

Después inicia `Wow.exe`, entra con la cuenta creada y crea un personaje. No copies datos del
cliente al repositorio ni al servidor: el servidor usa su propia descarga oficial de datos.

## 9. Flujo de desarrollo

El código se compila dentro de las imágenes. Tras cambiar C++, SQL o módulos:

```powershell
# C++ o módulos
docker compose build ac-worldserver ac-authserver
docker compose up -d

# SQL o datos que usa el importador
docker compose build ac-db-import
docker compose up -d
```

Reglas:

- SQL propio de mundo: `data/sql/updates/pending_db_world/`, nunca `base/`, `archive/` ni
  `updates/db_*/`. Ver `.agents/docs/sql-guidelines.md`.
- SQL de un módulo: dentro del módulo, en su carpeta `data/sql/`.
- Módulos: clonar en `modules/`, reconstruir y reiniciar. Se autodescubren si tienen `src/`.
- Configuración: preferir variables `AC_*` en un `docker-compose.override.yml` (ignorado por Git)
  antes que editar archivos dentro del contenedor.
- Logs: `docker compose logs -f ac-worldserver` y `env/dist/logs`.
- Para desarrollo intensivo con bind mounts, el `ac-dev-server` y el devcontainer oficiales usan el
  árbol del repo; en Windows puede ser lento sobre E:. Si molesta, clona el repo dentro de WSL2 y
  trabaja allí, manteniendo los cambios en GitHub.

## 10. Arranque, parada y backups

```powershell
docker compose up -d          # arrancar
docker compose stop           # parar sin borrar contenedores
docker compose restart        # reiniciar
docker compose down           # parar y borrar contenedores, conserva volúmenes
docker compose ps             # estado
```

Nunca uses `docker compose down -v` salvo que quieras borrar las bases de datos y los datos de
cliente. Esa orden destruye los volúmenes.

Backup sin problemas de codificación en PowerShell (el dump se genera dentro del contenedor y se
copia al host):

```powershell
$dump = 'mysqldump -uroot -p"$MYSQL_ROOT_PASSWORD" --single-transaction ' +
        '--routines --triggers acore_characters > /tmp/acore_characters.sql'
docker compose exec -T ac-database sh -c $dump
docker cp ac-database:/tmp/acore_characters.sql "E:\WoW-Backups\acore_characters-$(Get-Date -Format yyyy-MM-dd).sql"
```

Repite con `acore_auth` y `acore_world`. Guarda los backups en `E:\WoW-Backups`, fuera del repo.

## 11. Problemas frecuentes

| Síntoma | Causa probable | Solución |
| --- | --- | --- |
| Docker no arranca y avisa de virtualización | SVM deshabilitado en BIOS | Activa `SVM Mode` y reinicia |
| `wsl --version` falla | WSL2 ausente o antiguo | `wsl --install --no-distribution`, reinicia, `wsl --update` |
| `docker` no se reconoce | Docker Desktop cerrado o fuera de PATH | Abre Docker Desktop y reinicia la terminal |
| `ac-database` no llega a `healthy` | Contraseña distinta | Revisa logs; no borres el volumen sin autorización |
| El build se queda sin memoria | Demasiados trabajos paralelos | Baja CPUs a 6 o cierra aplicaciones |
| `ac-client-data-init` falla | Red o GitHub inaccesible | Reintenta `docker compose up -d ac-client-data-init` |
| El cliente no ve el reino | `realmlist.wtf`, `realmlist.address` o puertos | Revisa los tres y los logs |
| El cliente conecta pero el mundo no carga | Datos de cliente incompletos | Mira los logs del worldserver |
| Permisos en `env/dist/etc` o logs | Bind mount sin escritura | Revisa el aviso del entrypoint; evita root |
| Puerto ocupado | Otro proceso usa 3724/8085/3306/7878 | Cambia el puerto en `.env` o detén el proceso |

## 12. Seguridad

- Publica solo en `127.0.0.1`; no abras puertos en el router ni en el firewall para Internet.
- `.env` contiene la contraseña local y está ignorado por Git. No lo commitees.
- No borres volúmenes ni datos persistentes sin backup y autorización.
- No uses imágenes de terceros; solo `mysql:8.4`, las imágenes construidas desde este repo y los
  datos de cliente de la fuente oficial del instalador de AzerothCore.
- No modifiques el cliente WoW salvo `realmlist.wtf`, y haz backup antes.

## 13. Procedencia y atribución

- `docker-compose.yml`, `apps/docker/Dockerfile` y `apps/docker/*`: AzerothCore (GPL v2).
- Datos de cliente: `https://github.com/wowgaming/client-data` v20.0, la fuente que usa el
  instalador oficial de AzerothCore.
- MySQL: imagen oficial `mysql:8.4`.
- Docker Desktop: instalador oficial de Docker; SHA256 del instalador descargado:
  `98B24FD6F56313899953CA4D2ED05645D69963B598143B75C6585FA4F7F639B2`.
