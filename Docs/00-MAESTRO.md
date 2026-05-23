# 00 — MAESTRO (contexto + arquitectura + índice)

*Documento maestro de Falta Uno. Lectura obligatoria al inicio de cada sesión de Cowork (según custom instructions del proyecto). Contiene la fotografía vigente del proyecto: qué es, cómo está armado, y a dónde buscar lo demás.*

*Última actualización: 2026-05-23 — sumada §2.8 con la estructura real de plugin v0.1.0 y theme v0.1.0 creados en la sesión (§22.8).*

---

## §1 — Qué es Falta Uno

**Falta Uno** es una plataforma web para reservar canchas de fútbol en Argentina, con pago online.

- **Quién la usa:**
  - **Jugadores** buscan canchas por zona, ven disponibilidad, eligen horario y pagan online (MercadoPago).
  - **Dueños de cancha** publican sus espacios, gestionan horarios y ven sus reservas — todo desde el frontend, sin tocar el WP admin.
  - **Admin (Manu)** modera canchas pendientes de aprobación y ve reportes globales.
- **Dónde corre:** navegador (desktop y mobile). Sin app móvil; el sitio es responsive.
- **Pagos:** MercadoPago Checkout Pro (sandbox para desarrollo, producción al ir live).
- **MVP target:** 30 de junio 2026.

### Posicionamiento

El proyecto replica un patrón conocido (booking de canchas) en un mercado donde la mayoría usa WhatsApp + cuaderno. La diferenciación del MVP no es feature-rich, es **confiabilidad y simplicidad**: que un jugador entre, vea disponibilidad real y reserve en menos de dos minutos.

---

## §2 — Arquitectura

### §2.1 Stack tecnológico

| Capa | Tecnología | Notas |
|---|---|---|
| CMS | WordPress (última estable) | Instalación local hoy en `C:\xampp\htdocs\falta-uno\` |
| Backend | PHP 8.x + plugin custom | Plugin: `falta-uno` |
| Base de datos | MySQL 8.x (vía MariaDB en XAMPP) | DB local: `falta_uno`. Tablas nativas WP + 1 tabla custom |
| Frontend | HTML5, CSS3, JavaScript vanilla | **Sin React/Vue.** AJAX nativo de WordPress |
| Estilos | **Bootstrap 5 utilities only** | Sin SCSS custom, sin overrides. Cero CSS propio salvo lo mínimo para FullCalendar y Google Maps |
| Templates | WordPress Template Hierarchy (PHP) | Tema custom `falta-uno` |
| Calendario UI | FullCalendar.js | Para vista de disponibilidad por cancha |
| Mapas | Google Maps JS API + Geocoding API | Pines en home y ficha de cancha |
| Pagos | MercadoPago PHP SDK v3 (Checkout Pro) | Sandbox primero, prod al deploy |
| Emails | `wp_mail()` + SMTP plugin (SendGrid o Gmail) | Para confirmaciones y notificaciones |
| Tareas programadas | WP Cron | Para recordatorios automáticos (Fase 2) |
| Campos custom | Meta fields nativos de WP (`add_meta_box` + `*_post_meta`) dentro del plugin `falta-uno` | Sin ACF Pro ni ACF free — decisión §22.4 |
| Hosting (prod, a definir) | SiteGround / Cloudways / Hostinger | PHP 8 + MySQL 8 + SSL |

### §2.2 Custom Post Types

Se registran desde `wp-content/plugins/falta-uno/includes/class-cpt.php` usando `register_post_type()`.

#### CPT `canchas`

El título del post es el nombre de la cancha. Meta fields nativos (registrados por el plugin con `add_meta_box`, meta keys con prefijo `fu_`):

| Campo | Tipo | Descripción |
|---|---|---|
| `fu_cancha_direccion` | Texto | Dirección completa |
| `fu_cancha_lat` | Número | Latitud GPS |
| `fu_cancha_lng` | Número | Longitud GPS |
| `fu_cancha_tipo` | Select | `futbol5` / `futbol7` / `futbol11` |
| `fu_cancha_precio_hora` | Número | Precio base en ARS |
| `fu_cancha_fotos` | Galería | Array de IDs de adjuntos WP |
| `fu_cancha_telefono` | Texto | Teléfono de contacto |
| `fu_cancha_dueno_id` | Número | ID de `wp_users` (dueño) |
| `fu_cancha_horarios` | Textarea (JSON) | Horarios por día de semana |
| `fu_cancha_activa` | True/False | 1 = visible, 0 = pendiente de aprobación |

#### CPT `reservas`

Meta fields nativos (mismas convenciones que `canchas` — prefijo `fu_` por §22.4, idioma español por §22.6 / `02-GUIAS-TECNICAS.md §5`):

| Campo | Tipo | Descripción |
|---|---|---|
| `fu_reserva_cancha_id` | Número | ID del post cancha |
| `fu_reserva_jugador_id` | Número | ID de `wp_users` (jugador) |
| `fu_reserva_fecha` | Fecha | Fecha de la reserva |
| `fu_reserva_hora_inicio` | Texto | Ej: `19:00` |
| `fu_reserva_hora_fin` | Texto | Ej: `20:00` |
| `fu_reserva_monto` | Número | Monto total en ARS |
| `fu_reserva_estado` | Select | `pendiente` / `confirmado` / `cancelado` / `reembolsado` |
| `fu_reserva_mp_preference_id` | Texto | ID de preferencia MercadoPago |
| `fu_reserva_mp_payment_id` | Texto | ID de pago (llega por webhook) |

### §2.3 Tabla custom `wp_falta_uno_slots`

Se crea en la activación del plugin con `dbDelta()`. Almacena la disponibilidad horaria de cada cancha. Es una tabla dedicada (no `wp_postmeta`) porque los queries de disponibilidad ("¿qué horarios están libres para esta cancha en esta fecha?") son mucho más eficientes así.

| Campo | Tipo | Descripción |
|---|---|---|
| `id` | BIGINT AUTO PK | Clave primaria |
| `cancha_id` | BIGINT | FK → `wp_posts.ID` |
| `fecha` | DATE | Fecha del slot |
| `hora_inicio` | TIME | Ej: `19:00:00` |
| `hora_fin` | TIME | Ej: `20:00:00` |
| `estado` | ENUM | `disponible` / `reservado` / `bloqueado` |
| `reserva_id` | BIGINT NULL | FK → `wp_posts.ID` (NULL si libre) |
| `precio` | DECIMAL(10,2) | Precio del slot en ARS |

### §2.4 Roles de usuario

Se registran con `add_role()` en la activación del plugin.

| Rol | Capacidades clave | Descripción |
|---|---|---|
| `jugador` | `read`, `fu_crear_reservas` | Busca canchas y hace reservas |
| `dueno_cancha` | `read`, `fu_gestionar_canchas_propias`, `fu_ver_reservas_propias` | Publica y gestiona sus canchas |
| `administrator` (WP nativo) | Todo | Modera canchas, ve reportes |

El rol se asigna automáticamente al registrarse según qué opción elige el usuario ("Quiero reservar" → jugador / "Soy dueño de cancha" → dueño_cancha).

### §2.5 Flujo principal: reserva con pago

1. Jugador entra a la home → busca por zona o usa el mapa.
2. Ve ficha de cancha con calendario FullCalendar.js.
3. Toca un slot disponible → AJAX consulta `wp_falta_uno_slots` en tiempo real.
4. Si no está logueado → redirige a login/registro (asigna rol `jugador`).
5. Confirma la reserva → el plugin crea el CPT `reserva` en estado `pendiente`.
6. El plugin crea la preferencia en MercadoPago y redirige al Checkout Pro.
7. El usuario paga en MercadoPago → MercadoPago llama al webhook IPN.
8. El webhook actualiza la reserva a `confirmado` y bloquea el slot.
9. Se envían emails automáticos al jugador y al dueño con `wp_mail()`.

### §2.6 Naming convention

Decidido el 2026-05-19. Justificación completa en `00b-MAESTRO-HISTORIAL.md §22.1`.

| Elemento | Valor | Por qué |
|---|---|---|
| Marca comercial | Falta Uno | Lo que ve el usuario |
| Carpeta WP raíz | `C:\xampp\htdocs\falta-uno\` | Convención WP: hyphen para multi-palabra |
| Carpeta docs | `C:\Proyectos\Falta Uno\Docs\` | Por preferencia de Manu (mantener nombre con espacio) |
| Slug plugin | `falta-uno` | `wp-content/plugins/falta-uno/` |
| Archivo principal plugin | `falta-uno.php` | Mismo nombre que la carpeta |
| Slug theme | `falta-uno` | `wp-content/themes/falta-uno/` |
| Text domain | `falta-uno` | Coincide con slug |
| Prefijo funciones | `fu_` | Corto, fácil grep. Patrón Yoast (`wpseo_`), WooCommerce (`wc_`) |
| Prefijo clases | `FU_` | Ej: `FU_Reservas`, `FU_Pagos`, `FU_CPT_Canchas` |
| Prefijo opciones WP | `fu_` | `get_option('fu_mp_access_token')` |
| Tabla custom | `wp_falta_uno_slots` | MySQL no admite guión en nombres de tabla sin backticks → underscore |
| Base de datos | `falta_uno` | Mismo motivo |
| Repo Git código | `falta-uno` | Convención GitHub |
| Repo Git docs | `falta-uno-docs` | Convención GitHub |
| Dominio (a definir) | `faltauno.com.ar` (sugerido) | Sin guión por convención de dominios |

### §2.7 Decisiones de diseño explícitas

- **Sin app móvil.** Responsive nativo. El foco es que funcione bien en navegador del celular.
- **Sin React ni Vue.** JavaScript vanilla + AJAX nativo de WP. Más simple para mantener.
- **Sin Supabase ni otras DBs.** MySQL nativo de WP + 1 tabla custom (`wp_falta_uno_slots`).
- **Sin diseño custom (por ahora).** Bootstrap 5 utilities only. El theme custom hace el scaffolding, Bootstrap se ocupa de lo visual.
- **Pagos solo por MercadoPago.** Sin otras pasarelas en el MVP.
- **Los dueños no tocan el WP admin.** Todas las acciones del dueño (cargar cancha, ver reservas, bloquear horarios) son desde frontend.
- **Canchas requieren aprobación del admin** antes de ser visibles.

---

## §3 — Índice navegable de documentos

Documentos canon en `C:\Proyectos\Falta Uno\Docs\`:

| Archivo | Contenido | Cuándo consultarlo |
|---|---|---|
| `00-ARRANQUE.md` | Reglas operativas + identidad operador + naming rápido | Al inicio de cada sesión |
| `00-MAESTRO.md` (este) | Contexto + arquitectura + índice | Al inicio de cada sesión (obligatorio) |
| `00b-MAESTRO-HISTORIAL.md` | §22 — aprendizajes / decisiones revertidas / casos | Cuando hay una decisión arquitectónica que parece equivocada o se quiere entender el "por qué" de algo |
| `01-ESTADO-ACTUAL.md` | Qué está hecho, qué está pendiente, día del plan | Al inicio de cada sesión, después del MAESTRO |
| `02-GUIAS-TECNICAS.md` | Procedimientos operativos: deploy, SMTP, MercadoPago sandbox, cache-bust, etc. | Cuando hay que ejecutar una operación específica documentada |
| `03-INVENTARIO-TECNICO.md` | Mapa de archivos del plugin y theme | Cuando hay que ubicar una función o entender qué hace cada archivo |
| `05-ROADMAP.md` | Backlog ordenado: Fase 2, Fase 3, ideas futuras | Cuando se discute qué hacer después del MVP |

> *El slot `04-` queda reservado para un futuro doc (probablemente `04-BACKLOG-OPERATIVO.md` o similar — definir cuando aparezca la necesidad).*

Material histórico de referencia (no canon):

- `C:\Proyectos\Falta Uno\arranque-falta-uno.md` — bootstrap de primera sesión (no se vuelve a usar).
- `C:\Proyectos\Falta Uno\briefing-desarrollador.md` — briefing técnico inicial del proyecto (referencia de origen, autor histórico — Falta Uno no tiene dev externo: el código sale de Claude+Manu, ver `§22.7`).
- `C:\Proyectos\Falta Uno\futbol\` — mockups HTML, doc de onboarding de canchas piloto, sesión 25/abril. Material previo, solo referencia.

---

### §2.8 — Estructura real del plugin y del theme (snapshot v0.1.0, 2026-05-23)

Creada en la sesión 2026-05-23 (`§22.8`). Snapshot de bootstrap: lo mínimo para que el sitio sea navegable con contenido real. El detalle archivo por archivo (qué hace cada uno, qué función vive en qué archivo) vive en `03-INVENTARIO-TECNICO.md`.

**Plugin `falta-uno` v0.1.0** (`wp-content/plugins/falta-uno/`, tag `fu-plugin-v0.1.0`):

```
falta-uno/
├── falta-uno.php              # bootstrap: header WP + constantes + require + hooks de activación/desactivación
└── includes/
    └── class-fu-plugin.php    # clase singleton FU_Plugin: registra CPTs canchas (público, archive /canchas/) y reservas (interno) en el hook init
```

Lo que **no está todavía** y se va a sumar en los Días 3-4: meta boxes nativos para los meta keys `fu_cancha_*` y `fu_reserva_*` (§2.2), helper `FU_Meta`, roles `jugador` y `dueno_cancha` con `add_role()`, tabla `wp_falta_uno_slots` con `dbDelta()`, clase `FU_Activator` que orqueste todo eso al activar.

**Theme `falta-uno` v0.1.0** (`wp-content/themes/falta-uno/`, tag `fu-theme-v0.1.0`):

```
falta-uno/
├── style.css            # header WP + override --bs-primary al verde #00c850 (excepción aceptada §22.8)
├── functions.php        # supports + menus + enqueue Bootstrap 5.3.3 CDN + Bootstrap Icons 1.11.3 + filemtime cache-bust + admin bar oculta en frontend
├── index.php            # fallback genérico
├── header.php           # navbar Bootstrap sticky con brand, menú, botones Ingresar/Registrarse
├── footer.php           # footer oscuro 3 columnas + copyright
├── front-page.php       # home: hero + buscador (name=s) + grid de 6 canchas destacadas + CTA dueños
├── archive-canchas.php  # listado /canchas/ con breadcrumb, buscador, filtros UI de tipo (placeholder hasta Día 3), grid + paginación
├── single-canchas.php   # ficha de cancha con galería placeholder + datos en "—" hasta meta boxes + contenedor #fu-calendario-cancha listo para FullCalendar (Día 6)
├── page-login.php       # Template Name "Login (Falta Uno)" — form Bootstrap + nonce fu_login (lógica handler Día 8)
├── page-registro.php    # Template Name "Registro (Falta Uno)" — form con radio jugador/dueno_cancha + nonce fu_registro
└── page-mi-panel.php    # Template Name "Mi panel (Falta Uno)" — dashboard cards condicionado por rol, gated por is_user_logged_in()
```

Stack visual: Bootstrap 5.3.3 + Bootstrap Icons 1.11.3, ambos desde CDN jsdelivr. **Único CSS propio:** ~30 líneas de override de variables en `style.css` (`--bs-primary` y derivados `.btn-primary`/`.btn-outline-primary`/links). Esto es excepción explícita al "cero CSS custom" del `§2.7` y `§22.2`, equiparada con las excepciones ya previstas para FullCalendar y Google Maps. Cualquier nuevo CSS custom requiere su propia §22.X.

**Páginas WP creadas (manualmente en wp-admin) para que los page templates se carguen:**

- `/login/` → plantilla "Login (Falta Uno)"
- `/registro/` → plantilla "Registro (Falta Uno)"
- `/mi-panel/` → plantilla "Mi panel (Falta Uno)"

**Lo que el theme tiene como placeholder y se completa en días posteriores:**

| Placeholder visible hoy | Se completa en |
|---|---|
| Filtros de tipo en home + archive (chips cambian URL pero no filtran) | Día 3 (meta boxes `fu_cancha_tipo` + hook `pre_get_posts`) |
| Datos de single-canchas en "—" (dirección, precio, contacto) | Día 3 (meta boxes) |
| Contenedor `#fu-calendario-cancha` vacío con mensaje "próximamente" | Día 6 (FullCalendar.js + endpoint AJAX `fu_disponibilidad_cancha`) |
| Mapas en home y single (íconos estáticos) | Día 7 (Google Maps JS API) |
| Forms de login y registro sin handler (envían POST sin nada que lo procese) | Día 8 (handler con `wp_signon()` + `wp_create_user()` + `add_role()`) |
| Cards de mi-panel con texto "Se conecta en el Día X" | Día 7 (mis canchas), Día 9 (mis reservas), post-MVP (ingresos) |

---

*Próxima edición esperada: cuando se cierre el Día 3 del Plan Fase 1 (meta boxes + roles + helper `FU_Meta`), actualizar §2.8 con los archivos nuevos del plugin y marcar qué placeholders del theme quedaron conectados.*
