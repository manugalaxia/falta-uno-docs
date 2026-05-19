# 00b — HISTORIAL — Falta Uno (§22)

*Registro de aprendizajes, decisiones tomadas y decisiones revertidas. Cada entrada es §22.N — donde N es un número secuencial que no se reusa. Si una decisión se revierte, se agrega una entrada nueva que la apunta, no se reescribe la original.*

*Formato de cada entrada:*

- *Fecha (YYYY-MM-DD)*
- *Caso concreto (qué pasó / qué se discutió)*
- *Decisión / regla derivada*
- *Por qué (justificación)*
- *Cómo aplica a futuro*

---

## §22.1 — 2026-05-19 — Naming convention: `falta-uno`

**Caso:** Al armar la estructura inicial del proyecto, surge ambigüedad entre:

- El nombre comercial *Falta Uno* (dos palabras con espacio).
- La carpeta XAMPP que Manu había creado como `faltauno` (todo junto).
- El briefing técnico de Franco que usa `fulboapp` en todos lados (`fulboapp.php`, `wp_fulboapp_slots`, etc.) — nombre de proyecto anterior, descartado.
- Tres alternativas viables: `falta-uno` (con guión), `faltauno` (todo junto), `falta_uno` (con underscore).

**Decisión:** Adoptar la convención WordPress estándar para nombres multi-palabra:

- **Slugs y carpetas:** `falta-uno` (con guión). Plugin, theme, repos.
- **Funciones y opciones WP:** prefijo `fu_` (corto, fácil grep).
- **Clases:** prefijo `FU_`.
- **Tabla custom y nombre de DB:** `wp_falta_uno_slots` y `falta_uno` (con underscore — MySQL no admite guión en nombres de tabla/DB sin escapado).

Detalle completo en `00-MAESTRO.md §2.6`.

**Por qué:**

1. Es la convención del ecosistema WordPress para nombres multi-palabra (Yoast `wordpress-seo` con prefijo `wpseo_`, Contact Form 7 `contact-form-7` con `wpcf7_`). Cualquier dev que tome el proyecto en el futuro espera ver esta forma.
2. `falta-uno` es más legible que `faltauno` cuando crece el código (`fu_get_slots()` vs `faltaunogetslots()`).
3. MySQL impone underscore en nombres de tabla/DB — ese pedazo no era opcional. Mantener guión en slugs y underscore en tablas es la práctica estándar (lo hace el propio WordPress: tabla `wp_options`, slug `wordpress`).

**Cómo aplica a futuro:**

- Todo archivo, función, clase, tabla, opción, slug y repo nuevo respeta esta convención.
- Si en el futuro aparece un nombre nuevo en el dominio (ej. un módulo "torneos"), seguir el mismo patrón: slug `falta-uno-torneos`, prefijo función `fu_torneos_*`, clase `FU_Torneos_*`, tabla `wp_falta_uno_torneos`.
- **No revertir esta decisión sin discusión explícita.** Renombrar todo después cuesta horas y arriesga regresiones.

**Friction colateral cerrado en la misma sesión:**

- Carpeta XAMPP renombrada `faltauno` → `falta-uno`.
- DB MySQL renombrada `faltauno` → `falta_uno` (vía copy + drop, porque el rename directo de phpMyAdmin tiró error #1030 "Read page with wrong checksum" sobre Aria — corrupción menor en `mysql.db`, no relacionada con Falta Uno).
- `wp-config.php` actualizado con nuevo `DB_NAME` y `WP_HOME` / `WP_SITEURL` definidos en código (override del valor en DB, más limpio que UPDATE SQL).

---

## §22.2 — 2026-05-19 — Estilos: Bootstrap 5 utilities only

**Caso:** Al definir el stack visual del MVP, hay tensión entre diseño custom (más trabajo, identidad propia) y framework (más rápido, menos personalizado).

**Decisión:** Para el MVP, usar **Bootstrap 5 utilities only**. Sin SCSS custom, sin overrides. Si algo se ve feo, se resuelve con clases utility de Bootstrap. Cero CSS propio en el theme salvo lo mínimo necesario para FullCalendar y Google Maps.

**Por qué:**

1. MVP el 30 de junio — el tiempo no alcanza para diseño custom.
2. Bootstrap utility-first cubre el 95% de los casos visuales sin escribir CSS.
3. Cuando llegue el momento de tener identidad propia, se hace una pasada de branding sobre algo que ya funciona. Mejor que retrasar el lanzamiento por un diseño que después igual va a cambiar.

**Cómo aplica a futuro:**

- Si un dev sugiere "agreguemos un .css custom para X", primero buscar la utility de Bootstrap que resuelve eso.
- Si una utility de Bootstrap no existe para el caso, considerar si el caso justifica un CSS propio o si se puede resolver con composición de utilities.
- Esta decisión es **explícitamente temporal hasta el MVP**. Post-MVP se evalúa si seguir así o invertir en diseño custom.

---

*Próxima entrada: §22.3 (pendiente — todavía no surgió).*
