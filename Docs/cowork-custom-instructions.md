# Custom instructions del proyecto Cowork — Falta Uno

*Texto exacto para pegar en la UI de Cowork (Settings del proyecto → Project instructions). Reemplaza a la versión vieja registrada como "pendiente de Manu" en `00b-MAESTRO-HISTORIAL.md §22.5`.*

*Última revisión: 2026-05-21.*

---

## Qué cambia

### Versión vieja (la que está hoy cargada en Cowork)

```
Estás trabajando en Falta Uno. Siempre leé el archivo 00-MAESTRO antes de responder. Respetá las decisiones de arquitectura ya tomadas. Usá el stack stock antes de cualquier override. Al finalizar sesiones con decisiones importantes, generá un resumen para actualizar los documentos.
```

### Versión nueva (ESTE es el bloque a pegar)

```
Estás trabajando en Falta Uno. Siempre leé el archivo 00-ARRANQUE.md antes de responder. Respetá las decisiones de arquitectura ya tomadas. Cuando tengas acceso de escritura al repo de Docs, actualizá los documentos vos directamente al cierre de sesiones con decisiones importantes — no pases resúmenes para que el operador pegue.
```

---

## Por qué cambia cada parte

1. **`00-MAESTRO` → `00-ARRANQUE`.** El ARRANQUE es el bootstrap operativo: reglas de comportamiento + trampas del entorno + orden de lectura. Tiene puntero al MAESTRO, así que cargar ARRANQUE primero implica cargar MAESTRO después igual. Al revés no: cargar MAESTRO primero deja afuera las reglas operativas del ARRANQUE.

2. **Se borra "Usá el stack stock antes de cualquier override".** Esa regla ya vive en `00-ARRANQUE.md` y en `MANUAL §2`. Las custom instructions del proyecto Cowork se mantienen mínimas (todo lo demás lo cargan los docs).

3. **"Generá un resumen" → "actualizá los documentos vos directamente".** Cambio clave alineado al `MANUAL-TRABAJO-CON-CLAUDE.md` rev 2. Antes Claude tiraba resumen estructurado al final de cada sesión para que Manu copiara y pegara a mano. Ahora Claude edita los archivos directamente (con bash + heredoc + validación post-write según `MANUAL §6.1`) y solo pasa los comandos `git add` + commit + push.

---

## Pasos para aplicar

1. Abrí Cowork (desktop o web), entrá al proyecto **Falta Uno**.
2. Settings del proyecto → **Project instructions** (o la sección equivalente en la UI vigente — donde aparece el texto viejo de arriba).
3. Seleccioná y borrá el contenido entero del campo.
4. Pegá la **versión nueva** del bloque de arriba (los 4 renglones dentro del code fence).
5. Guardá / aplicá los cambios desde la UI.

### Importante

- **La próxima sesión** que arranques en Cowork va a cargar el contrato nuevo automáticamente.
- **Esta sesión actual** sigue con las custom instructions viejas en memoria — los cambios no se aplican retroactivamente. Eso está OK: ya estoy operando bajo el contrato del MANUAL rev 2 desde que lo adoptamos en `§22.5`. El cambio en la UI es para que las **futuras** sesiones arranquen alineadas sin tener que re-explicar el contrato cada vez.

---

## Verificación post-aplicación

Cuando arranques la próxima sesión, el primer mensaje de Claude debería:

1. Mencionar la lectura de `00-ARRANQUE.md` (no de `00-MAESTRO.md` directamente).
2. Si la sesión cierra con decisiones, **editar los docs (HISTORIAL/ESTADO-ACTUAL) directamente** y pasarte solo los comandos Git, en lugar de tirarte un resumen estructurado para pegar a mano.

Si en la próxima sesión Claude empieza leyendo el MAESTRO directo o te ofrece un "resumen para que pegues", las custom instructions no se guardaron bien o quedó cache vieja — repetir los pasos 1-5.

---

*Este archivo es un helper one-shot: una vez que Manu actualiza las custom instructions en Cowork, queda como referencia histórica de cómo era el contrato anterior y cómo se migró. No volver a tocarlo salvo que el MANUAL cambie de nuevo el bloque base.*
