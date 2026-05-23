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
- El briefing técnico inicial del proyecto que usa `fulboapp` en todos lados (`fulboapp.php`, `wp_fulboapp_slots`, etc.) — nombre de proyecto anterior, descartado.
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

**Caso:** El MAESTRO original (sembrado desde el briefing inicial del proyecto) listaba **Advanced Custom Fields (ACF Pro)** como dependencia para los meta fields de los CPTs `canchas` y `reservas` (ver §2.2 del MAESTRO en su versión previa). Al llegar al ítem "instalar ACF Pro" del checklist del Día 1, Manu decide **no** usar ACF y hacer toda la administración de campos dentro del plugin custom `falta-uno`.

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

## §22.6 — 2026-05-21 — Convención de idioma en código y datos: español con prefijo `fu_`, infra WP en inglés

**Caso:** Decisión diferida desde la sesión del 2026-05-19 (registrada como #1 en `01-ESTADO-ACTUAL.md`). El briefing técnico inicial del proyecto mezclaba nombres en español (`cancha_direccion`, `reserva_estado`, rol `dueno_cancha`) con hooks de WP en inglés (`add_action`, `get_post_meta`). Al alinear las inconsistencias residuales en la sesión del 2026-05-21 — ver `§22.5` — quedó como pendiente formalizar la convención global y aplicarla a las meta keys del MAESTRO (`§2.2`), las capabilities de los roles (`§2.4`) y los identificadores futuros (functions, hooks custom, AJAX actions, REST endpoints).

**Decisión:** Convención **(b) datos en español, infra WP en inglés**, con prefijo `fu_` para todo lo que sea custom del proyecto.

| Categoría | Idioma | Prefijo |
|---|---|---|
| CPT slugs | Español | sin prefijo (es un slug) |
| Meta keys | Español | `fu_` |
| Roles | Español | sin prefijo (es un slug) |
| Capabilities custom | Español | `fu_` |
| Hooks/filters custom (`do_action`/`apply_filters`) | Español | `fu_` |
| AJAX actions custom | Español | `fu_` |
| REST endpoints | Español | namespace `falta-uno/v1` |
| Funciones del plugin | Español | `fu_` |
| Clases del plugin | Inglés-PHP (PascalCase) | `FU_` |
| Constantes | Inglés convencional | `FU_` |
| Variables locales / parámetros | Español | sin prefijo |
| Hooks/funciones nativas de WP | **Inglés (no se cambia)** | el que tengan |
| Columnas de tabla custom | Español | sin prefijo |
| Mensajes de UI | Español rioplatense | — |

Detalle completo y tabla con ejemplos en `02-GUIAS-TECNICAS.md §5`.

**Por qué:**

1. **Lectura natural para Manu.** Los datos los va a ver Manu en el admin de WP, en exports de DB, en mensajes de error de Claude. `fu_cancha_direccion = "Av. Cabildo 1234"` se lee directo, sin traducir.
2. **Coherencia con los CPTs.** Los CPTs ya son `canchas` y `reservas` (en español, en el MAESTRO desde §2.2). Si las meta keys de esos CPTs fueran en inglés sería raro (`fu_field_address` adentro de un post `canchas`).
3. **Continuidad con el briefing inicial.** El briefing arrancó en español para los datos del dominio. Cambiarlo a inglés ahora sería rehacer trabajo sin valor agregado, porque ningún dev externo va a tomar el código (el equipo real es Manu + José socios no-devs + Claude técnico, ver `§22.7`).
4. **Sin acentos en identificadores.** PHP/SQL aceptan UTF-8 en identificadores pero no es portable. Convención: `dueno_cancha`, no `dueño_cancha`. La regla queda registrada en `02-GUIAS §5`.

**Regla derivada:**

- **Al nombrar cualquier campo, meta key, rol, capability, función, hook custom, filtro custom o action AJAX nuevo, consultar `02-GUIAS-TECNICAS.md §5`.** El nombre se escribe en español, sin acentos, con guion bajo entre palabras, con prefijo `fu_` si es del namespace del proyecto.
- **Hooks y funciones nativas de WordPress (`add_action`, `get_post_meta`, `wp_enqueue_scripts`, etc.) se usan tal cual.** No traducir, no envolver, no abstraer.
- **Si en algún momento Falta Uno necesita i18n real** (traducción a otros idiomas), envolver strings de UI con `__( 'texto', 'falta-uno' )`. El text domain `falta-uno` ya está reservado. Pero en el MVP no se envuelve nada.

**Estado al cierre:**

- `02-GUIAS-TECNICAS.md §5` escrita con tabla por categoría y convenciones particulares.
- `00-MAESTRO.md §2.2` actualizado: 10 meta keys de `canchas` y 9 de `reservas` ahora llevan prefijo `fu_` (ej. `fu_cancha_direccion`, `fu_reserva_estado`).
- `00-MAESTRO.md §2.4` actualizado: capabilities custom ahora son `fu_crear_reservas`, `fu_gestionar_canchas_propias`, `fu_ver_reservas_propias`.
- Decisión diferida #1 cerrada en `01-ESTADO-ACTUAL.md`.
- Esta entrada §22.6 sumada al HISTORIAL.

---

## §22.7 — 2026-05-21 — Corrección del equipo del proyecto: no existe "Franco"

**Caso:** El `arranque-falta-uno.md` original (histórico, no canon) y el sembrado inicial del `00-ARRANQUE.md` listaban un "dev part-time" llamado **Franco** como tercer integrante del proyecto. Ese dato se propagó a `00-MAESTRO.md §3` ("briefing técnico de Franco") y al `00b-MAESTRO-HISTORIAL.md §22.1` / `§22.4` ("briefing de Franco"). Al confirmar el equipo real con Manu durante la sesión del 2026-05-21, sale que **Franco no existe en el proyecto** y que la composición real es otra.

**Decisión:** Registrar la composición real del equipo y limpiar todas las menciones a "Franco" en los docs canon.

Equipo real de Falta Uno:

- **Manu** — socio comercial / producto. Opera Cowork, decide producto/negocio, ejecuta los comandos Git y de filesystem que Claude le indica, testea el resultado en `http://localhost/falta-uno/`. No es desarrollador.
- **José** — socio comercial / producto. No codea. Su rol exacto en el día a día del producto queda fuera del scope de esta doc por ahora (no afecta las decisiones técnicas).
- **Claude** (asistente técnico vía Cowork) — escribe el código del plugin y del theme en cada sesión, guía a Manu paso a paso para guardar, validar y commitear.

**No hay dev externo ni part-time.** El código del proyecto sale 100% de la colaboración Claude+Manu.

**Por qué importa:**

1. **La doc canon hablaba de un coordinador con Franco que no existe.** Frases tipo "coordinar con Franco lo que va al backlog" o "si Manu o Franco piden el archivo X" inducen comportamientos equivocados (esperar inputs de Franco, escribir docs pensando en un dev externo que las va a leer).
2. **El briefing-desarrollador.md no tiene autor confirmado.** Quedó atribuido a Franco por error de redacción inicial. Hoy se trata como "briefing técnico inicial del proyecto, referencia de origen, autor histórico" — sin atribución personal.
3. **Implicancia operativa.** Como no hay dev externo, las decisiones de "qué cuenta como interfaz pública" o "qué hay que dejar documentado para alguien que no participó de la sesión" cambian. Lo que documentamos es para Manu (no-dev) y para futuras sesiones de Claude, no para un colega técnico.

**Regla derivada:**

- **No volver a usar el nombre "Franco" en docs canon.** Si aparece, es un residual a limpiar.
- **Cuando se necesite atribuir un archivo histórico** (briefing, mockups, doc previa) y el autor real no está confirmado, escribir "autor histórico, referencia de origen" sin nombre — no inventar atribución.
- **Las menciones genéricas a "el dev" en futuras docs canon** deben ser reemplazadas por "Claude" (que es quien escribe el código). Excepción: cuando el contexto sea claramente futuro hipotético ("si en algún momento se contrata un dev externo").

**Estado al cierre:**

- `00-ARRANQUE.md` sección "Operador" reescrita como "Operador y equipo" con la composición real.
- `00-ARRANQUE.md` línea "Equipo" en "Identidad ultra breve" actualizada (2 socios + Claude).
- `00-ARRANQUE.md` regla "Si Manu o Franco piden 'el archivo X'" → "Si Manu pide 'el archivo X'".
- `00-MAESTRO.md §3` (índice de docs históricos) — `briefing-desarrollador.md` queda como "briefing técnico inicial del proyecto (referencia de origen, autor histórico)".
- `00b-MAESTRO-HISTORIAL.md §22.1` y `§22.4` — referencias a "briefing de Franco" cambiadas a "briefing técnico inicial" / "briefing inicial".
- `arranque-falta-uno.md` (histórico, raíz del workspace) NO se toca — es snapshot del bootstrap original, queda como fuente de verdad de "esto pensábamos en ese momento".
- Esta entrada §22.7 sumada al HISTORIAL.

---

## §22.8 — Bootstrap del plugin (CPTs vacíos) + theme completo en una sola sesión (Día 2 parcial + Día 5)

*Fecha: 2026-05-23.*

**Caso:**

Manu arrancó la sesión pidiendo el theme directamente, salteando el Día 2 (bootstrap del plugin con CPTs + autoloader + clases) y los Días 3-4 (roles + meta boxes + tabla `wp_falta_uno_slots`) del plan original de Fase 1. Hipótesis razonable: visualmente sentís más progreso viendo el sitio con páginas que registrando hooks en PHP sin nada para mostrar. Pero el theme no se sostiene solo: sin el CPT `canchas` registrado, los templates `archive-canchas.php` y `single-canchas.php` no son reconocidos por WordPress y no se cargan nunca.

**Decisión / Hipótesis:**

Scaffolding mínimo del plugin antes de arrancar el theme, en la misma sesión. Concretamente:

- `wp-content/plugins/falta-uno/falta-uno.php` (header WP + constantes + clase singleton + hooks de activación/desactivación) y `includes/class-fu-plugin.php` (registro de los CPTs `canchas` público con archive y `reservas` interno). **Sin meta boxes, sin helper `FU_Meta`, sin tabla `wp_falta_uno_slots`, sin roles `add_role()`.** Solo los CPTs vacíos. Versión: `0.1.0`. Tag: `fu-plugin-v0.1.0`.
- Theme completo: 11 archivos (`style.css`, `functions.php`, `header.php`, `footer.php`, `front-page.php`, `archive-canchas.php`, `single-canchas.php`, `index.php`, `page-login.php`, `page-registro.php`, `page-mi-panel.php`). Encolado de Bootstrap 5 CDN + Bootstrap Icons + style.css del tema con cache-bust por `filemtime`. Admin bar oculta en frontend para todos los usuarios. Versión: `0.1.0`. Tag: `fu-theme-v0.1.0`.

Alternativas descartadas:

- *Respetar el roadmap (Día 2 → 3 → 4 → 5).* Habría dado un Día 2 visualmente nulo (todo en PHP sin frontend), seguido de dos días más de metadata sin pantallas. Suma fricción para Manu, que necesita ver progreso visual.
- *Theme antes que plugin, con datos dummy hardcodeados.* Hubiera funcionado para el día, pero al sumar el plugin después había que volver a tocar los templates para reemplazar dummies por queries reales. Doble laburo.

**Excepción CSS explícita (relacionada con §22.2):**

El theme tiene un `style.css` con ~30 líneas de override de variables Bootstrap (`--bs-primary` y `--bs-primary-rgb` al verde Falta Uno `#00c850`, más overrides puntuales de `.btn-primary` y `.btn-outline-primary` para mantener consistencia en hover/active, y color de links). Esto contradice literalmente la regla `Bootstrap utilities only, cero CSS custom` de `§22.2`. **Excepción aceptada** porque el override del primario no es "diseño custom" sino "configuración de la paleta de marca" — cambiar dos custom properties es menos custom que reemplazar `btn-primary` por una clase propia en cada template. Se equipara con las excepciones ya previstas para FullCalendar y Google Maps en `§22.2`.

**Regla derivada:**

- **Si el theme necesita templates de archive/single de un CPT, el CPT tiene que estar registrado antes — aunque sea vacío.** No es opcional ni "lo dejamos para después". Un registro de CPT son ~30 líneas y media hora; sin eso el template es código muerto.
- **Override del primario de Bootstrap vía custom properties cuenta como excepción aceptada al "cero CSS custom"**, equivalente a las excepciones de FullCalendar y Maps. Cualquier otro CSS propio sigue siendo violación y requiere su propia entrada §22.X que lo justifique.
- **Los page templates auxiliares (login, registro, mi-panel) requieren que Manu cree las páginas en wp-admin y les asigne el template** desde el panel "Atributos de página → Plantilla". El template solo no genera URL. Documentar esto en el inventario.
- **Para mensajes de commit con `-m` en Git Bash sobre Windows, evitar multilinea con caracteres especiales (`(`, `)`, `:`, `§`).** Usar mensajes de una línea cortos y dejar el detalle en el HISTORIAL — el repo es para hash + título, el porqué vive acá.

**Estado al cierre:**

- Plugin `falta-uno` v0.1.0 activo en local, registra CPTs `canchas` y `reservas`. Validado con `php -l`. Cargadas 4 canchas dummy.
- Theme `falta-uno` v0.1.0 activo en local. 11 templates funcionando, navbar Bootstrap con verde primario, archive y single de canchas operativos, páginas auxiliares creadas y asignadas al template correspondiente. Admin bar oculta. Validado con `php -l`.
- Commit `a943c0d` en `main` del repo `falta-uno`, dos tags anotados pusheados (`fu-plugin-v0.1.0`, `fu-theme-v0.1.0`).
- Smoke test completo: 6 URLs visitadas (home, /canchas/, single de cancha, /login/, /registro/, /mi-panel/), todas renderizan sin error.

**Queda pendiente del roadmap (no bloqueante para el siguiente paso visible):**

- **Día 2 (resto):** autoloader + clases adicionales del plugin (cuando aparezca la necesidad — hoy con el singleton `FU_Plugin` alcanza).
- **Día 3 completo:** roles `jugador` / `dueno_cancha` con `add_role()` en activación + meta boxes nativos para `canchas` (todos los `fu_cancha_*` de `00-MAESTRO.md §2.2`) y `reservas` (`fu_reserva_*`) + helper `FU_Meta`.
- **Día 4 completo:** tabla `wp_falta_uno_slots` con `dbDelta()` en activación + funciones de generación y consulta de slots.
- **Día 6:** JS de FullCalendar enchufado al contenedor `#fu-calendario-cancha` de `single-canchas.php` + endpoint AJAX `fu_disponibilidad_cancha` que lee de `wp_falta_uno_slots`.
- **Día 7:** Google Maps en home (placeholder con ícono ya está) y en single (placeholder en `#fu-mapa`) + form frontend de carga de cancha.
- **Día 8:** handler de login + registro (los forms HTML ya están en los page templates) + asignación de rol según radio button `fu_rol`.
- **Día 9:** flujo de reserva sin pago (slot → CPT reserva → bloqueo de slot → email).

**Convención derivada de filtros UI placeholder:**

Los chips de tipo de cancha en `front-page.php` y `archive-canchas.php` están como UI pero no filtran realmente (faltan los meta boxes que pueblen `fu_cancha_tipo`). Cuando los meta boxes existan, conectar vía hook `pre_get_posts` que lea `?tipo=` y agregue `meta_query`. Comentario `// TODO: conectar Día 3` queda visible en los archivos para no perder la referencia.

---

*Próxima entrada: §22.9 (pendiente — todavía no surgió).*
