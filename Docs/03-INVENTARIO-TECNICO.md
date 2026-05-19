# 03 — INVENTARIO TÉCNICO

*Mapa de archivos del plugin `falta-uno` y del theme `falta-uno`. Sirve para ubicar funciones, clases y assets sin tener que abrir el IDE. Se actualiza cada vez que se crea, mueve o renombra un archivo del código.*

*Última actualización: 2026-05-19. Estado: estructura objetivo definida desde el briefing — todos los archivos están en `[a crear]`.*

---

## Raíz WordPress local

```
C:\xampp\htdocs\falta-uno\
├── wp-config.php                       # ✅ Configurado: DB falta_uno, WP_HOME y WP_SITEURL = localhost/falta-uno
├── wp-content\
│   ├── plugins\
│   │   ├── falta-uno\                  # [a crear] — plugin custom (Día 2)
│   │   ├── advanced-custom-fields-pro\ # [a instalar] — ACF Pro, licencia de Manu
│   │   └── wp-mail-smtp\               # [a instalar] — SMTP para wp_mail()
│   ├── themes\
│   │   └── falta-uno\                  # [a crear] — theme custom (Día 5)
│   └── uploads\                        # auto, lo maneja WP
└── (core de WordPress — no se toca, no se versiona)
```

---

## Plugin `falta-uno` (estructura objetivo)

Ubicación: `wp-content/plugins/falta-uno/`

```
falta-uno/
├── falta-uno.php                       # [a crear] bootstrap del plugin: header WP + carga de clases + hooks de activación
├── README.md                           # [a crear] descripción mínima para el repo
├── uninstall.php                       # [a crear] cleanup al borrar plugin (dropea wp_falta_uno_slots, borra opciones fu_*)
├── includes\
│   ├── class-fu-plugin.php             # [a crear] clase principal singleton FU_Plugin
│   ├── class-fu-activator.php          # [a crear] crea tabla wp_falta_uno_slots, registra roles, flushea rewrite rules
│   ├── class-fu-deactivator.php        # [a crear] hooks de desactivación (no borrar datos)
│   ├── class-fu-cpt.php                # [a crear] registra CPTs `canchas` y `reservas`
│   ├── class-fu-roles.php              # [a crear] registra roles custom `jugador` y `dueno_cancha`
│   ├── class-fu-reservas.php           # [a crear] lógica de booking + gestión de slots
│   ├── class-fu-pagos.php              # [a crear] integración MercadoPago PHP SDK (Fase 2)
│   ├── class-fu-notificaciones.php     # [a crear] emails via wp_mail()
│   ├── class-fu-mapas.php              # [a crear] helpers Google Maps (geocoding, etc.)
│   └── class-fu-helpers.php            # [a crear] utilidades varias (formateo fecha/hora ARS, etc.)
├── admin\
│   ├── panel-dueno.php                 # [a crear] panel frontend del dueño de cancha
│   └── panel-admin.php                 # [a crear] panel global del admin (moderación, reportes)
├── ajax\
│   ├── ajax-slots.php                  # [a crear] devuelve horarios disponibles (JSON)
│   ├── ajax-reservar.php               # [a crear] crea reserva + inicia pago MP
│   └── ajax-webhook.php                # [a crear] recibe notificaciones IPN de MercadoPago
├── rest\
│   └── webhook-mp.php                  # [a crear] endpoint REST `/wp-json/falta-uno/v1/webhook` (alternativa al ajax)
└── assets\
    ├── css\
    │   └── falta-uno-admin.css         # [a crear] estilos mínimos del panel frontend del dueño
    └── js\
        ├── calendario.js               # [a crear] FullCalendar.js + llamadas AJAX
        ├── mapas.js                    # [a crear] Google Maps embed + interacción
        └── reserva.js                  # [a crear] flujo de reserva (selección slot → confirmación)
```

### Convenciones internas

- Toda clase en `includes/` sigue patrón `class-fu-{nombre}.php` (lowercase + hyphen) con clase `FU_{Nombre}` (PascalCase + underscore).
- Autoload simple: en `falta-uno.php` se hace un `require_once` explícito por cada clase. Sin Composer hasta que duela.
- Cada clase nueva se agrega también al inventario de arriba — esto no se hace solo.

---

## Theme `falta-uno` (estructura objetivo)

Ubicación: `wp-content/themes/falta-uno/`

```
falta-uno/
├── style.css                           # [a crear] header del tema (Theme Name, Version, etc.) + casi sin CSS — todo Bootstrap
├── functions.php                       # [a crear] enqueue Bootstrap + enqueue del JS/CSS del theme + setup básico
├── index.php                           # [a crear] fallback obligatorio (mínimo loop)
├── header.php                          # [a crear] doctype + nav con Bootstrap
├── footer.php                          # [a crear] footer + wp_footer()
├── sidebar.php                         # [a crear] sidebar opcional (probablemente no se use en MVP)
├── front-page.php                      # [a crear] home con buscador + mapa
├── archive-canchas.php                 # [a crear] listado de canchas con filtros
├── single-canchas.php                  # [a crear] ficha de cancha + calendario de disponibilidad
├── page-mi-panel.php                   # [a crear] panel del dueño de cancha (template para página estática)
├── page-reserva.php                    # [a crear] confirmación de reserva
├── 404.php                             # [a crear] página de error
├── searchform.php                      # [a crear] form de búsqueda reusable
└── assets\
    ├── css\
    │   └── custom.css                  # [a crear] mínimos overrides para FullCalendar y Google Maps (ver 02-GUIAS-TECNICAS §1)
    ├── js\
    │   └── main.js                     # [a crear] JS general del theme (no específico de calendario/mapa/reserva — eso está en el plugin)
    ├── img\
    │   └── (logos, placeholders)       # [a poblar]
    └── vendor\
        └── bootstrap\                  # [a poblar antes de prod] descarga local de Bootstrap para no depender de CDN
```

---

## Repos Git (cuando se inicialicen)

Pendientes — ver `01-ESTADO-ACTUAL.md` (decisión diferida sobre si un solo repo o dos).

Si va con dos repos (como dice el arranque original):

| Repo | Path local | Qué contiene | `.gitignore` clave |
|---|---|---|---|
| `falta-uno` | `C:\xampp\htdocs\falta-uno\wp-content\` (probablemente solo el subdir del plugin+theme) | Plugin + theme custom | `node_modules/`, `vendor/`, `*.log`, `@*`, `.DS_Store`, `.vscode/` |
| `falta-uno-docs` | `C:\Proyectos\Falta Uno\` | Carpeta `Docs/` + arranque histórico + briefing | `futbol/` (material de referencia, no canon) — *evaluar* |

---

## Tabla de búsqueda rápida (a poblar)

Cuando el código exista, esta sección se llena con "función X → archivo Y línea Z" para grep rápido. Por ahora vacía.

| Función / clase | Archivo | Línea |
|---|---|---|
| (vacío hasta que exista el plugin) | | |
