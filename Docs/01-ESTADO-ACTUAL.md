# 01 — ESTADO ACTUAL

*Fotografía vigente: qué está hecho, qué está pendiente, en qué día del plan estamos. Se actualiza al cierre de cada sesión.*

*Última actualización: 2026-05-19 (Día 1 del Plan Fase 1, parcial — Git cerrado, faltan plugins y hosting).*

---

## En una línea

**Día 1 de Fase 1 en curso.** WordPress local funcionando en `http://localhost/falta-uno/` con DB `falta_uno`. Falta el resto del setup (Git, plugins base, plugin custom + theme, hosting).

---

## Plan Fase 1 (13 al 22 de mayo)

| Día | Fecha | Tarea | Estado | Notas |
|---|---|---|---|---|
| 1 | 13 May | Setup hosting + WordPress + SSL + Git repo + ACF Pro | 🟡 En curso | WP local OK. Git, hosting, SSL y ACF Pro pendientes. Ver detalle abajo. |
| 2 | 14 May | Bootstrap del plugin: activación, autoloader, CPTs canchas y reservas | ⚪ Pendiente | Esperando cerrar Día 1 |
| 3 | 15 May | Roles custom + todos los ACF fields para canchas y reservas | ⚪ Pendiente | |
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

- [ ] **Cleanup DB:** dropear la DB vieja `faltauno` desde phpMyAdmin (sobra ahora que `falta_uno` es la real).
- [ ] **Plugins base de WP:**
  - [ ] Instalar y activar **ACF Pro** (licencia a gestionar — Manu).
  - [ ] Instalar y activar **WP Mail SMTP** (o equivalente).
- [ ] **Hosting de producción:** decidir proveedor (SiteGround / Cloudways / Hostinger) y dominio. *Diferible — no bloquea Días 2-9.*
- [ ] **SSL:** depende del hosting. *Diferible.*
- [ ] **Custom instructions del proyecto Cowork:** confirmar que las que tenés cargadas dicen "leé `00-MAESTRO`" (lo dicen). Si querés actualizarlas con la versión más completa ver final de esta sesión.

---

## Decisiones tomadas en esta sesión (2026-05-19)

1. **Naming convention:** `falta-uno` (slugs), `fu_` (funciones), `FU_` (clases), `wp_falta_uno_slots` (tabla), `falta_uno` (DB). Detalle: `00b-MAESTRO-HISTORIAL.md §22.1`.
2. **Estilos:** Bootstrap 5 utilities only para MVP. Detalle: `§22.2`.
3. **Carpeta del workspace de docs:** se mantiene `C:\Proyectos\Falta Uno\` (con espacio).
4. **Material viejo en `futbol/`:** queda como referencia histórica, no entra al canon. La intención es migrar lo conceptualmente útil a WordPress + Bootstrap.
5. **Estructura de repos Git:** **dos repos separados** (`falta-uno` para código, `falta-uno-docs` para docs), con allowlist en el `.gitignore` del repo de código y meta-docs como snapshot en el repo de docs. Detalle: `§22.3`. (Cierra la decisión diferida #4 que estaba abierta.)

## Decisiones diferidas (a cerrar pronto)

1. **Idioma del código y de los datos.** El briefing mezcla nombres en español (`cancha_direccion`, `reserva_estado`) y los hooks de WP están en inglés. Convención sugerida: **datos en español, hooks/APIs WP en inglés (porque vienen así de WP)**. A registrar en `02-GUIAS-TECNICAS.md` cuando se confirme.
2. **Branding mínimo:** color primario, logo provisorio, dominio definitivo. Aunque sea placeholder, fijarlo evita rehacer el theme dos veces.
3. **Hosting de producción.**

---

## Próxima sesión

**Cerrar completamente el Día 1 antes de tocar código.** Plan acordado con Manu el 2026-05-19. Checklist en orden:

1. **Cleanup DB:** dropear `faltauno` vieja desde phpMyAdmin (Manu lo puede hacer antes de la sesión).
2. **ACF Pro:** Manu trae el ZIP de la licencia → instalar y activar en el WP local. Verificar que las opciones de "Custom Fields" aparezcan en el admin.
3. **SMTP plugin:** instalar WP Mail SMTP, dejarlo configurado para enviar por Gmail (o equivalente) — todavía sin credenciales reales, solo el plugin listo para configurar.
4. **Decisión diferida — idioma del código/datos.** Confirmar la sugerencia: datos y meta keys en español, hooks/APIs WP en inglés. Registrar en `02-GUIAS-TECNICAS.md §5`.

Cuando esos 4 puntos estén cerrados, el Día 1 queda 🟢 y arrancamos el **Día 2 (bootstrap del plugin)** en la sesión siguiente: crear `falta-uno.php` con header WP + clase `FU_Plugin` + autoloader + activación con CPTs (`canchas`, `reservas`), todo validado con `php -l`. Primer feature del Día 2 inaugura la convención de tags: `fu-plugin-v0.1.0` al cierre del bootstrap.
