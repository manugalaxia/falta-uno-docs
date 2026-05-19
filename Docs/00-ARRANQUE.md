# 00 — ARRANQUE (lectura obligatoria al inicio de cada sesión)

*Este documento se lee al comienzo de **cada** sesión de Cowork sobre Falta Uno. Define quién es el operador, cómo Claude debe comportarse, y qué documentos canon consultar antes de actuar.*

*Para la versión "primera sesión" (la que armó la estructura inicial del proyecto), ver `arranque-falta-uno.md` en la raíz del workspace — ese archivo es histórico y no se vuelve a usar.*

---

## Operador

Hablás con **Manu**. Lleva la parte comercial / producto del proyecto, no es desarrollador. El dev part-time se llama **Franco** y trabaja directo sobre el código. El rol de Manu en estas sesiones es:

- Tomar decisiones de producto / negocio
- Mantener la documentación viva del proyecto
- Coordinar con Franco lo que va al backlog y al roadmap

Las respuestas a Manu evitan jerga innecesaria. Cuando hace falta detalle técnico, se explica en términos legibles (no asumir que sabe la diferencia entre `wp_postmeta` y un CPT).

## Identidad del proyecto (ultra breve)

- **Nombre:** Falta Uno
- **Qué hace:** plataforma web para reservar canchas de fútbol en Argentina, con pago online (MercadoPago). Jugadores reservan, dueños de cancha gestionan disponibilidad y reservas. Sin app móvil — todo desde el navegador, responsive.
- **Stack:** WordPress + plugin custom + tema custom. PHP 8, MySQL 8, JS vanilla, **Bootstrap 5 utilities only** (sin diseño custom todavía), FullCalendar.js, Google Maps, MercadoPago Checkout Pro, ACF Pro.
- **MVP:** 30 de junio 2026.
- **Equipo:** 1 dev part-time (Franco), 1 operador comercial (Manu).

## Orden de lectura al inicio de sesión

1. **Este archivo** (`00-ARRANQUE.md`) — reglas operativas.
2. **`00-MAESTRO.md`** — contexto + arquitectura + índice navegable. **Lectura obligatoria** según las custom instructions del proyecto Cowork.
3. **`01-ESTADO-ACTUAL.md`** — qué está hecho, qué está pendiente, en qué día del plan estamos.
4. El resto (`02`, `03`, `05`, `00b`) según necesidad puntual de la tarea.

## Naming convention (referencia rápida)

Decidido el 2026-05-19. Detalle completo y justificación en `00-MAESTRO.md` §2.6 y `00b-MAESTRO-HISTORIAL.md` §22.1.

| Elemento | Valor |
|---|---|
| Marca comercial | Falta Uno |
| Carpeta WP raíz | `C:\xampp\htdocs\falta-uno\` |
| Carpeta docs | `C:\Proyectos\Falta Uno\Docs\` |
| Slug plugin | `falta-uno` (`wp-content/plugins/falta-uno/`) |
| Archivo principal plugin | `falta-uno.php` |
| Slug theme | `falta-uno` (`wp-content/themes/falta-uno/`) |
| Text domain | `falta-uno` |
| Prefijo funciones | `fu_` |
| Prefijo clases | `FU_` |
| Prefijo opciones WP | `fu_` |
| Tabla custom | `wp_falta_uno_slots` |
| Base de datos MySQL | `falta_uno` |
| Repo Git código | `falta-uno` |
| Repo Git docs | `falta-uno-docs` |

---

## Reglas de comportamiento (heredadas de Landing+)

### Sobre el stack y las decisiones tomadas

- **Stack stock antes de cualquier override.** Si una funcionalidad puede resolverse con WP nativo, ACF, hooks, plugins comunes — usar eso primero. Crear módulos custom o sobreescribir core es la **última** opción, no la primera. Falta Uno es un WP normal: no inventar abstracciones que no tiene la plataforma.
- **Respetar las decisiones de arquitectura del MAESTRO.** Si una decisión parece equivocada, plantearlo antes de actuar — no actuar como si no existiera. Si hay que revertir una decisión vieja, hacerlo explícito con razón y fecha en `00b-MAESTRO-HISTORIAL.md`.
- **Bootstrap 5 utilities only.** Sin SCSS custom, sin overrides. Si algo se ve feo, se resuelve con clases utility de Bootstrap. Cero CSS custom en el theme salvo lo mínimo para FullCalendar y Google Maps.

### Sobre cómo entregar trabajo

- **Archivos completos para reemplazar, no snippets para insertar.** Si Manu o Franco piden "el archivo X", devolver el archivo entero. Snippets solo si el archivo es muy grande (>2000 líneas) y el cambio es muy puntual.
- **PHP siempre validado con `php -l` antes de entregar.** Cero excepciones. Un parse error en producción es vergonzoso y evitable.
- **Cuando haya que generar archivos largos, hacerlos en el contenedor de Cowork y entregarlos con `present_files`** — no streamear en chat.
- **Bundles diferenciales** (solo archivos modificados) > acumulativos.
- **Git: comandos explícitos siempre.** Cuando haya que commitear, hacer push, crear branch, etc., pegar el comando completo en bloque de código. No decir "hacé un commit" a secas.
- **Cache-bust con `filemtime(__FILE__)`** en todo encolado de CSS/JS del plugin y del theme. No usar versiones hardcodeadas.

### Sobre cuándo parar y preguntar

- Ante ambigüedad real, parar y preguntar. Mejor una pregunta corta que una hora de trabajo desviado.
- Si Manu dice "ya está hecho", confirmar **qué** está hecho exactamente antes de descartar trabajo.
- **Si entregaste algo dos veces y el operador dice "sigue mal" o "no entiendo qué hacés" → PARAR.** Releer el código canónico desde cero, dibujar el flujo, verificar el estado real. No entregar otra "fix" inmediata.

### Sobre el cierre de sesión

Cuando la sesión cierra con decisiones importantes (cambios de arquitectura, reglas nuevas, aprendizajes), generar un resumen estructurado y **editar directamente** `01-ESTADO-ACTUAL.md` y `00b-MAESTRO-HISTORIAL.md` sin pedir confirmación adicional. El resumen tiene que cubrir:

1. Qué se decidió / construyó.
2. Archivos modificados o creados.
3. Aprendizajes nuevos que merezcan entrar al HISTORIAL §22 (con caso concreto + regla derivada). Numerar secuencialmente.
4. Pendientes / decisiones diferidas.
5. Bloques sugeridos para sumar al MAESTRO.

---

*Última actualización: 2026-05-19 — primera versión, generada al armar la estructura inicial de docs.*
