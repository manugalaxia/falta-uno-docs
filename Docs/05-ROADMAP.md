# 05 — ROADMAP

*Backlog ordenado de lo que viene después del MVP. Cada item tiene su fase y una nota corta. Esto no es un plan de ejecución día por día (eso es `01-ESTADO-ACTUAL`); es la vista panorámica de qué hay que hacer en algún momento.*

*Última actualización: 2026-05-19.*

---

## Fase 1 — MVP (13 al 22 de mayo)

Tracking detallado en `01-ESTADO-ACTUAL.md`. En una línea: tener el sitio funcionando end-to-end **sin pago real** (reserva libre, sin MercadoPago).

---

## Fase 2 — Pago + admin global (23 mayo al 12 junio)

| Item | Prioridad | Notas |
|---|---|---|
| Integración completa MercadoPago Checkout Pro + webhook IPN | 🔴 Crítico | Sin esto no hay negocio. Pasar de sandbox a producción al final |
| Panel admin global (moderación de canchas, reportes) | 🔴 Crítico | Manu lo va a usar a diario para aprobar canchas nuevas |
| Historial de reservas del jugador | 🟠 Alto | Mejora retención. Página `/mi-cuenta/reservas/` |
| Filtros avanzados en el buscador (precio, tipo, zona) | 🟠 Alto | El MVP solo filtra por zona; sumar tipo (F5/F7/F11) y rango de precio |
| Bloqueo de horarios por el dueño desde su panel | 🟠 Alto | Para feriados, mantenimiento, etc. |
| Recordatorios automáticos por email (WP Cron, 24hs antes de la reserva) | 🟡 Medio | Reduce no-shows |
| Optimización de performance (caching, lazy load imágenes) | 🟡 Medio | Hacerlo cuando haya tráfico real, no antes |

---

## Fase 3 — Crecimiento (post MVP estable)

Ideas todavía no comprometidas, ordenadas por probabilidad de implementación:

| Item | Estado idea |
|---|---|
| Sistema de calificaciones (jugador califica cancha post-reserva) | 🟢 Probable |
| Promociones / descuentos configurables por el dueño | 🟢 Probable |
| Notificaciones push web (Web Push API) | 🟡 A evaluar |
| Multi-deporte (paddle, tenis, etc.) — abstraer "cancha" como "espacio" | 🟡 A evaluar |
| App nativa mobile (iOS / Android) | 🔴 Improbable corto plazo |
| Marketplace de equipos (alquiler de pelotas, pecheras, etc.) | 🔴 Improbable corto plazo |
| Sistema de torneos / ligas amateurs | 🔴 Improbable corto plazo |
| Integración con WhatsApp Business para confirmaciones | 🟡 A evaluar |
| Programa de afiliados / referidos | 🟡 A evaluar |
| Cobro split: comisión automática para Falta Uno + neto al dueño | 🟢 Probable (depende de MP) |

---

## Deuda técnica anticipada

Cosas que se van a posponer en el MVP por tiempo y que hay que pagar en algún momento:

- **Bootstrap local en vez de CDN.** Antes del primer deploy a producción.
- **Tests automatizados.** Cero en el MVP. Considerar PHPUnit para lógica de booking y MercadoPago post-MVP.
- **Internacionalización (i18n).** El text domain `falta-uno` está reservado pero no se traduce nada. Si en algún momento Falta Uno cruza fronteras, hay que pasarse a `__()` / `_e()` real.
- **Diseño custom (post Bootstrap utilities only).** En algún momento Falta Uno necesita identidad propia más allá de "se ve a Bootstrap". Definir cuándo (probablemente cuando haya tracción y branding maduro).
- **Tabla `wp_falta_uno_slots` con índices optimizados.** Empieza con índice básico en `(cancha_id, fecha)`. Cuando haya volumen, revisar el plan de queries.

---

## Decisiones que el roadmap necesita en algún momento

- **Modelo de monetización exacto:** ¿comisión sobre cada reserva? ¿suscripción mensual al dueño? ¿freemium con destacados pagos? Hoy no está definido y bloquea Fase 3.
- **Política de cancelaciones y reembolsos.** Quién paga, con qué timing, qué pasa con la comisión. Mínimamente decidirlo antes de subir el flujo de pago real.
- **Términos y condiciones / política de privacidad.** Obligatorio antes de cobrar.
- **Expansión geográfica:** ¿se arranca con CABA o con todo el país desde el día 1?
