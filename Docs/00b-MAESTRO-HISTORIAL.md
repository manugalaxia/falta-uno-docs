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

## §22.3 — 2026-05-19 — Setup Git inicial: dos repos + .gitignore por allowlist

**Caso:** Cerrar el ítem "Git" del Día 1 (Plan Fase 1). Tres sub-decisiones quedaron pendientes hasta esta sesión:

1. ¿Un solo repo (código + docs juntos) o dos repos separados? (Decisión diferida del `§22.1`/ESTADO-ACTUAL.)
2. ¿Cómo evitar que el core de WordPress termine commiteado, si el repo vive sobre la instalación XAMPP entera?
3. ¿Dónde van los meta-docs transversales (`MANUAL-TRABAJO-CON-CLAUDE.md`, `PROMPT-INICIAL-CLAUDE.md`) que Manu lleva entre proyectos?

**Decisión:**

1. **Dos repos separados** en GitHub (ambos privados):
   - `manugalaxia/falta-uno` → código (raíz `C:\xampp\htdocs\falta-uno\`).
   - `manugalaxia/falta-uno-docs` → docs (raíz `C:\Proyectos\Falta Uno\`).
2. **Repo de código con `.gitignore` por allowlist:** ignorar `/*` y volver a habilitar selectivamente solo `wp-content/plugins/falta-uno/` y `wp-content/themes/falta-uno/` (+ `.gitignore` y `README.md`). El core de WP, plugins de terceros, themes ajenos, `wp-config.php`, `.env`, `vendor/`, `node_modules/` y logs quedan fuera por construcción, no por listas negras frágiles.
3. **Meta-docs adentro del repo de docs como snapshot.** `MANUAL-TRABAJO-CON-CLAUDE.md` y `PROMPT-INICIAL-CLAUDE.md` viven en la raíz de `C:\Proyectos\Falta Uno\` y entraron al commit inicial de `falta-uno-docs`. Cuando se actualicen en otro proyecto, se copia la versión nueva sobre estos y se commitea.

**Por qué:**

1. **Dos repos:** distinta cadencia (código cambia con cada feature; docs cambia post-sesión), distinto destino (código va a producción; docs no), distinta visibilidad potencial. Alineado con el `MANUAL-TRABAJO-CON-CLAUDE.md §3.1` que Manu trae de Landing+.
2. **Allowlist sobre denylist en el `.gitignore` de código:** un `.gitignore` que enumera "todo lo de WP a excluir" se rompe en cuanto WordPress se actualiza y aparece una carpeta nueva. Con allowlist (`/*` + `!` para lo que sí queremos), el repo solo crece cuando agregamos archivos de **nuestro** código — el core de WP es invisible a Git por defecto.
3. **Meta-docs como snapshot:** la alternativa "fuera de Git, se mueven a mano" pierde trazabilidad — no sabés qué versión del manual estaba vigente cuando se tomó tal decisión. Snapshot adentro del repo de docs es trivial de mantener (copy + commit) y queda atado al estado del proyecto.

**Cómo aplica a futuro:**

- **Cualquier código custom nuevo del MVP** (helpers, scripts de migración, plantillas) vive bajo `wp-content/plugins/falta-uno/` o `wp-content/themes/falta-uno/` para entrar automáticamente al repo. Crear código en otra ruta de la instalación queda explícitamente fuera de Git.
- **Si en algún momento se agrega un segundo plugin custom** (poco probable pre-MVP), sumarlo al `.gitignore` con `!wp-content/plugins/<nombre>/`.
- **`wp-config.php` jamás se commitea.** Si se necesita versionar configuración no-sensible, crear un `wp-config-sample.php` o variables de entorno separadas.
- **Para actualizar los meta-docs:** copiar la versión nueva sobre la actual en `C:\Proyectos\Falta Uno\` y commitear con mensaje tipo `docs: actualizo MANUAL-TRABAJO-CON-CLAUDE al snapshot de YYYY-MM-DD`.

**Estado al cierre:** ambos repos pusheados a GitHub con commit inicial. Repo de código contiene solo `.gitignore` + `README.md` (todavía no existe el plugin ni el theme — eso es Día 2). Repo de docs contiene los 7 docs canon + meta-docs + briefing histórico + material `futbol/`.

---

## §22.4 — 2026-05-19 — Sale ACF Pro: meta fields nativos en el plugin propio

**Caso:** El MAESTRO original (sembrado desde el briefing de Franco) listaba **Advanced Custom Fields (ACF Pro)** como dependencia para los meta fields de los CPTs `canchas` y `reservas` (ver §2.2 del MAESTRO en su versión previa). Al llegar al ítem "instalar ACF Pro" del checklist del Día 1, Manu decide **no** usar ACF y hacer toda la administración de campos dentro del plugin custom `falta-uno`.

**Decisión:** Eliminar la dependencia de ACF (ni Pro ni free). Todos los meta fields de `canchas` y `reservas` se registran y se administran con la API nativa de WordPress dentro del propio plugin:

- `add_meta_box()` para las cajas en el editor de cada CPT.
- `update_post_meta()` / `get_post_meta()` para persistir y leer.
- Render manual de los inputs en PHP (probablemente con un helper `FU_Meta::render_field()` para no repetir markup).
- Sanitización y validación en hooks `save_post_<cpt>`.

El plugin sigue siendo uno solo (`wp-content/plugins/falta-uno/`) y absorbe la responsabilidad que antes tercerizaba ACF.

**Por qué:**

1. **Cero dependencias pagas.** ACF Pro cuesta licencia anual. El MVP arranca sin que ese costo entre antes de validar el negocio.
2. **Control total.** Los meta fields son ~9 en `canchas` y ~9 en `reservas` — bajo volumen, baja complejidad. La API nativa de WP los cubre sin pelearse con ningún plugin de terceros para mover validaciones o lookups.
3. **Menos magia en el frontend.** Los dueños cargan canchas desde un formulario propio (decisión §2.7 del MAESTRO: "los dueños no tocan el WP admin"). Esa UI ya iba a tener que ser custom — usar ACF para el admin y otra cosa para el frontend duplicaba trabajo.
4. **Coherencia con la convención `fu_*`.** Las meta keys quedan `fu_cancha_direccion`, `fu_reserva_estado`, etc. — todo bajo nuestro namespace, sin prefijos ajenos.

**Cómo aplica a futuro:**

- **Cualquier campo nuevo de un CPT existente o nuevo** se agrega como meta field nativo. No se vuelve a discutir ACF.
- **Si en algún momento aparece un caso donde ACF resolvería algo en 10 minutos** (ej. flexible fields, repeater complejo), revisar primero si el caso justifica realmente esa complejidad o si se puede modelar como tabla custom (como ya hicimos con `wp_falta_uno_slots`).
- **El Día 3 del Plan Fase 1 cambia de scope.** Donde decía "Roles custom + todos los ACF fields", ahora es: "Roles custom + meta boxes nativos para `canchas` y `reservas` + helper `FU_Meta`".
- **MAESTRO §2.1 y §2.2 actualizados** en esta misma sesión para reflejar el cambio (sacar fila ACF de la tabla del stack y cambiar el header "Campos ACF" por "Meta fields").

**Estado al cierre:** decisión registrada. MAESTRO actualizado. Día 1 pierde el sub-ítem "Instalar y activar ACF Pro" — queda solo SMTP (diferido por Manu) más el cierre del Día 2 que arranca el plugin.

---

## §22.5 — 2026-05-21 — Adopción del MANUAL rev 2 + PROMPT-INICIAL como contrato operativo

**Caso:** En la sesión del 2026-05-21 Manu trae al workspace de docs dos meta-docs actualizados (`MANUAL-TRABAJO-CON-CLAUDE.md` rev 2 del 2026-05-20 y `PROMPT-INICIAL-CLAUDE.md` del 2026-05-20) consolidados durante las últimas sesiones de Landing+. Los meta-docs traen reglas nuevas que no estaban en `00-ARRANQUE.md` original ni en las custom instructions del proyecto Cowork:

1. **"Un comando Git por mensaje"** — pegar el comando, esperar la salida del operador, recién después el siguiente. No listar 4-5 pasos consecutivos.
2. **"Commit baseline antes de tocar varios archivos seguidos"** — punto de retorno seguro de 30 segundos.
3. **"Si entregás algo dos veces y Manu dice 'sigue mal' → PARÁ"** — releer el código canónico desde cero, no entregar otra fix inmediata.
4. **Edición directa de docs al cierre** — no pasar resúmenes estructurados para que Manu copie y pegue cuando Claude tiene acceso de escritura.
5. **Trampas del entorno conocidas** — sección nueva con cuatro casos: FS de Cowork (FUSE) que puede truncar archivos al editarlos, `git checkout` con `autocrlf=true` que puede fallar silenciosamente, `git status` desde sandbox vs Git Bash con discrepancias de EOL, editores que truncan al guardar.

Además, al revisar la docs vigente aparecieron cinco inconsistencias residuales por ACF Pro (decisión §22.4 ya tomada pero no propagada): `00-ARRANQUE.md` línea 23 del stack, `02-GUIAS-TECNICAS.md §3` fila "Meta key ACF" y §7 del índice "ACF Pro: import/export", `03-INVENTARIO-TECNICO.md` carpeta `advanced-custom-fields-pro/` en el árbol del plugin, `01-ESTADO-ACTUAL.md` título Día 1 "+ ACF Pro". Adicionalmente `03-INVENTARIO-TECNICO.md` mantenía la sección "Repos Git (cuando se inicialicen)" como decisión diferida cuando ya estaba cerrada en `§22.3`.

**Decisión:**

1. **Adoptar el MANUAL rev 2 y el PROMPT-INICIAL como contrato operativo vigente de Falta Uno.** Ambos meta-docs viven en `C:\Proyectos\Falta Uno\` (raíz del workspace de docs, ya commiteados al repo `falta-uno-docs` como baseline de esta sesión). Cuando se actualicen en otro proyecto, se copia la versión nueva sobre estos y se commitea con mensaje tipo `docs: actualizo MANUAL al snapshot YYYY-MM-DD (rev N)`.

2. **Reescritura completa de `00-ARRANQUE.md`** para reflejar el MANUAL: entorno del operador, edición directa también para docs, "un comando por mensaje" + "commit baseline" + "sigue mal x2 → PARÁ" en reglas de comportamiento, sección nueva "Trampas del entorno conocidas" con los cuatro casos del MANUAL §6, checklist de cierre alineado al MANUAL §5.

3. **Limpieza de las cinco inconsistencias ACF Pro** en `01-ESTADO-ACTUAL.md`, `02-GUIAS-TECNICAS.md` y `03-INVENTARIO-TECNICO.md`. Citas legítimas a la decisión §22.4 (contexto histórico) se mantuvieron; menciones residuales que la contradecían se borraron o reescribieron.

4. **Cierre de la decisión diferida "repos Git"** en `03-INVENTARIO-TECNICO.md` — sección reescrita reflejando los dos repos ya inicializados según §22.3 (allowlist en el de código, lista mínima en el de docs).

**Regla derivada:**

- **Cuando un meta-doc transversal (MANUAL, PROMPT-INICIAL) llega actualizado a un proyecto, no se mete en silencio.** Se commitea como baseline aparte, se reescribe `00-ARRANQUE.md` para alinearlo, se suma entrada al HISTORIAL para que la próxima sesión sepa que el contrato cambió.
- **Para editar archivos existentes con caracteres no-ASCII (`§`, tildes, eñes, em-dash) en el sandbox de Cowork: NUNCA `Edit` ni `Write` del File API.** Usar bash con heredoc, `sed -i 's#old#new#'` (separador `#` cuando el contenido tiene `/` o `|`), o `python3` con `os.open + os.write + os.fsync + os.close`. Validar siempre post-write con `wc -c`, `tail -c 300`, `file`, y `tr -d -c '\000' | wc -c`. No usar `grep -q $'\0'` para detectar null bytes — falso positivo conocido.
- **Las custom instructions del proyecto Cowork mandan, pero el MANUAL es la fuente canónica.** Si hay conflicto entre lo que dicen las custom instructions del proyecto y el MANUAL/PROMPT-INICIAL, alinear las custom instructions a la versión del MANUAL — el operador las actualiza desde la UI de Cowork.

**Estado al cierre:**

- Meta-docs (`MANUAL-TRABAJO-CON-CLAUDE.md`, `PROMPT-INICIAL-CLAUDE.md`) commiteados al repo `falta-uno-docs` como baseline de la sesión.
- `00-ARRANQUE.md` reescrito (~12 KB).
- `01-ESTADO-ACTUAL.md`, `02-GUIAS-TECNICAS.md`, `03-INVENTARIO-TECNICO.md` limpiados de inconsistencias.
- Esta entrada §22.5 sumada al HISTORIAL.
- **Pendiente para Manu (no se puede tocar desde acá):** actualizar las custom instructions del proyecto Cowork desde la UI para que digan *"Siempre leé `00-ARRANQUE.md` antes de responder"* (en vez de `00-MAESTRO`) y *"actualizá los documentos vos directamente — no pases resúmenes para que el operador pegue"* (en vez de *"generá un resumen para actualizar los documentos"*). Bloque sugerido en `MANUAL §1.1`.
- **Pendiente para próxima sesión:** revisar si `00-MAESTRO.md §2.2` (tablas de meta fields de `canchas` y `reservas`) debería listar las meta keys con prefijo `fu_` para alinear con §22.4 — hoy aparecen como `cancha_direccion`, `reserva_estado` sin prefijo en el MAESTRO mientras §22.4 (y ahora `02-GUIAS §3`) dicen `fu_cancha_direccion`, `fu_reserva_estado`.

---

*Próxima entrada: §22.6 (pendiente — todavía no surgió).*
