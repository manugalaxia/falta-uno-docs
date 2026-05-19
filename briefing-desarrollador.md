# FulboApp — Briefing para el Desarrollador

**Fecha:** 13 de Mayo 2026  
**Lanzamiento MVP:** 30 de Junio 2026  
**Stack:** WordPress custom (tema + plugin)  
**Equipo:** 1 desarrollador WordPress part-time

---

## Qué es FulboApp

Plataforma web para **reservar canchas de fútbol** en Argentina. Los jugadores buscan canchas disponibles, eligen un horario y pagan online. Los dueños de canchas publican sus espacios y gestionan sus reservas. Todo desde el navegador, sin app móvil.

---

## Stack tecnológico

| Capa | Tecnología |
|---|---|
| CMS | WordPress (última versión estable) |
| Backend | PHP 8.x + plugin custom |
| Base de datos | MySQL 8.x (tablas nativas WP + 1 tabla custom) |
| Frontend | HTML5, CSS3, JavaScript vanilla |
| Templates | WordPress Template Hierarchy (PHP) |
| Calendario UI | FullCalendar.js |
| Mapas | Google Maps JS API + Geocoding API |
| Pagos | MercadoPago PHP SDK v3 (Checkout Pro) |
| Emails | wp_mail() + SMTP plugin (SendGrid o Gmail) |
| Tareas programadas | WP Cron |
| Campos custom | Advanced Custom Fields (ACF Pro) |
| Hosting | SiteGround / Cloudways / Hostinger (PHP 8 + MySQL 8 + SSL) |

---

## Estructura del proyecto WordPress

El desarrollo consiste en **dos piezas custom**:

### 1. Tema custom (`fulboapp-theme/`)

Tema propio desde cero (o tema hijo). Contiene:

- `functions.php` — enqueue de scripts y estilos del tema
- `index.php`, `header.php`, `footer.php` — estructura base
- `front-page.php` — home con buscador y mapa
- `archive-canchas.php` — listado de canchas con filtros
- `single-canchas.php` — ficha de cancha + calendario de disponibilidad
- `page-mi-panel.php` — panel del dueño de cancha
- `page-reserva.php` — confirmación de reserva
- `assets/css/`, `assets/js/` — estilos y scripts del tema

### 2. Plugin custom (`fulboapp-plugin/`)

```
fulboapp-plugin/
  fulboapp.php                  ← bootstrap / registra hooks
  includes/
    class-cpt.php               ← registra CPT: canchas, reservas
    class-roles.php             ← roles custom: jugador, dueño_cancha
    class-reservas.php          ← lógica de booking + gestión de slots
    class-pagos.php             ← integración MercadoPago PHP SDK
    class-notificaciones.php    ← emails via wp_mail()
    class-mapas.php             ← helpers Google Maps
  admin/
    panel-dueno.php             ← panel frontend del dueño
    panel-admin.php             ← panel global del admin
  ajax/
    ajax-slots.php              ← devuelve horarios disponibles (JSON)
    ajax-reservar.php           ← crea reserva + inicia pago MP
    ajax-webhook.php            ← recibe notificaciones IPN de MercadoPago
  assets/
    css/fulboapp.css
    js/calendario.js            ← FullCalendar.js + llamadas AJAX
    js/mapas.js                 ← Google Maps embed
```

---

## Custom Post Types

### CPT: `canchas`

Registrado con `register_post_type('canchas', [...])`.  
El título del post es el nombre de la cancha.

**Campos ACF:**

| Campo | Tipo | Descripción |
|---|---|---|
| `cancha_direccion` | Texto | Dirección completa |
| `cancha_lat` | Número | Latitud GPS |
| `cancha_lng` | Número | Longitud GPS |
| `cancha_tipo` | Select | `futbol5` / `futbol7` / `futbol11` |
| `cancha_precio_hora` | Número | Precio base en ARS |
| `cancha_fotos` | Galería | Array de IDs de adjuntos WP |
| `cancha_telefono` | Texto | Teléfono de contacto |
| `cancha_dueno_id` | Número | ID de wp_users (dueño) |
| `cancha_horarios` | Textarea (JSON) | Horarios por día de semana |
| `cancha_activa` | True/False | 1 = visible, 0 = pendiente de aprobación |

### CPT: `reservas`

**Campos ACF:**

| Campo | Tipo | Descripción |
|---|---|---|
| `reserva_cancha_id` | Número | ID del post cancha |
| `reserva_jugador_id` | Número | ID de wp_users (jugador) |
| `reserva_fecha` | Fecha | Fecha de la reserva |
| `reserva_hora_inicio` | Texto | Ej: `19:00` |
| `reserva_hora_fin` | Texto | Ej: `20:00` |
| `reserva_monto` | Número | Monto total en ARS |
| `reserva_estado` | Select | `pendiente` / `confirmado` / `cancelado` / `reembolsado` |
| `reserva_mp_preference_id` | Texto | ID de preferencia MercadoPago |
| `reserva_mp_payment_id` | Texto | ID de pago (llega por webhook) |

---

## Tabla custom de base de datos

Se crea en la activación del plugin con `dbDelta()`.

### `wp_fulboapp_slots`

Almacena la disponibilidad horaria de cada cancha. Esta tabla es necesaria porque los queries de disponibilidad (¿qué horarios están libres para esta cancha en esta fecha?) son más eficientes con una tabla dedicada que con `wp_postmeta`.

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

---

## Roles de usuario

Se registran con `add_role()` en la activación del plugin.

| Rol | Capacidades clave | Descripción |
|---|---|---|
| `jugador` | `read`, `crear_reservas` | Busca canchas y hace reservas |
| `dueno_cancha` | `read`, `gestionar_canchas_propias`, `ver_reservas_propias` | Publica y gestiona sus canchas |
| `administrator` (WP nativo) | Todo | Modera canchas, ve reportes |

> El rol se asigna automáticamente al registrarse según qué opción elige el usuario ("Quiero reservar" → jugador / "Soy dueño de cancha" → dueño_cancha).

---

## Flujo principal: Reserva con pago

1. Jugador entra a la home → busca por zona o usa el mapa
2. Ve ficha de cancha con calendario FullCalendar.js
3. Toca un slot disponible → AJAX consulta `wp_fulboapp_slots` en tiempo real
4. Si no está logueado → redirige a login/registro (asigna rol jugador)
5. Confirma la reserva → el plugin crea el CPT reserva en estado `pendiente`
6. El plugin crea la preferencia en MercadoPago y redirige al Checkout Pro
7. El usuario paga en MercadoPago → MercadoPago llama al webhook IPN
8. El webhook actualiza la reserva a `confirmado` y bloquea el slot
9. Se envían emails automáticos al jugador y al dueño con `wp_mail()`

### Integración MercadoPago (resumen técnico)

```php
// Crear preferencia
$sdk = new MercadoPago\SDK();
$pref = new Preference();
$pref->items = [['title' => 'Cancha F5 - 19hs', 'quantity' => 1, 'unit_price' => 5000]];
$pref->back_urls = ['success' => site_url('/gracias/'), 'failure' => site_url('/error/')];
$pref->notification_url = site_url('/wp-json/fulboapp/v1/webhook');
$pref->save();
// Redirigir al usuario a $pref->init_point

// Webhook IPN (REST API endpoint de WordPress)
register_rest_route('fulboapp/v1', '/webhook', [
    'methods'             => 'POST',
    'callback'            => 'fulboapp_handle_webhook',
    'permission_callback' => '__return_true',
]);

function fulboapp_handle_webhook($request) {
    $payment_id = $request['data']['id'];
    $payment    = Payment::find_by_id($payment_id);
    if ($payment->status === 'approved') {
        fulboapp_confirmar_reserva($payment->metadata->reserva_id);
        fulboapp_enviar_emails($payment->metadata->reserva_id);
    }
}
```

---

## Plan de trabajo — Fase 1 (13 al 22 de Mayo)

| Día | Fecha | Tarea | Horas est. |
|---|---|---|---|
| 1 | 13 May | Setup hosting + WordPress + SSL + Git repo + ACF Pro | 3–4 hs |
| 2 | 14 May | Bootstrap del plugin: activación, autoloader, CPTs canchas y reservas | 4–5 hs |
| 3 | 15 May | Roles custom + todos los ACF fields para canchas y reservas | 3–4 hs |
| 4 | 16 May | Tabla `wp_fulboapp_slots` con dbDelta + funciones de generación y consulta de slots | 4–5 hs |
| 5 | 17 May | Tema custom: header, footer, home con buscador, archive-canchas, single-canchas (mobile-first) | 5–6 hs |
| 6 | 18 May | Calendario FullCalendar.js en ficha de cancha + endpoint AJAX de disponibilidad | 4–5 hs |
| 7 | 19 May | Google Maps en home y ficha + formulario de carga de cancha (frontend, sin WP admin) | 5–6 hs |
| 8 | 20 May | Login/registro custom (sin wp-login.php) + asignación de roles + página /mi-panel/ | 5–6 hs |
| 9 | 21 May | Flujo completo de reserva sin pago: elige slot → crea reserva → bloquea slot → email | 5–6 hs |
| 10 | 22 May | Testing end-to-end + ajustes mobile + SMTP + deploy a producción | 4–5 hs |

### Entregables al finalizar Fase 1

- WordPress en producción con SSL y dominio
- Plugin `fulboapp-plugin` activo con CPTs y roles
- Tema custom: home, listado y ficha de cancha
- Mapa Google Maps con pines de canchas
- Calendario FullCalendar.js con disponibilidad en tiempo real
- Formulario de carga de cancha (frontend, para dueños)
- Flujo de reserva end-to-end (sin pago)
- Panel del dueño: listado de reservas y stats básicos
- Emails de confirmación con `wp_mail()`

---

## Fase 2 (23 May al 12 Jun) — Vista previa

Lo que viene después de Fase 1:

- Integración completa MercadoPago Checkout Pro + webhook IPN
- Panel admin global (moderación de canchas, reportes)
- Historial de reservas del jugador
- Filtros avanzados en el buscador (precio, tipo, zona)
- Bloqueo de horarios por el dueño desde su panel
- Recordatorios automáticos por email (WP Cron, 24hs antes de la reserva)
- Optimización de performance (caching, lazy load imágenes)

---

## Notas y decisiones de diseño

- **Sin app móvil.** El sitio debe ser completamente responsive. El foco es que funcione bien en el navegador del celular.
- **Sin React ni Vue.** Toda la interactividad se hace con JavaScript vanilla + AJAX de WordPress. Más simple para mantener.
- **Sin Supabase.** La base de datos es MySQL nativa de WordPress. Se agrega solo 1 tabla custom para los slots.
- **Pagos exclusivamente por MercadoPago.** Sin integrar otras pasarelas por ahora.
- **Los dueños no tocan el WP admin.** Todas las acciones del dueño (cargar cancha, ver reservas, bloquear horarios) se hacen desde páginas frontend del sitio.
- **Las canchas requieren aprobación del admin** antes de ser visibles en el sitio.

---

## Accesos y recursos

- Repositorio Git: *(a definir)*
- Credenciales de hosting: *(a definir)*
- API Key Google Maps: *(a definir)*
- Credenciales MercadoPago: *(a definir — usar sandbox para desarrollo)*
- ACF Pro: *(licencia a gestionar)*

---

*Documento generado el 13 de Mayo 2026 — FulboApp v0.1*
