# 03 — INVENTARIO TÉCNICO

*Mapa de archivos del plugin `falta-uno` y del theme `falta-uno`. Sirve para ubicar funciones, clases y assets sin tener que abrir el IDE. Se actualiza cada vez que se crea, mueve o renombra un archivo del código.*

*Última actualización: 2026-05-23. Plugin v0.1.0 (CPTs registrados, sin meta boxes ni tabla custom) + theme v0.1.0 (11 templates, Bootstrap utilities + verde primario) creados en la sesión del 2026-05-23. Detalle: `00b-MAESTRO-HISTORIAL.md §22.8`.*

---

## Raíz WordPress local

```
C:\xampp\htdocs\falta-uno\
├── wp-config.php                       # OK Configurado: DB falta_uno, WP_HOME y WP_SITEURL = localhost/falta-uno
├── wp-content\
│   ├── plugins\
│   │   ├── falta-uno\                  # OK v0.1.0 (sesión 2026-05-23, §22.8)
│   │   └── wp-mail-smtp\               # [a instalar] SMTP para wp_mail() — antes del Día 9
│   ├── themes\
│   │   └── falta-uno\                  # OK v0.1.0 (sesión 2026-05-23, §22.8)
│   └── uploads\                        # auto, lo maneja WP
└── (core de WordPress — no se toca, no se versiona)
```

Convención de marcadores: `OK` = creado y funcionando · `[a crear]` = previsto en el plan · `[a instalar]` = paquete de terceros pendiente.

---

## Plugin `falta-uno` v0.1.0

Ubicación: `wp-content/plugins/falta-uno/`. Header WP en `falta-uno.php`. Versión actual: `0.1.0`. Tag Git: `fu-plugin-v0.1.0`.

```
falta-uno/
├── falta-uno.php                       # OK Bootstrap: header WP + constantes FU_VERSION/FU_PATH/FU_URL/FU_FILE + require de class-fu-plugin + register_activation_hook
├── includes\
│   └── class-fu-plugin.php             # OK Clase singleton FU_Plugin. Registra CPTs `canchas` y `reservas` en `init`. Métodos estáticos activar()/desactivar() flushean rewrite rules
├── README.md                           # [a crear] descripción mínima para el repo
├── uninstall.php                       # [a crear] cleanup al borrar plugin (dropea wp_falta_uno_slots, borra opciones fu_*)
├── includes\ (resto)
│   ├── class-fu-activator.php          # [a crear — Día 3-4] crea tabla wp_falta_uno_slots, registra roles
│   ├── class-fu-deactivator.php        # [a crear] hooks de desactivación (no borrar datos)
│   ├── class-fu-cpt.php                # [a crear — opcional, hoy los CPTs los registra FU_Plugin] separar registro de CPTs cuando crezca
│   ├── class-fu-meta.php               # [a crear — Día 3] helper FU_Meta para meta boxes nativos de canchas y reservas
│   ├── class-fu-roles.php              # [a crear — Día 3] roles `jugador` y `dueno_cancha` con add_role()
│   ├── class-fu-reservas.php           # [a crear — Día 9] lógica de booking + gestión de slots
│   ├── class-fu-pagos.php              # [a crear — Fase 2] integración MercadoPago PHP SDK
│   ├── class-fu-notificaciones.php     # [a crear — Día 9] emails via wp_mail()
│   ├── class-fu-mapas.php              # [a crear — Día 7] helpers Google Maps (geocoding)
│   └── class-fu-helpers.php            # [a crear] utilidades varias (formateo fecha/hora ARS, etc.)
├── admin\
│   ├── panel-dueno.php                 # [a crear — Día 7-8] panel frontend del dueño de cancha
│   └── panel-admin.php                 # [a crear] panel global del admin (moderación, reportes)
├── ajax\
│   ├── ajax-slots.php                  # [a crear — Día 6] devuelve horarios disponibles (JSON) — endpoint `fu_disponibilidad_cancha`
│   ├── ajax-reservar.php               # [a crear — Día 9] crea reserva + inicia pago MP
│   └── ajax-webhook.php                # [a crear — Fase 2] recibe notificaciones IPN de MercadoPago
├── rest\
│   └── webhook-mp.php                  # [a crear — Fase 2] endpoint REST alternativo
└── assets\
    ├── css\
    │   └── falta-uno-admin.css         # [a crear — Día 7-8] estilos del panel frontend del dueño
    └── js\
        ├── calendario.js               # [a crear — Día 6] FullCalendar.js + llamadas AJAX al endpoint de slots
        ├── mapas.js                    # [a crear — Día 7] Google Maps embed + interacción
        └── reserva.js                  # [a crear — Día 9] flujo de reserva (selección slot → confirmación)
```

### Convenciones internas

- Toda clase en `includes/` sigue patrón `class-fu-{nombre}.php` (lowercase + hyphen) con clase `FU_{Nombre}` (PascalCase + underscore).
- Autoload simple: en `falta-uno.php` se hace un `require_once` explícito por cada clase. Sin Composer hasta que duela.
- Cada clase nueva se agrega también al inventario de arriba — esto no se hace solo.
- **Decisión sesión 2026-05-23:** mientras los CPTs sean solo dos y vacíos, los registra directamente `FU_Plugin::registrar_cpts()` sin separar en `class-fu-cpt.php`. Cuando aparezcan meta boxes (Día 3) se evalúa separar.

---

## Theme `falta-uno` v0.1.0

Ubicación: `wp-content/themes/falta-uno/`. Versión actual: `0.1.0`. Tag Git: `fu-theme-v0.1.0`. Stack visual: Bootstrap 5.3.3 desde CDN + Bootstrap Icons 1.11.3 desde CDN. Override CSS único: `--bs-primary` al verde `#00c850` (excepción §22.8).

```
falta-uno/
├── style.css                           # OK Header WP + override de --bs-primary al verde Falta Uno + ajustes puntuales de .btn-primary/.btn-outline-primary/links (~30 líneas)
├── functions.php                       # OK Supports (title-tag, post-thumbnails, custom-logo, html5) + register_nav_menus + enqueue Bootstrap CDN + cache-bust por filemtime + admin bar oculta en frontend
├── index.php                           # OK Fallback genérico con loop básico y paginación
├── header.php                          # OK Navbar Bootstrap sticky con brand "⚽ Falta Uno", menú principal, botones Ingresar/Registrarse en verde
├── footer.php                          # OK Footer oscuro 3 columnas (marca + plataforma + contacto) + copyright dinámico
├── front-page.php                      # OK Hero con buscador (name=s, action=/canchas/) + grid de 6 canchas destacadas + CTA dueños
├── archive-canchas.php                 # OK Breadcrumb + buscador + filtros UI de tipo (placeholder hasta Día 3) + grid responsive con paginación + estado vacío
├── single-canchas.php                  # OK Breadcrumb + galería (1 foto + 3 thumbs placeholder) + ficha de datos (tipo, dirección, precio, contacto en placeholders) + contenedor #fu-calendario-cancha listo para FullCalendar + sección "Sobre la cancha" + placeholder mapa
├── page-login.php                      # OK Template Name: "Login (Falta Uno)". Form Bootstrap email + password + recordar + nonce fu_login. Lógica de wp_signon() pendiente Día 8
├── page-registro.php                   # OK Template Name: "Registro (Falta Uno)". Form con radio jugador/dueno_cancha + nombre + email + teléfono + password + términos + nonce fu_registro. Lógica de wp_create_user() + add_role() pendiente Día 8
├── page-mi-panel.php                   # OK Template Name: "Mi panel (Falta Uno)". Saludo personalizado + cards condicionadas por rol (mis reservas / mis canchas / ingresos / mi cuenta) + estado no logueado con CTA login/registro
├── 404.php                             # [a crear] página de error
├── searchform.php                      # [a crear] form de búsqueda reusable (opcional)
├── sidebar.php                         # [a crear — opcional] probablemente no se use en MVP
├── page-reserva.php                    # [a crear — Día 9] confirmación de reserva post-pago
└── assets\
    ├── css\
    │   └── custom.css                  # [a crear — Día 6-7] overrides mínimos para FullCalendar y Google Maps
    ├── js\
    │   └── main.js                     # [a crear si hace falta] JS general del theme — lo específico (calendario/mapas/reserva) vive en el plugin
    ├── img\
    │   └── (logos, placeholders)       # [a poblar — depende de branding mínimo, decisión diferida #1]
    └── vendor\
        └── bootstrap\                  # [a poblar antes de prod] descarga local de Bootstrap para no depender de CDN
```

### Páginas WP creadas para los page templates

Los page templates (`page-login.php`, `page-registro.php`, `page-mi-panel.php`) solo se activan si en wp-admin existe una página que los referencie. Estado:

| Página WP | Slug | Plantilla asignada | Estado |
|---|---|---|---|
| Login | `/login/` | Login (Falta Uno) | OK creada en sesión 2026-05-23 |
| Registro | `/registro/` | Registro (Falta Uno) | OK creada en sesión 2026-05-23 |
| Mi panel | `/mi-panel/` | Mi panel (Falta Uno) | OK creada en sesión 2026-05-23 |

---

## Repos Git

Inicializados el 2026-05-19 — detalle de la decisión en `00b-MAESTRO-HISTORIAL.md §22.3`. Convención de versionado, tags y flujo de commits documentada en `MANUAL-TRABAJO-CON-CLAUDE.md §3`.

| Repo | Path local | Remoto GitHub | Qué contiene | `.gitignore` clave |
|---|---|---|---|---|
| `falta-uno` (código) | `C:\xampp\htdocs\falta-uno\` | `manugalaxia/falta-uno` (privado) | Plugin `falta-uno` + theme `falta-uno` | **Allowlist:** ignora `/*` y reabilita solo `wp-content/plugins/falta-uno/` y `wp-content/themes/falta-uno/`. Core de WP, plugins de terceros, `wp-config.php`, `vendor/`, `node_modules/` y logs quedan fuera por construcción |
| `falta-uno-docs` (docs) | `C:\Proyectos\Falta Uno\` | `manugalaxia/falta-uno-docs` (privado) | Carpeta `Docs/` + meta-docs (`MANUAL-TRABAJO-CON-CLAUDE.md`, `PROMPT-INICIAL-CLAUDE.md`) + material histórico (`futbol/`, `briefing-desarrollador.md`, `arranque-falta-uno.md`) | `Thumbs.db`, `.DS_Store`, `.vscode/`, `*.swp`, `*.swo` |

### Tags Git activos

| Tag | Repo | Commit | Qué marca |
|---|---|---|---|
| `fu-plugin-v0.1.0` | `falta-uno` | `a943c0d` | Plugin bootstrap mínimo (CPTs canchas y reservas registrados, sin meta boxes ni tabla) |
| `fu-theme-v0.1.0` | `falta-uno` | `a943c0d` | Theme completo con 11 templates, Bootstrap utilities, verde primario, admin bar oculta |

---

## Tabla de búsqueda rápida

| Función / clase | Archivo | Para qué |
|---|---|---|
| `FU_Plugin` (clase singleton) | `wp-content/plugins/falta-uno/includes/class-fu-plugin.php` | Punto de entrada del plugin. Registra CPTs en `init`, expone callbacks estáticos `activar()`/`desactivar()` para los hooks de WP |
| `FU_Plugin::registrar_cpts()` | idem | Registra `canchas` (público, archive `/canchas/`, supports title/editor/thumbnail) y `reservas` (interno, supports title) |
| `FU_Plugin::activar()` | idem | Callback de `register_activation_hook`. Registra CPTs + `flush_rewrite_rules()` para que `/canchas/` quede disponible al toque |
| `FU_VERSION`, `FU_PATH`, `FU_URL`, `FU_FILE` (constantes) | `wp-content/plugins/falta-uno/falta-uno.php` | Disponibles desde cualquier archivo del plugin |
| `fu_theme_setup()` | `wp-content/themes/falta-uno/functions.php` | Engancha `after_setup_theme`. Supports + menus + textdomain |
| `fu_theme_enqueue()` | idem | Engancha `wp_enqueue_scripts`. Bootstrap CDN + Bootstrap Icons + style.css con cache-bust por `filemtime` + Bootstrap JS bundle |
