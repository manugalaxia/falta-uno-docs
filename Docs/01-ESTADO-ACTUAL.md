# 01 — ESTADO ACTUAL

*Fotografía vigente: qué está hecho, qué está pendiente, en qué día del plan estamos. Se actualiza al cierre de cada sesión.*

*Última actualización: 2026-05-23 (Día 2 parcial + Día 5 cerrados: plugin bootstrap mínimo + theme completo — ver `§22.8`).*

---

## En una línea

**Días 1, 2 (parcial) y 5 cerrados (2026-05-23).** WordPress local OK + plugin `falta-uno` v0.1.0 (CPTs `canchas` y `reservas` vacíos) + theme `falta-uno` v0.1.0 con 11 templates en Bootstrap utilities (verde primario, admin bar oculta). Sitio navegable en `http://localhost/falta-uno/` con 4 canchas dummy cargadas. Repo de código en `a943c0d`, tags `fu-plugin-v0.1.0` y `fu-theme-v0.1.0` pusheados.

**Sesión 2026-05-23:** salto del plan — Manu pidió arrancar por el theme. Antes, scaffolding mínimo del plugin (CPTs vacíos, sin meta boxes ni tabla todavía) para que el theme tuviera contra qué renderizar. Theme completo en una sola sesión. Decisión y detalle en `§22.8`.

---

## Plan Fase 1 (13 al 22 de mayo)

| Día | Fecha | Tarea | Estado | Notas |
|---|---|---|---|---|
| 1 | 13 May | Setup hosting + WordPress + SSL + Git repo | 🟢 Cerrado | WP local OK + Git cerrado con dos repos pusheados a GitHub (§22.3). Hosting y SSL diferidos no-bloqueantes. ACF Pro descartado (§22.4). |
| 2 | 14 May | Bootstrap del plugin: activación, autoloader, CPTs canchas y reservas | 🟡 Parcial | CPTs `canchas` (público) y `reservas` (interno) registrados con clase singleton `FU_Plugin`. Sin autoloader ni clases adicionales (no requeridas hoy). Detalle: `§22.8`. Resto cae en Día 3-4. |
| 3 | 15 May | Roles custom + meta boxes nativos para canchas y reservas (helper `FU_Meta`) | ⚪ Pendiente | Cambió scope — sin ACF, ver `§22.4`. Bloqueante de filtros reales de tipo en archive-canchas (hoy son UI placeholder, ver `§22.8`). |
| 4 | 16 May | Tabla `wp_falta_uno_slots` con dbDelta + funciones de generación y consulta | ⚪ Pendiente | Bloqueante del calendario real en single-canchas (hoy contenedor vacío con placeholder). |
| 5 | 17 May | Tema custom: header, footer, home con buscador, archive-canchas, single-canchas (mobile-first) | 🟢 Cerrado | Adelantado a la sesión 2026-05-23. 11 templates + override del primario verde como excepción CSS (§22.8). Detalle de archivos en `03-INVENTARIO-TECNICO.md`. |
| 6 | 18 May | Calendario FullCalendar.js en ficha de cancha + endpoint AJAX de disponibilidad | ⚪ Pendiente | |
| 7 | 19 May | Google Maps en home y ficha + formulario de carga de cancha (frontend, sin WP admin) | ⚪ Pendiente | |
| 8 | 20 May | Login/registro custom (sin wp-login.php) + asignación de roles + página `/mi-panel/` | ⚪ Pendiente | Los page templates HTML ya están (page-login.php, page-registro.php, page-mi-panel.php). Falta el handler PHP del POST + `wp_signon()` + `wp_create_user()` + `add_role()` según `fu_rol`. |
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

## Decisiones tomadas en esta sesión (2026-05-23)

1. **Salto del plan Día 2 → Día 5 con scaffolding mínimo intermedio (`§22.8`).** Manu pidió arrancar por el theme. Antes del theme se hizo un mini-bootstrap del plugin (solo los CPTs `canchas` y `reservas` vacíos, sin meta boxes ni tabla custom) porque sin CPT registrado los templates `archive-canchas.php` y `single-canchas.php` son código muerto. Plugin v0.1.0 + theme v0.1.0 cerrados en la misma sesión.
2. **Excepción CSS aceptada para el override del primario de Bootstrap (`§22.8`).** El `style.css` del theme tiene ~30 líneas que sobreescriben `--bs-primary` al verde `#00c850` y derivados puntuales en `.btn-primary` / `.btn-outline-primary` y color de links. Equiparado con las excepciones de FullCalendar y Google Maps de `§22.2`. Cualquier otro CSS custom sigue requiriendo justificación en una §22.X.
3. **Admin bar de WP oculta en el frontend para todos los usuarios.** Decisión de producto de Manu — sitio público limpio sin la barra negra. El admin sigue accediendo al panel yendo directo a `/wp-admin/`. Implementado en `functions.php` del theme con `add_filter( 'show_admin_bar', '__return_false' )`.
4. **Filtros de tipo en archive y home son UI placeholder hasta Día 3.** Los chips de Fútbol 5/7/11 cambian el query string pero no filtran porque falta el meta box que pueble `fu_cancha_tipo`. Comentario `// TODO: conectar Día 3` queda en el código.

Detalle completo: `00b-MAESTRO-HISTORIAL.md §22.8`.

## Decisiones tomadas en esta sesión (2026-05-21)

1. **Adopción del `MANUAL-TRABAJO-CON-CLAUDE.md` rev 2 (2026-05-20) y `PROMPT-INICIAL-CLAUDE.md` (2026-05-20) como contrato operativo vigente de Falta Uno.** Ambos meta-docs commiteados al repo `falta-uno-docs` como baseline de la sesión.
2. **Reescritura de `00-ARRANQUE.md`** alineado al MANUAL: reglas "un comando Git por mensaje", "commit baseline antes de tocar varios archivos", "sigue mal x2 → PARÁ", edición directa de docs (no resúmenes pegables), sección nueva "Trampas del entorno conocidas" con FUSE/autocrlf/editor-trunca.
3. **Limpieza de cinco inconsistencias ACF Pro** en `01-ESTADO-ACTUAL.md`, `02-GUIAS-TECNICAS.md` y `03-INVENTARIO-TECNICO.md` (decisión §22.4 propagada al resto de los docs canon).
4. **Cierre formal de la decisión diferida "repos Git"** en `03-INVENTARIO-TECNICO.md` (ya estaba decidido en §22.3, faltaba el reflejo en el inventario).
5. **Convención de idioma código/datos (`§22.6`):** datos, meta keys, roles, capabilities, hooks/filters/AJAX custom **en español con prefijo `fu_`**; hooks y funciones nativas de WP **en inglés** porque vienen así. Formalizada en `02-GUIAS-TECNICAS.md §5` con tabla por categoría. Alineado `00-MAESTRO.md §2.2` (meta keys de `canchas` y `reservas` ahora llevan prefijo `fu_`) y `§2.4` (capabilities `fu_crear_reservas`, `fu_gestionar_canchas_propias`, `fu_ver_reservas_propias`). Cierra la decisión diferida #1.
6. **Corrección del equipo del proyecto (`§22.7`):** salió el dato falso de "dev part-time Franco" que venía del bootstrap inicial. Equipo real = dos socios (Manu + José, ambos no-devs) + Claude como asistente técnico que escribe el código. Reescrita la sección "Operador" del `00-ARRANQUE.md` y limpiadas las menciones a Franco en `00-MAESTRO.md §3` y `00b-HISTORIAL §22.1 / §22.4`.

Detalle completo: `00b-MAESTRO-HISTORIAL.md §22.5`, `§22.6`, `§22.7`.

## Decisiones tomadas en esta sesión (2026-05-19)

1. **Naming convention:** `falta-uno` (slugs), `fu_` (funciones), `FU_` (clases), `wp_falta_uno_slots` (tabla), `falta_uno` (DB). Detalle: `00b-MAESTRO-HISTORIAL.md §22.1`.
2. **Estilos:** Bootstrap 5 utilities only para MVP. Detalle: `§22.2`.
3. **Carpeta del workspace de docs:** se mantiene `C:\Proyectos\Falta Uno\` (con espacio).
4. **Material viejo en `futbol/`:** queda como referencia histórica, no entra al canon. La intención es migrar lo conceptualmente útil a WordPress + Bootstrap.
5. **Estructura de repos Git:** **dos repos separados** (`falta-uno` para código, `falta-uno-docs` para docs), con allowlist en el `.gitignore` del repo de código y meta-docs como snapshot en el repo de docs. Detalle: `§22.3`. (Cierra la decisión diferida #4 que estaba abierta.)
6. **Sin ACF Pro:** meta fields nativos (`add_meta_box` + `*_post_meta`) dentro del plugin `falta-uno`. Detalle: `§22.4`.

## Decisiones diferidas (a cerrar pronto)

1. **Branding mínimo:** color primario, logo provisorio, dominio definitivo. Aunque sea placeholder, fijarlo evita rehacer el theme dos veces.
2. **Hosting de producción.**

*Cerradas en sesiones previas: idioma del código/datos (2026-05-21, §22.6).*

---

## Próxima sesión

Con Día 1, Día 2 (parcial) y Día 5 cerrados, lo que sigue para destrabar los placeholders del theme y avanzar el plan real:

1. **Cerrar Día 3 completo** — los meta boxes son la prioridad alta. Sin ellos: los filtros de tipo en archive y home siguen siendo UI muerta, single-canchas muestra todo en "—" (dirección, precio, teléfono), y no se puede cargar una cancha real con sus datos. Roles `jugador` / `dueno_cancha` con `add_role()` en activación del plugin también van acá.
2. **Cerrar Día 4** — tabla `wp_falta_uno_slots` con `dbDelta()` en activación + funciones de generación y consulta. Bloqueante del calendario real (Día 6).
3. **Día 6** — FullCalendar.js enchufado al `#fu-calendario-cancha` que ya está en `single-canchas.php` + endpoint AJAX `fu_disponibilidad_cancha`.

Diferidas no bloqueantes (siguen abiertas):

- **Branding mínimo:** color primario cerrado (verde `#00c850`), falta logo provisorio y dominio definitivo.
- **Hosting de producción.**

SMTP (diferido) se ataca antes del Día 9 (flujo de reserva con emails).
