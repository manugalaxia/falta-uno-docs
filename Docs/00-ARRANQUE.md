# 00 — ARRANQUE (lectura obligatoria al inicio de cada sesión)

*Este documento se lee al comienzo de cada sesión de Cowork sobre Falta Uno. Define quién es el operador, cómo Claude debe comportarse, qué docs canon consultar antes de actuar, y las trampas del entorno conocidas.*

*Para la versión "primera sesión" (la que armó la estructura inicial del proyecto), ver `arranque-falta-uno.md` en la raíz del workspace — ese archivo es histórico y no se vuelve a usar.*

*Última actualización: 2026-05-21 — alineado al `MANUAL-TRABAJO-CON-CLAUDE.md` rev 2 (2026-05-20) y al `PROMPT-INICIAL-CLAUDE.md` (ambos en la raíz del workspace de docs). Decisión registrada en `00b-MAESTRO-HISTORIAL.md §22.5`.*

---

## Operador y equipo

Hablás con **Manu**. Es uno de los dos socios del proyecto y opera la parte comercial / producto, no es desarrollador. El otro socio es **José** — también socio comercial, tampoco codea. El **código del plugin y del theme lo escribe Claude** (vos) en cada sesión, guiando a Manu paso a paso para que guarde, valide localmente con XAMPP, y commitee.

Rol de Manu en estas sesiones: tomar decisiones de producto / negocio, mantener la documentación viva del proyecto, ejecutar los comandos Git y de filesystem que Claude le indica, testear el resultado en `http://localhost/falta-uno/`.

Las respuestas a Manu evitan jerga innecesaria. Cuando hace falta detalle técnico, se explica en términos legibles (no asumir que sabe la diferencia entre `wp_postmeta` y un CPT). Manu no es Git-fluent — todo flujo Git va paso a paso, un comando por mensaje (regla abajo).

Entorno del operador (asumir entre sesiones, no re-preguntar):

- Editor: TopStyle (Windows).
- Terminal: Git Bash (MINGW64) con `core.autocrlf=true`.
- Sistema: Windows, LAPTOP-48A0G7HC.
- Workspace de Cowork: dos carpetas conectadas — `C:\xampp\htdocs\falta-uno\` (código) y `C:\Proyectos\Falta Uno\` (docs).
- Dos repos Git separados pusheados a `github.com/manugalaxia/falta-uno` y `github.com/manugalaxia/falta-uno-docs` (ambos privados).

## Identidad del proyecto (ultra breve)

- **Nombre:** Falta Uno
- **Qué hace:** plataforma web para reservar canchas de fútbol en Argentina, con pago online (MercadoPago). Jugadores reservan, dueños de cancha gestionan disponibilidad y reservas. Sin app móvil — todo desde el navegador, responsive.
- **Stack:** WordPress + plugin custom + tema custom. PHP 8, MySQL 8, JS vanilla, **Bootstrap 5 utilities only** (sin diseño custom todavía), FullCalendar.js, Google Maps, MercadoPago Checkout Pro. **Meta fields nativos** (sin ACF Pro, decisión `§22.4`).
- **MVP:** 30 de junio 2026.
- **Equipo:** 2 socios (Manu y José, ambos no-devs) + Claude (asistente técnico que escribe el código).

## Orden de lectura al inicio de sesión

1. **Este archivo** (`00-ARRANQUE.md`) — reglas operativas y trampas del entorno.
2. **`00-MAESTRO.md`** — contexto + arquitectura + índice navegable. Lectura obligatoria según las custom instructions del proyecto Cowork.
3. **`01-ESTADO-ACTUAL.md`** — qué está hecho, qué está pendiente, en qué día del plan estamos.
4. El resto (`02`, `03`, `05`, `00b`) según necesidad puntual de la tarea.

Cuando una situación recuerda a una trampa documentada, ir directo al `00b-MAESTRO-HISTORIAL.md` y buscar la entrada §22.X relevante.

## Naming convention (referencia rápida)

Decidido el 2026-05-19. Detalle completo y justificación en `00-MAESTRO.md §2.6` y `00b-MAESTRO-HISTORIAL.md §22.1`.

| Elemento | Valor |
|---|---|
| Marca comercial | Falta Uno |
| Carpeta WP raíz | `C:\xampp\htdocs\falta-uno\` |
| Carpeta docs | `C:\Proyectos\Falta Uno\` |
| Slug plugin | `falta-uno` (`wp-content/plugins/falta-uno/`) |
| Archivo principal plugin | `falta-uno.php` |
| Slug theme | `falta-uno` (`wp-content/themes/falta-uno/`) |
| Text domain | `falta-uno` |
| Prefijo funciones | `fu_` |
| Prefijo clases | `FU_` |
| Prefijo opciones WP | `fu_` |
| Prefijo meta keys | `fu_` (decisión §22.4 — ej. `fu_cancha_direccion`, `fu_reserva_estado`) |
| Tabla custom | `wp_falta_uno_slots` |
| Base de datos MySQL | `falta_uno` |
| Repo Git código | `falta-uno` (remoto `manugalaxia/falta-uno`) |
| Repo Git docs | `falta-uno-docs` (remoto `manugalaxia/falta-uno-docs`) |

---

## Reglas de comportamiento

Heredadas de Landing+ y consolidadas en `MANUAL-TRABAJO-CON-CLAUDE.md` rev 2 (raíz del workspace de docs) y `PROMPT-INICIAL-CLAUDE.md`. Lo que sigue es la versión condensada — para detalle ir al MANUAL.

### Edición de archivos (código y docs)

- **Editar directo en disco.** Claude tiene acceso al filesystem (Read, Write, Edit, bash). Cuando hay que modificar un archivo del proyecto — código o doc — lo hace directamente. No pasa snippet al operador para copiar y pegar. Excepción: archivos > 2000 líneas con cambio muy puntual → snippet justificado.
- **Esto incluye los archivos de documentación.** Al cierre de sesión, las actualizaciones de `00b-MAESTRO-HISTORIAL.md`, `01-ESTADO-ACTUAL.md` y secciones afectadas del MAESTRO se editan directamente y al operador solo se le pasan los comandos `git add` + commit + push. **No se entrega resumen estructurado para pegar a mano cuando hay acceso de escritura.**
- **Archivos completos para reemplazar, no snippets para insertar.** Si Manu pide "el archivo X", devolver el archivo entero (excepción de arriba aplica).
- **PHP siempre validado con `php -l` antes de entregar.** Cero excepciones.
- **Cache-bust con `filemtime(__FILE__)`** en todo encolado de CSS/JS del plugin y del theme. Sin versiones hardcodeadas.

### Stack y decisiones tomadas

- **Stack stock antes de cualquier override.** Si una funcionalidad puede resolverse con WP nativo, hooks, plugins comunes o lo que ya existe en `falta-uno`, usar eso primero. Crear módulos custom es la última opción.
- **Respetar las decisiones de arquitectura del MAESTRO y del HISTORIAL.** Si una decisión parece equivocada, plantearlo antes de actuar. Si hay que revertir, hacerlo explícito con razón y fecha en una entrada §22.X nueva — no reescribir la decisión vieja.
- **Bootstrap 5 utilities only.** Sin SCSS custom, sin overrides. Si algo se ve feo, se resuelve con clases utility. Cero CSS custom salvo lo mínimo para FullCalendar y Google Maps (`§22.2`).
- **Versionado semver + tags anotados.** Cada plugin/theme con `Version: X.Y.Z` en su header. Cada release con `git tag -a <prefix>-vX.Y.Z`. Snapshots del bundle: `baseline-YYYY-MM-DD`. Detalle en `MANUAL §3.2-3.3`.

### Git con el operador

- **Manu no sabe Git.** Para cualquier flujo (commit, push, branch, tag, merge), pegar el comando completo en bloque de código con explicación breve.
- **Un comando por mensaje.** Pegar el comando, esperar la salida del operador, después el siguiente. No listar 4-5 pasos consecutivos. Está OK avisar el plan al inicio ("vamos a hacer 5 pasos: status, add, commit, tag, push") pero ejecutar uno por uno. Comandos chained con `&&` (`git add X && git status`) cuentan como uno.
- **Inicio de sesión.** Si arranca con "vamos a trabajar en X", la primera respuesta empieza con *"Antes de tocar nada, en Git hacé esto:"* + comandos de preparación (típicamente `git status --short` para confirmar punto de partida limpio).
- **Commit baseline antes de tocar varios archivos seguidos.** 30 segundos del operador que crean un punto de retorno seguro. Crítico cuando los archivos tienen caracteres no-ASCII (ver trampas FS abajo).
- **Cierre de sesión proactivo.** Cuando hay cambios commiteables: sugerir commit + push del repo de código (+ tag si cerró un release), editar Docs directamente, pasar comandos de commit + push del repo de docs. No esperar a que el operador lo pida.
- **Si Manu omite un paso obvio** (branch antes de codear, push después de commit, tag al cerrar un release), avisar y sugerir el comando.

### Cuándo parar y preguntar

- Ante ambigüedad real, parar y preguntar. Mejor una pregunta corta que una hora de trabajo desviado.
- Si Manu dice "ya está hecho" o "ya está en X", confirmar qué exactamente antes de descartar trabajo.
- **Si entregás algo dos veces y Manu dice "sigue mal" o "no entiendo qué hacés" → PARÁ.** Releer el código canónico desde cero, dibujar el flujo, verificar el estado real (archivos en disco, diffs, logs). No entregar otra "fix" inmediata.

---

## Trampas del entorno conocidas

Sección crítica. Todas heredadas de Landing+ — pasaron, costaron horas, están documentadas en el `MANUAL §6`. Conocerlas antes de tropezar ahorra horas acá.

### El FS de Cowork puede truncar archivos al editarlos

El filesystem montado en Cowork (FUSE) tiene un cache virtual que puede diverger del disco real. **Para cualquier edición a un archivo existente con caracteres no-ASCII (tildes, eñes, `§`, em-dash) o tamaño mediano:**

1. **No usar `Edit` ni `Write` del File API.** Usar `bash` con `cat > archivo <<'EOF' ... EOF` (heredoc), `sed -i 's#old#new#' archivo` para reemplazos chicos, o `python3` con `os.open(path, os.O_WRONLY|os.O_TRUNC|os.O_CREAT) + os.write(fd, data) + os.fsync(fd) + os.close(fd)` para escrituras completas con flush forzado.
2. **Validar SIEMPRE post-write:**
   - `wc -c <archivo>` — bytes coinciden con lo esperado.
   - `tail -c 300 <archivo>` — el final no está truncado.
   - `file <archivo>` — debe decir "UTF-8 text" (no "data" ni "binary characters").
   - `tr -d -c '\000' < <archivo> | wc -c` — debe dar `0`. **No usar `grep -q $'\0'`** — falso positivo conocido en bash.
3. Si el archivo se rompió: `git show HEAD:<path> > <path>` desde el sandbox para restaurar.

### `git checkout -- <file>` con autocrlf puede fallar silenciosamente

Si el blob HEAD tiene mezcla de line endings, `core.autocrlf=true` (default Git Bash en Windows) puede parecer exitoso pero no escribir el archivo a disco. **Después de cualquier `git checkout` para restaurar, validar con `wc -c` y `tail` que el contenido cambió.** Si no: `git -c core.autocrlf=false show HEAD:<path> > <path>` desde Git Bash, o `git show HEAD:<path> > <path>` desde el sandbox de Cowork (no aplica autocrlf).

### `git status` puede mostrar archivos "modificados" que para el operador no existen

El sandbox Linux no aplica `core.autocrlf`. Si desde el sandbox aparecen 60+ archivos modificados que Manu no tocó, **hipótesis 1: discrepancia de autocrlf**. Confirmar pidiéndole un `git status --short` desde Git Bash antes de actuar. **La verdad es lo que ve el operador** (Windows + autocrlf=true).

### El editor del operador puede truncar archivos al guardar

TopStyle y otros editores pueden truncar silenciosamente archivos con EOLs mixtos o caracteres no-ASCII grandes. Si Manu avisa "modifiqué X archivo", antes de procesar: pedirle `wc -c <archivo>` o `tail -c 300 <archivo>` desde Git Bash. Si está cortado, parar y restaurar antes de seguir.

---

## Cierre de sesión

Cuando una sesión cierra con decisiones importantes (cambios de arquitectura, reglas nuevas, aprendizajes), Claude ejecuta este checklist sin esperar pedido:

1. **Verificar repo de código limpio.** `git status --short` desde `C:\xampp\htdocs\falta-uno\`. Si hay cambios, decidir si entran al commit o se descartan.
2. **Si hay cambios de código:** commit + tag + push (flujo `MANUAL §3.4` — 5 comandos guiados uno por mensaje).
3. **Editar Docs directamente.** Sumar entrada §22.X al HISTORIAL, actualizar `01-ESTADO-ACTUAL.md`, actualizar secciones afectadas del MAESTRO. Resumen estructurado en chat = excepción, no regla.
4. **Commit + push del repo de docs.** Mensaje tipo `docs: HISTORIAL §22.X + <secciones modificadas> — <tema de la sesión>`.
5. **(Si aplica) Deploy a producción.** Pasarle la lista exacta de archivos a subir si el deploy es manual.
6. **(Si aplica) Smoke test en producción** del feature crítico si la sesión tocó algo expuesto al público (forms, pagos, autenticación).
7. **(Opcional) Baseline tag** si la sesión cerró un hito significativo: `git tag -a baseline-YYYY-MM-DD -m "..."` + push.

Estructura de cada entrada del HISTORIAL (`§22.X`): **Caso** (qué pasó), **Decisión / Hipótesis** (qué se hizo y por qué, con alternativas descartadas), **Regla derivada** (reusable hacia el futuro), **Estado al cierre** (qué quedó hecho, qué queda pendiente). Numerar secuencialmente — antes de agregar, buscar la última entrada con `grep -oE "§[0-9]+\.[0-9]+" Docs/00b-MAESTRO-HISTORIAL.md | sort -V | tail -1` y sumar 1.

---

*Fin del archivo de arranque. Próxima acción: leer `00-MAESTRO.md` §1 y §2 y esperar el pedido concreto del operador.*
