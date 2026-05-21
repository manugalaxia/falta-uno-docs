# 02 — GUÍAS TÉCNICAS

*Procedimientos operativos puntuales: cómo hacer algo concreto en Falta Uno. Cada guía vive en su propia sección numerada. Para discusiones de "por qué decidimos X" ir al HISTORIAL, no acá.*

*Última actualización: 2026-05-21.*

---

## Índice

| Guía | Estado | Cuándo se consulta |
|---|---|---|
| §1 Convención visual: Bootstrap 5 utilities | ✅ Escrita | Al armar cualquier template/componente del theme |
| §2 Cache-bust con `filemtime()` | ✅ Escrita | Al encolar cualquier CSS/JS del plugin o theme |
| §3 Naming convention (rápido) | ✅ Escrita | Al crear archivos/funciones/clases nuevas |
| §4 Validación PHP con `php -l` | ✅ Escrita | Antes de entregar cualquier `.php` |
| §5 Convención de idioma en código y datos | ✅ Escrita | Al nombrar cualquier campo, meta key, rol, función, hook o variable nueva |
| §6 Setup local de XAMPP desde cero | ⚪ Pendiente | Si un dev nuevo arma el entorno |
| §7 Helper `FU_Meta::render_field()` para meta boxes nativos | ⚪ Pendiente | Cuando se arme el render de los meta fields (Día 3) |
| §8 SMTP local con WP Mail SMTP | ⚪ Pendiente | Día 10 (testing emails) |
| §9 MercadoPago sandbox: credenciales y testing | ⚪ Pendiente | Inicio de Fase 2 (integración MP) |
| §10 Deploy a producción (provider TBD) | ⚪ Pendiente | Día 10 |
| §11 Git workflow del proyecto | ⚪ Pendiente | Cuando se inicialicen los repos |
| §12 Backup y restore de DB local | ⚪ Pendiente | Cuando aparezca un caso |
| §13 Debug: WP_DEBUG, debug.log, Query Monitor | ⚪ Pendiente | Cuando aparezca el primer bug serio |

---

## §1 — Convención visual: Bootstrap 5 utilities only

**Regla:** Todo el frontend de Falta Uno usa **Bootstrap 5 utility classes**. Sin SCSS custom, sin overrides, sin CSS propio salvo lo mínimo imprescindible.

### Qué sí

- Clases utility de Bootstrap para layout (`container`, `row`, `col-md-6`, `d-flex`, `justify-content-between`, etc.).
- Componentes de Bootstrap (`btn`, `card`, `navbar`, `modal`, `alert`, `form-control`, etc.).
- Spacing y typography utilities (`mb-3`, `py-4`, `fs-5`, `text-muted`, etc.).
- Responsive utilities (`d-none d-md-block`, `col-12 col-md-6`, etc.).

### Qué no

- ❌ Sobreescribir variables SCSS de Bootstrap.
- ❌ Compilar Bootstrap desde fuente con un build step (Webpack, Sass).
- ❌ Escribir CSS custom para "personalizar un poco" un componente Bootstrap. Si no se puede con utilities, se cambia el approach.
- ❌ Mezclar otros frameworks (Tailwind, Bulma, etc.).

### Excepciones permitidas (mínimas)

Único CSS custom permitido en el theme, en `wp-content/themes/falta-uno/assets/css/custom.css`:

- Estilos mínimos para que FullCalendar.js se integre visualmente (overrides puntuales de FullCalendar, no de Bootstrap).
- Ajustes del contenedor del mapa de Google Maps (`height`, `width`, posicionamiento).
- *Nada más sin discusión previa y entrada en HISTORIAL.*

### Cómo encolar Bootstrap

Desde `wp-content/themes/falta-uno/functions.php`:

```php
function fu_enqueue_bootstrap() {
    // CSS desde CDN (MVP). Local cuando vayamos a producción.
    wp_enqueue_style(
        'bootstrap-5',
        'https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css',
        [],
        '5.3.3'
    );

    // JS bundle (incluye Popper)
    wp_enqueue_script(
        'bootstrap-5',
        'https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js',
        [],
        '5.3.3',
        true // en footer
    );
}
add_action( 'wp_enqueue_scripts', 'fu_enqueue_bootstrap' );
```

**Antes de deploy a producción:** descargar Bootstrap a `wp-content/themes/falta-uno/assets/vendor/bootstrap/` y cambiar las URLs a locales. El versionado pasa a `filemtime()` (ver §2).

---

## §2 — Cache-bust con `filemtime()`

**Regla:** Todo encolado de CSS/JS propio (del plugin o del theme) usa `filemtime(__FILE__)` como versión, no un string hardcodeado.

**Por qué:** Cuando se modifica un archivo, la versión cambia automáticamente y el navegador refresca el cache. Sin esto, dos horas perdidas con "subí el archivo pero veo lo viejo".

### Patrón canónico

```php
function fu_enqueue_theme_assets() {
    $css_path = get_stylesheet_directory() . '/assets/css/custom.css';
    $css_uri  = get_stylesheet_directory_uri() . '/assets/css/custom.css';

    wp_enqueue_style(
        'fu-custom',
        $css_uri,
        ['bootstrap-5'], // depende de Bootstrap
        filemtime( $css_path ) // ← cache-bust automático
    );

    $js_path = get_stylesheet_directory() . '/assets/js/main.js';
    $js_uri  = get_stylesheet_directory_uri() . '/assets/js/main.js';

    wp_enqueue_script(
        'fu-main',
        $js_uri,
        ['jquery'],
        filemtime( $js_path ),
        true
    );
}
add_action( 'wp_enqueue_scripts', 'fu_enqueue_theme_assets' );
```

**Para Bootstrap (vendor)** se sigue usando un string ('5.3.3') porque no lo editamos.

---

## §3 — Naming convention (rápido)

Referencia completa en `00-MAESTRO.md §2.6`. Resumen para tener a mano al crear cosas nuevas:

| Tipo | Ejemplo |
|---|---|
| Función | `fu_get_slots_disponibles( $cancha_id, $fecha )` |
| Clase | `class FU_Reservas { ... }` |
| Opción WP | `get_option( 'fu_mp_access_token' )` |
| Hook custom | `do_action( 'fu_reserva_confirmada', $reserva_id )` |
| Filtro custom | `apply_filters( 'fu_precio_slot', $precio, $slot )` |
| Constante | `FU_PLUGIN_DIR`, `FU_PLUGIN_VERSION` |
| Tabla custom | `wp_falta_uno_slots` |
| Slug CPT | `canchas`, `reservas` (en español, singular conceptual aunque plural gramatical) |
| Meta key (CPT) | `fu_cancha_direccion`, `fu_reserva_estado` (snake_case, en español, prefijo `fu_` por §22.4) |
| ID HTML / clase CSS custom | `id="fu-calendario-cancha-{ID}"`, `class="fu-mapa-home"` (solo si no se puede usar Bootstrap) |
| Endpoint REST | `/wp-json/falta-uno/v1/webhook` |
| Action AJAX | `wp_ajax_fu_get_slots`, `wp_ajax_nopriv_fu_get_slots` |

---

## §4 — Validación PHP con `php -l`

**Regla:** Todo `.php` se valida con `php -l` antes de entregarse. Cero excepciones.

### Local (XAMPP)

```bash
# Validar un archivo
C:\xampp\php\php.exe -l C:\xampp\htdocs\falta-uno\wp-content\plugins\falta-uno\falta-uno.php

# Validar todos los .php del plugin
for /R C:\xampp\htdocs\falta-uno\wp-content\plugins\falta-uno %f in (*.php) do @C:\xampp\php\php.exe -l "%f"
```

Salida esperada: `No syntax errors detected in <archivo>`.

Si aparece `PHP Parse error:` o `PHP Warning:`, **no se entrega hasta resolverlo.**

### En el contenedor de Cowork

Si Claude genera un archivo PHP largo en el contenedor antes de entregarlo, validarlo ahí con `php -l` antes del `present_files`.

---

## §5 — Convención de idioma en código y datos

**Regla:** Los datos y nombres del dominio del negocio (CPTs, meta keys, roles, estados, mensajes de UI) van **en español**. Los hooks, funciones y APIs nativas de WordPress quedan **en inglés** porque vienen así de WP y no se pueden traducir.

Decisión cerrada el 2026-05-21 — detalle en `00b-MAESTRO-HISTORIAL.md §22.6`.

### Aplicación por categoría

| Categoría | Idioma | Ejemplos |
|---|---|---|
| Slug de CPT | Español | `canchas`, `reservas` |
| Meta keys de CPT | Español, prefijo `fu_` | `fu_cancha_direccion`, `fu_reserva_estado`, `fu_cancha_precio_hora` |
| Slug de rol | Español | `jugador`, `dueno_cancha` |
| Valores de estado | Español | `pendiente`, `confirmado`, `cancelado`, `reembolsado` |
| Capabilities custom | Español, prefijo `fu_` | `fu_crear_reservas`, `fu_gestionar_canchas_propias` |
| Action AJAX custom | Español, prefijo `fu_` | `fu_get_slots`, `fu_crear_reserva`, `fu_webhook_mp` |
| Endpoint REST custom | Español | `/wp-json/falta-uno/v1/webhook`, `/wp-json/falta-uno/v1/slots` |
| Hook custom (`do_action`) | Español, prefijo `fu_` | `fu_reserva_confirmada`, `fu_slot_bloqueado` |
| Filtro custom (`apply_filters`) | Español, prefijo `fu_` | `fu_precio_slot`, `fu_horarios_disponibles` |
| Mensajes de UI / errores | Español rioplatense | "Reservá tu turno", "Ese horario ya está ocupado", "Cancha pendiente de aprobación" |
| Constantes del plugin | Inglés convencional + `FU_` | `FU_PLUGIN_DIR`, `FU_PLUGIN_URL`, `FU_PLUGIN_VERSION` |
| Funciones del plugin | Español, prefijo `fu_` | `fu_get_slots_disponibles()`, `fu_crear_reserva()`, `fu_enqueue_assets()` |
| Clases del plugin | Inglés-PHP convencional + `FU_` | `FU_Plugin`, `FU_CPT`, `FU_Reservas`, `FU_Pagos` (los nombres de archivos PHP siguen mayúsculas-camel-case) |
| Tabla custom | Mixto | `wp_falta_uno_slots` (prefijo WP `wp_` + dominio `falta_uno`) |
| Columnas de tabla custom | Español | `cancha_id`, `fecha`, `hora_inicio`, `hora_fin`, `estado`, `reserva_id`, `precio` |
| Hooks nativos WP | Inglés (no se cambia) | `add_action`, `wp_enqueue_scripts`, `save_post`, `init` |
| Funciones nativas WP | Inglés (no se cambia) | `get_post_meta`, `wp_insert_post`, `wp_mail`, `register_post_type` |
| Variables locales / parámetros | Español | `$cancha_id`, `$fecha`, `$hora_inicio`, `$reserva` |

### Convenciones particulares

- **Sin acentos en identificadores de código.** PHP y SQL aceptan UTF-8 en identificadores pero no es portable. Usar `dueno_cancha` (no `dueño_cancha`), `direccion` (no `dirección`), `telefono` (no `teléfono`).
- **Sin caracteres especiales en slugs.** Convención WP: solo `[a-z0-9-_]`.
- **Singular vs plural en meta keys.** Las meta keys van en singular (`fu_cancha_direccion`, no `fu_canchas_direccion`). Los CPTs van en plural gramatical (`canchas`, `reservas`) siguiendo la convención WP estándar.
- **Comentarios en el código:** español. Es código que va a leer principalmente Manu y Claude en español, no hay valor en traducir.
- **Strings traducibles con `__()`.** Aunque el text domain `falta-uno` está reservado, en el MVP no se traduce nada — los strings van directos en español. Si en el futuro hay i18n real, ahí se envuelven con `__( 'texto', 'falta-uno' )`.

---

## §6 a §13

⚪ *Pendientes.* Se escriben cuando aparece la necesidad concreta. Hasta entonces este doc tiene solo el índice arriba.
