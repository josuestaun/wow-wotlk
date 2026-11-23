# Roadmap

El roadmap se organiza por fases con un criterio de aceptación verificable. Una fase no se cierra
hasta que su criterio se cumple y queda registrado en el commit o PR correspondiente.

## Fase 0 — Fundación (completada)

Objetivo: dejar el repositorio preparado para trabajar con agentes y GitFlow sin tocar la lógica del
juego.

- [x] Documentación de arquitectura, instalación, roadmap y decisiones.
- [x] `AGENTS.md` adaptado sin eliminar las reglas de AzerothCore.
- [x] Configuración de OpenCode con modelo gratuito y permisos seguros.
- [x] GitFlow ligero y Conventional Commits documentados.
- [x] Estrategia de módulos y mínimo impacto en el núcleo.
- [x] Reglas de higiene del repositorio y `.gitignore` ampliado.
- [x] Revisión del usuario y publicación de `develop` en `origin`.
- [x] Autonomía Git rutinaria y límites de seguridad documentados.

Criterio de aceptación: el repositorio tiene documentación navegable, reglas de agente claras y un
flujo de ramas/commits/PR definido.

## Fase 1 — Servidor local MVP (actual)

Objetivo: compilar y ejecutar un servidor local funcional.

Entorno: Docker Desktop + WSL2 (ADR-0015) con almacenamiento en E: (ADR-0016). Procedimiento en
[DOCKER.md](DOCKER.md).

- [x] Decidir el entorno de ejecución y documentarlo (ADR-0015/0016).
- [x] Descargar el instalador oficial de Docker Desktop y crear el `.env` local.
- [ ] Habilitar `SVM Mode` en la BIOS del MSI B450-A PRO y reiniciar.
- [ ] Instalar WSL2 y Docker Desktop con data-root en `E:\Docker\wsl`.
- [ ] Configurar recursos (8 CPUs, 12 GB RAM, 4 GB swap) y verificar el motor.
- [ ] Construir las imágenes con `docker compose build`.
- [ ] Arrancar el stack con `docker compose up -d` (importación y datos de cliente).
- [ ] Crear una cuenta GM y ajustar el realm a `127.0.0.1:8085`.
- [ ] Configurar `realmlist.wtf` del cliente 3.3.5a y entrar al mundo.
- [ ] Verificar movimiento, combate básico y quests.

Criterio de aceptación: un personaje nuevo puede iniciar sesión, moverse y completar una quest sin
errores fatales en los logs.

## Fase 2 — Base operativa

Objetivo: poder actualizar y recuperar el servidor sin miedo.

- [ ] Script de backup de `acore_characters` y `acore_world` fuera del repo.
- [ ] Procedimiento de restauración probado.
- [ ] Procedimiento de actualización desde `upstream` documentado y probado.
- [ ] Configuración local separada por entorno (dev/test) sin secretos versionados.
- [ ] Checklist de smoke test tras cada actualización.
- [ ] Logs y crash dumps revisables; build `RelWithDebInfo` con símbolos.

Criterio de aceptación: se puede actualizar el núcleo, recompilar y restaurar un backup sin pérdida
de personajes.

## Fase 3 — Módulo propio base

Objetivo: crear la infraestructura de personalización sin parchear el núcleo.

- [ ] Crear `modules/mod-wowwotlk-core/` con configuración propia.
- [ ] Definir un hook o comando de prueba que demuestre la integración.
- [ ] Añadir SQL del módulo en su propio directorio, no en `data/sql/base`.
- [ ] Documentar build, configuración y desactivación.
- [ ] Añadir prueba manual y, si aplica, unitaria.
- [ ] Verificar que el módulo se puede desactivar sin romper el servidor.

Criterio de aceptación: el módulo se compila, carga, configura y desactiva sin modificar archivos
del núcleo.

## Fase 4 — Primer vertical de gameplay

Objetivo: entregar una personalización completa y acotada de principio a fin.

Elegir **una** de estas líneas:

- [ ] NPC o criatura personalizada con SmartAI, loot y diálogo.
- [ ] Quest propia con objetivos, recompensas y condiciones.
- [ ] Habilidad o hechizo modificado con script de spell.
- [ ] Evento de mundo o instancia con script de mapa.

Para la línea elegida:

- [ ] Documentar el comportamiento esperado y las fuentes.
- [ ] Implementar con la vía de menor impacto (datos > script > módulo > núcleo).
- [ ] Añadir SQL en `data/sql/updates/pending_db_world/` si aplica.
- [ ] Probar en local y registrar evidencia.
- [ ] Actualizar documentación y ADR si hay decisiones nuevas.

Criterio de aceptación: la funcionalidad se puede probar de forma reproducible y no introduce
regresiones conocidas.

## Fase 5 — Pipeline de contenido y base de datos

Objetivo: mantener el contenido personalizado ordenado y reversible.

- [ ] Convención de nombres para SQL pendiente y módulos.
- [ ] Validación de SQL con las herramientas de AzerothCore.
- [ ] Estrategia de rollback para cambios de datos.
- [ ] Separar datos de mundo, auth y personajes.
- [ ] Revisar que no se editan directorios inmutables.

Criterio de aceptación: cada cambio de datos tiene un script reproducible y un plan de reversión.

## Fase 6 — Calidad, rendimiento y seguridad

Objetivo: endurecer el proyecto antes de ampliar el alcance.

- [ ] Unit tests para la lógica propia.
- [ ] Codestyle C++ y SQL en los cambios.
- [ ] Revisión de crashes y fugas con `RelWithDebInfo`.
- [ ] Medición básica de rendimiento en local.
- [ ] Revisión de configuración y permisos.
- [ ] Dependencias y actualizaciones de seguridad al día.

Criterio de aceptación: los cambios propios pasan las comprobaciones del repositorio y no degradan
el arranque ni el login.

## Fase 7 — Primera release

Objetivo: congelar una versión estable y reproducible.

- [ ] `develop` estable y probado.
- [ ] PR de `develop` a `master` (con autorización).
- [ ] Tag `v0.1.0` y notas de release (con autorización).
- [ ] Backup verificado.
- [ ] Documentación actualizada.

Criterio de aceptación: `master` en `v0.1.0` arranca desde cero siguiendo `DOCKER.md`.

## Backlog de ideas

- NPC y criaturas personalizadas.
- Quests y cadenas de misiones propias.
- Ajustes de spells, talentos y cooldowns.
- Loot, profesiones y economía.
- Eventos de mundo y estacionales.
- Comandos GM y herramientas de administración.
- Paneles de configuración del módulo.
- Integración con herramientas de análisis de logs.

## Fuera de alcance

- Servidor público o monetizado.
- Modificación o distribución del cliente.
- Assets propietarios de Blizzard.
- Reescrituras masivas del núcleo.
- Exponer el servidor a Internet o abrir puertos en el router.
- Dependencias de pago o credenciales en el repositorio.
