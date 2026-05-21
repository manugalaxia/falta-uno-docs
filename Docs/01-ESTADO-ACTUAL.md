# 01 — ESTADO ACTUAL

*Fotografía vigente: qué está hecho, qué está pendiente, en qué día del plan estamos. Se actualiza al cierre de cada sesión.*

*Última actualización: 2026-05-21 (Día 1 cerrado, housekeeping de docs + adopción del MANUAL rev 2 — ver `§22.5`).*

---

## En una línea

**Día 1 de Fase 1 cerrado (2026-05-19).** WordPress local funcionando en `http://localhost/falta-uno/` con DB `falta_uno`, dos repos Git pusheados a GitHub, ACF descartado.

**Sesión 2026-05-21:** housekeeping de docs — adopción del `MANUAL-TRABAJO-CON-CLAUDE.md` rev 2 + `PROMPT-INICIAL-CLAUDE.md` como contrato operativo, limpieza de inconsistencias residuales (ver `§22.5`). No avanza el plan: el calendario sigue al cerrar la decisión diferida de idioma y arrancar Día 2 (bootstrap del plugin).

---

## Plan Fase 1 (13 al 22 de mayo)

| Día | Fecha | Tarea | Estado | Notas |
|---|---|---|---|---|
| 1 | 13 May | Setup hosting + WordPress + SSL + Git repo | 🟢 Cerrado | WP local OK + Git cerrado con dos repos pusheados a GitHub (§22.3). Hosting y SSL diferidos no-bloqueantes. ACF Pro descartado (§22.4). |
| 2 | 14 May | Bootstrap del plugin: activación, autoloader, CPTs canchas y reservas | ⚪ Pendiente | Esperando cerrar Día 1 |
| 3 | 15 May | Roles custom + meta boxes nativos para canchas y reservas (helper `FU_Meta`) | ⚪ Pendiente | Cambió scope — sin ACF, ver `§22.4` |
| 4 | 16 May | Tabla `wp_falta_uno_slots` con dbDelta + funciones de generación y consulta | ⚪ Pendiente | |
| 5 | 17 May | Tema custom: header, footer, home con buscador, archive-canchas, single-canchas (mobile-first) | ⚪ Pendiente | Bootstrap 5 utilities only |
| 6 | 18 May | Calendario FullCalendar.js en ficha de cancha + endpoint AJAX de disponibilidad | ⚪ Pendiente | |
| 7 | 19 May | Google Maps en home y ficha + formulario de carga de cancha (frontend, sin WP admin) | ⚪ Pendiente | |
| 8 | 20 May | Login/registro custom (sin wp-login.php) + asignación de roles + página `/mi-panel/` | ⚪ Pendiente | |
| 9 | 21 May | Flujo completo de reserva sin pago: elige slot → crea reserva → bloquea slot → email | ⚪ Pendiente | |
| 10 | 22 May | Testing end-to-end + ajustes mobile + SMTP + deploy a producción | ⚪ Pendiente | |

Convención: 🟢 Completo · 🟡 En curso · 🔴 Bloqueado · ⚪ Pendiente

---

## Detalle del Día 1

### Hecho

- [x] XAMPP corriendo en local (Apache + MariaDB).
- [x] WordPress (última estable) descomprimido en `C:\xampp\htdocs\falta-uno\`.
- [x] Base de datos `falta_uno` creada en MariaDB (vía rename indirecto desde `faltauno` original).
- [x] `wp-config.php` configurado con `DB_NAME = 'falta_uno'`, `WP_HOME` y `WP_SITEURL` apuntando a `http://localhost/falta-uno/`.
- [x] Wizard de WP completado, admin user creado, sitio accesible en `http://localhost/falta-uno/wp-admin/`.
- [x] Estructura de carpetas del proyecto (workspace `C:\Proyectos\Falta Uno\` + `Docs\` adentro).
- [x] Siete documentos canon creados con esqueletos sembrados desde briefing.
- [x] Naming convention decidida y registrada (`§22.1` del HISTORIAL).
- [x] Decisión Bootstrap 5 utilities only registrada (`§22.2`).
- [x] **Git — repo del código** inicializado en `C:\xampp\htdocs\falta-uno\` con `.gitignore` por allowlist (solo trackea `wp-content/plugins/falta-uno/` y `wp-content/themes/falta-uno/`). Pusheado a `github.com/manugalaxia/falta-uno` (privado). Detalle: `§22.3`.
- [x] **Git — repo de docs** inicializado en `C:\Proyectos\Falta Uno\` con todos los docs canon + meta-docs (`MANUAL-TRABAJO-CON-CLAUDE.md`, `PROMPT-INICIAL-CLAUDE.md`) + material histórico de `futbol/`. Pusheado a `github.com/manugalaxia/falta-uno-docs` (privado). Detalle: `§22.3`.

### Pendiente para cerrar Día 1

- [x] **Cleanup DB:** DB vieja `faltauno` dropeada desde phpMyAdmin (2026-05-19).
- [ ] **Custom instructions del proyecto Cowork:** confirmar que las que tenés cargadas dicen "leé `00-MAESTRO`" (lo dicen). Si querés actualizarlas con la versión más completa ver final de esta sesión.

### Diferidos no bloqueantes (no impiden arrancar Día 2)

- [ ] **WP Mail SMTP:** instalar y configurar más adelante, antes del Día 9 (flujo de reserva con emails).
- [ ] **Hosting de producción:** decidir proveedor (SiteGround / Cloudways / Hostinger) y dominio. *No bloquea Días 2-9.*
- [ ] **SSL:** depende del hosting.
- ~~Instalar ACF Pro~~ → **Descartado.** Meta fields nativos en el plugin propio. Ver `§22.4`.

---

## Decisiones tomadas en esta sesión (2026-05-21)

1. **Adopción del `MANUAL-TRABAJO-CON-CLAUDE.md` rev 2 (2026-05-20) y `PROMPT-INICIAL-CLAUDE.md` (2026-05-20) como contrato operativo vigente de Falta Uno.** Ambos meta-docs commiteados al repo `falta-uno-docs` como baseline de la sesión.
2. **Reescritura de `00-ARRANQUE.md`** alineado al MANUAL: reglas "un comando Git por mensaje", "commit baseline antes de tocar varios archivos", "sigue mal x2 → PARÁ", edición directa de docs (no resúmenes pegables), sección nueva "Trampas del entorno conocidas" con FUSE/autocrlf/editor-trunca.
3. **Limpieza de cinco inconsistencias ACF Pro** en `01-ESTADO-ACTUAL.md`, `02-GUIAS-TECNICAS.md` y `03-INVENTARIO-TECNICO.md` (decisión §22.4 propagada al resto de los docs canon).
4. **Cierre formal de la decisión diferida "repos Git"** en `03-INVENTARIO-TECNICO.md` (ya estaba decidido en §22.3, faltaba el reflejo en el inventario).

Detalle completo: `00b-MAESTRO-HISTORIAL.md §22.5`.

## Decisiones tomadas en esta sesión (2026-05-19)

1. **Naming convention:** `falta-uno` (slugs), `fu_` (funciones), `FU_` (clases), `wp_falta_uno_slots` (tabla), `falta_uno` (DB). Detalle: `00b-MAESTRO-HISTORIAL.md §22.1`.
2. **Estilos:** Bootstrap 5 utilities only para MVP. Detalle: `§22.2`.
3. **Carpeta del workspace de docs:** se mantiene `C:\Proyectos\Falta Uno\` (con espacio).
4. **Material viejo en `futbol/`:** queda como referencia histórica, no entra al canon. La intención es migrar lo conceptualmente útil a WordPress + Bootstrap.
5. **Estructura de repos Git:** **dos repos separados** (`falta-uno` para código, `falta-uno-docs` para docs), con allowlist en el `.gitignore` del repo de código y meta-docs como snapshot en el repo de docs. Detalle: `§22.3`. (Cierra la decisión diferida #4 que estaba abierta.)
6. **Sin ACF Pro:** meta fields nativos (`add_meta_box` + `*_post_meta`) dentro del plugin `falta-uno`. Detalle: `§22.4`.

## Decisiones diferidas (a cerrar pronto)

1. **Idioma del código y de los datos.** El briefing mezcla nombres en español (`cancha_direccion`, `reserva_estado`) y los hooks de WP están en inglés. Convención sugerida: **datos en español, hooks/APIs WP en inglés (porque vienen así de WP)**. A registrar en `02-GUIAS-TECNICAS.md` cuando se confirme.
2. **Branding mínimo:** color primario, logo provisorio, dominio definitivo. Aunque sea placeholder, fijarlo evita rehacer el theme dos veces.
3. **Hosting de producción.**

---

## Próxima sesión

Con Día 1 cerrado y la adopción del MANUAL rev 2 propagada a los docs canon, lo pendiente real antes de arrancar Día 2 es:

1. **Manu actualiza las custom instructions del proyecto Cowork desde la UI** con el bloque del `MANUAL §1.1` — apuntar a `00-ARRANQUE.md` y *"actualizá los documentos vos directamente — no pases resúmenes"*. Sin esto, cada sesión nueva arranca con el contrato viejo cargado por Cowork.
2. **Decisión diferida — idioma del código/datos.** Confirmar la sugerencia: datos y meta keys en español con prefijo `fu_` (`fu_cancha_direccion`, `fu_reserva_estado`), hooks/APIs WP en inglés. Registrar en `02-GUIAS-TECNICAS.md §5`. Pasada esta decisión, alinear `00-MAESTRO.md §2.2` que todavía lista las meta keys sin prefijo (pendiente registrado en `§22.5`).

Con eso cerrado arrancamos el **Día 2 (bootstrap del plugin)**: crear `wp-content/plugins/falta-uno/falta-uno.php` con header WP + clase `FU_Plugin` + autoloader + activación con CPTs (`canchas`, `reservas`), todo validado con `php -l`. Primer feature del Día 2 inaugura la convención de tags: `fu-plugin-v0.1.0` al cierre del bootstrap.

SMTP (diferido) se ataca antes del Día 9 (flujo de reserva con emails).
