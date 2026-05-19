# ARRANQUE — Proyecto Falta Uno

*Este documento se sube al inicio de la **primera sesión** de Cowork del proyecto Falta Uno. Contiene las instrucciones para que Claude se autoconfigure, entienda el proyecto y deje todo el andamiaje montado. El operador (Manu) no tiene que acordarse de nada: subir este archivo + el briefing técnico del dev es suficiente.*

---

## Para Claude — cómo arrancar esta sesión

### Quién es el operador

Hablás con **Manu**. Lleva la parte comercial / producto del emprendimiento, no es desarrollador. El dev part-time se llama **Franco** y va a trabajar sobre el código directamente. El rol de Manu en estas sesiones es:

- Tomar decisiones de producto / negocio
- Mantener la documentación viva del proyecto
- Coordinar con Franco lo que va al backlog y al roadmap

Por eso las respuestas a Manu deben evitar jerga innecesaria. Cuando haga falta detalle técnico (lo va a haber, porque la doc lo necesita), explicarlo en términos legibles, no asumir que ya sabe la diferencia entre `wp_postmeta` y un CPT.

### Identidad del proyecto (ultra breve)

- **Nombre:** Falta Uno
- **Qué hace:** plataforma web para reservar canchas de fútbol en Argentina, con pago online (MercadoPago). Jugadores reservan, dueños de cancha gestionan disponibilidad y reservas. Sin app móvil — todo desde el navegador, responsive.
- **Stack:** WordPress + plugin custom + tema custom. PHP 8, MySQL 8, JS vanilla, FullCalendar.js, Google Maps, MercadoPago Checkout Pro, ACF Pro.
- **MVP:** 30 de junio 2026.
- **Equipo:** 1 dev part-time (Franco), 1 operador comercial (Manu).
- **El briefing técnico completo del dev se sube aparte como archivo adjunto** (`briefing-desarrollador.md`). Leerlo entero en la primera sesión.

### Paso 1 — Crear la estructura de carpetas

Falta Uno replica el patrón "código separado de docs" de un proyecto previo del operador (Landing+). Tiene que existir:

```
C:\xampp\htdocs\falta-uno\           ← código WordPress local (XAMPP)
C:\Proyectos\Falta-Uno\              ← raíz del proyecto fuera de XAMPP
C:\Proyectos\Falta-Uno\Docs\         ← documentación viva editable
```

Pedirle al operador que **conecte ambas carpetas como workspace folders** en Cowork (la app de Claude desktop). Sin eso, Claude no puede escribir en ellas. Si alguna de las dos no existe todavía, pedir que la cree (o crearla vía `mcp__workspace__bash` si está conectada).

### Paso 2 — Crear los docs canon

Una vez conectada `C:\Proyectos\Falta-Uno\Docs\`, crear los siguientes archivos canon. Ya con esqueletos mínimos alcanza para arrancar; se van llenando con uso.

| Archivo | Propósito | Estado inicial |
|---|---|---|
| `00-ARRANQUE.md` | Reglas de arranque de sesión (este archivo, pero ya adaptado a docs propias) | Adaptar este mismo doc, sacando el "primer arranque" y dejando "arranque recurrente" |
| `00-MAESTRO.md` | Contexto maestro: §1 qué es Falta Uno, §2 arquitectura, ÍNDICE NAVEGABLE | Sembrar §1 y §2 con el briefing del dev. Dejar el índice navegable preparado para crecer |
| `00b-MAESTRO-HISTORIAL.md` | §22 con aprendizajes / trampas / decisiones revertidas | Vacío con cabecera `# HISTORIAL — Falta Uno` y placeholder `§22.1 — (primer aprendizaje pendiente)` |
| `01-ESTADO-ACTUAL.md` | Qué está hecho, qué está pendiente, en qué fase del plan estamos | Sembrar con el "Plan de trabajo Fase 1" del briefing, todo en estado pendiente |
| `02-GUIAS-TECNICAS.md` | Procedimientos operativos: deploy, configuración de SMTP, MercadoPago sandbox, etc. | Vacío con índice de las guías que se van a necesitar |
| `03-INVENTARIO-TECNICO.md` | Mapa de archivos del plugin y theme. Para ubicar funciones rápido | Sembrar con la estructura del plugin del briefing (fulboapp.php, includes/, ajax/, etc.) y del theme |
| `05-ROADMAP.md` | Backlog ordenado de mejoras / Fase 2 / Fase 3 | Sembrar con la sección "Fase 2" del briefing |

> **Nota sobre nombres de archivos del plugin:** el briefing usa `fulboapp.php`, `fulboapp-plugin/`, etc. Como el nombre real del proyecto es **Falta Uno**, hay que decidir con el operador si renombramos a `falta-uno.php` / `faltauno-plugin/` antes de que Franco arranque, o si dejamos los nombres del briefing como están y solo cambiamos lo de cara al usuario. **Es la primera decisión a tomar en la sesión** — preguntar antes de crear archivos.

### Paso 3 — Git (dos repos separados)

Manu prefiere comandos Git **explícitos en bloque de código**, no instrucciones vagas. Cuando le digas que haga un commit, pegale el comando exacto.

Inicializar dos repos distintos:

```bash
# Repo del código (plugin + theme)
cd C:\xampp\htdocs\falta-uno\wp-content\plugins\falta-uno
git init
git add .
git commit -m "init: estructura plugin Falta Uno"
```

```bash
# Repo de la documentación viva
cd C:\Proyectos\Falta-Uno
git init
git add Docs/
git commit -m "init: docs canon Falta Uno"
```

`.gitignore` mínimo recomendado para el repo de código:

```
node_modules/
vendor/
*.log
@*           # backups con prefijo @, convención heredada de Landing+
.DS_Store
.vscode/
```

### Paso 4 — WordPress local (XAMPP)

Si todavía no está montado, asistir a Manu con:

1. Crear base de datos `falta_uno` en phpMyAdmin (`http://localhost/phpmyadmin`).
2. Descargar WordPress último estable y descomprimir en `C:\xampp\htdocs\falta-uno\`.
3. Configurar `wp-config.php` apuntando a la base `falta_uno`, usuario `root`, password vacío (default XAMPP local).
4. Correr el wizard de WP en `http://localhost/falta-uno/`.
5. Instalar plugins base: **ACF Pro** (licencia a cargo de Manu), un plugin SMTP (WP Mail SMTP o similar para SendGrid/Gmail).
6. Crear el esqueleto del plugin custom en `wp-content/plugins/falta-uno/` con el header estándar de WP y una clase principal vacía.
7. Crear el theme custom en `wp-content/themes/falta-uno/` con `style.css`, `functions.php`, `index.php` mínimos.

**Cache-bust desde el día 1:** todo encolado de CSS/JS del plugin y del theme debe usar `filemtime(__FILE__)` como versión, no un string hardcodeado. Esto evita una hora de "subí los archivos pero veo lo viejo" más adelante.

### Paso 5 — Texto para las custom instructions del proyecto Cowork

Manu va a pegar esto en la configuración del proyecto Cowork (Settings → Custom instructions). Generarlo y mostrárselo claramente al final del arranque:

```
Estás trabajando en Falta Uno. Siempre leé el archivo 00-MAESTRO antes de
responder. Respetá las decisiones de arquitectura ya tomadas. Usá el stack
stock antes de cualquier override. Al finalizar sesiones con decisiones
importantes, generá un resumen para actualizar los documentos.
```

(Idéntico patrón al de Landing+ — es lo que funciona.)

---

## Reglas de comportamiento durante toda sesión

Estas reglas vienen heredadas de Landing+ (otro proyecto del operador donde ya están probadas). Aplican igual en Falta Uno desde el día 1.

### Sobre el stack y las decisiones tomadas

- **Stack stock antes de cualquier override.** Si una funcionalidad puede resolverse con WP nativo, ACF, hooks, plugins comunes — usar eso primero. Crear módulos custom o sobreescribir core es la **última** opción, no la primera. Falta Uno es un WP normal: no inventar abstracciones que no tiene la plataforma.
- **Respetar las decisiones de arquitectura del MAESTRO.** Si una decisión parece equivocada, plantearlo antes de actuar — no actuar como si no existiera. Si hay que revertir una decisión vieja, hacerlo explícito con razón y fecha en el HISTORIAL.

### Sobre cómo entregar trabajo

- **Archivos completos para reemplazar, no snippets para insertar.** Si Manu o Franco piden "el archivo X", devolver el archivo entero. Snippets solo si el archivo es muy grande (>2000 líneas) y el cambio es muy puntual.
- **PHP siempre validado con `php -l` antes de entregar.** Cero excepciones. Un parse error en producción es vergonzoso y evitable.
- **Cuando haya que generar archivos largos, hacerlos en el contenedor de Cowork y entregarlos con `present_files`** — no streamear en chat.
- **Bundles diferenciales** (solo archivos modificados) > acumulativos.
- **Git: comandos explícitos siempre.** Cuando haya que commitear, hacer push, crear branch, etc., pegar el comando completo en bloque de código. No decir "hacé un commit" a secas.

### Sobre cuándo parar y preguntar

- Ante ambigüedad real, parar y preguntar. Mejor una pregunta corta que una hora de trabajo desviado.
- Si Manu dice "ya está hecho", confirmar **qué** está hecho exactamente antes de descartar trabajo.
- **Si entregaste algo dos veces y el operador dice "sigue mal" o "no entiendo qué hacés" → PARAR.** Releer el código canónico desde cero, dibujar el flujo, verificar el estado real. No entregar otra "fix" inmediata.

### Sobre el cierre de sesión

Cuando la sesión cierra con decisiones importantes (cambios de arquitectura, reglas nuevas, aprendizajes), generar un resumen estructurado para actualizar los docs. **Y editar directamente** `01-ESTADO-ACTUAL.md` y `00b-MAESTRO-HISTORIAL.md` sin pedir confirmación adicional. El resumen tiene que cubrir:

1. Qué se decidió / construyó.
2. Archivos modificados o creados.
3. Aprendizajes nuevos que merezcan entrar al HISTORIAL §22 (con caso concreto + regla derivada). Numerar secuencialmente.
4. Pendientes / decisiones diferidas.
5. Bloques sugeridos para sumar al MAESTRO.

---

## Cosas que conviene resolver en la primera sesión

Aprovechar la primera sesión para cerrar decisiones que después salen caras de revertir:

1. **Nombre técnico final.** ¿`falta-uno`, `faltauno`, `falta-1`? Tiene impacto en slug del plugin, prefijo de tabla custom (`wp_faltauno_slots` vs `wp_falta_uno_slots`), nombre del repo, etc. Una sola vez y para siempre.
2. **Idioma del código y de los datos.** El briefing mezcla nombres en español (`cancha_direccion`, `reserva_estado`) y los hooks de WP están en inglés. Definir convención y dejarla en `02-GUIAS-TECNICAS.md`.
3. **Prefijo de funciones / clases del plugin.** ¿`fu_`, `falta_uno_`, `FU_`? Importante para evitar colisiones con otros plugins.
4. **Branding mínimo:** color primario, logo provisorio, dominio definitivo. Aunque sea placeholder, fijarlo evita rehacer el theme dos veces.

---

## Fin del arranque

Próxima acción esperada del Claude del nuevo space:

1. Leer este archivo entero (lo estás haciendo).
2. Leer `briefing-desarrollador.md` (el adjunto técnico).
3. Confirmar con Manu las decisiones del bloque anterior ("Cosas que conviene resolver en la primera sesión").
4. Crear las carpetas (si no existen) y los docs canon mínimos.
5. Mostrarle a Manu el texto de las custom instructions para que las pegue en Cowork.
6. Cerrar la sesión con un mini-resumen y la pregunta "¿arrancamos el plugin la próxima?".

---

*Documento de arranque generado el 2026-05-19 por Claude, basado en el patrón canon del proyecto Landing+ del mismo operador.*
