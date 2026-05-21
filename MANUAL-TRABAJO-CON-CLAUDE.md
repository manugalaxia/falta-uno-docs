# Manual de trabajo con Claude en proyectos de código

*Documento maestro para llevar a cualquier proyecto donde trabajemos juntos con Claude (Cowork / claude.ai). Pegá la sección que corresponda en las custom instructions del proyecto, y subí este archivo entero como referencia al inicio.*

*Última revisión: 2026-05-20. Cambios: ver sección 7 "Changelog del manual".*

---

## Por qué este manual existe

En el proyecto Landing+ aprendimos una forma de trabajar entre Manu (operador, aprendiendo Git) y Claude (asistente con acceso a filesystem y bash) que reduce fricción y deja trazabilidad de todo. Este manual condensa esas reglas para poder replicarlas en otros proyectos sin re-aprenderlas.

Las reglas cubren cinco frentes: **cómo Claude se comporta**, **cómo se versiona en Git**, **cómo se documenta**, **cómo se cierra cada sesión**, y **las trampas del entorno que ya conocemos y cómo navegarlas**.

---

## 1. Setup inicial del proyecto (one-time, primera vez)

### 1.1. Custom instructions del proyecto

En Claude.ai (web) o Cowork, cada proyecto admite *custom instructions* que se cargan automáticamente. Pegar esto:

```
Estás trabajando en <NOMBRE DEL PROYECTO>. Siempre leé el archivo 00-ARRANQUE.md antes de responder. Respetá las decisiones de arquitectura ya tomadas. Cuando tengas acceso de escritura al repo de Docs, actualizá los documentos vos directamente al cierre de sesiones con decisiones importantes — no pases resúmenes para que el operador pegue.
```

### 1.2. Estructura de carpetas en disco

Recomendado:

```
C:\Proyectos\<proyecto>\
├── Docs\                       ← documentación viva (repo Git aparte)
│   ├── 00-ARRANQUE.md          ← el primer archivo que Claude lee
│   ├── 00-MAESTRO.md           ← contexto general del proyecto
│   ├── 00b-MAESTRO-HISTORIAL.md ← bitácora de decisiones y trampas
│   ├── 01-ESTADO-ACTUAL.md     ← qué está hecho, qué falta
│   ├── 02-GUIAS-TECNICAS.md    ← procedimientos detallados
│   └── 05-ROADMAP.md           ← próximas mejoras acordadas
├── imports\                    ← archivos sueltos sin versionar
└── sql\                        ← scripts de base de datos
```

El **código** del proyecto vive donde corresponda al stack (en mi caso, `C:\xampp\htdocs\<proyecto>\` para proyectos WordPress). Lo importante es que el código y la doc sean **dos repos Git separados**.

### 1.3. Inicialización de los dos repos Git

```bash
# Repo de código
cd <ruta-del-codigo>
git init
# crear .gitignore que excluya WordPress core, vendor/, node_modules/, etc.
git add .
git commit -m "Initial commit: <descripción>"
git branch -M main
git remote add origin https://github.com/<usuario>/<proyecto>.git
git push -u origin main

# Repo de docs (carpeta separada)
cd C:\Proyectos\<proyecto>\Docs
git init
git add .
git commit -m "Initial commit: documentación del proyecto"
git branch -M main
git remote add origin https://github.com/<usuario>/<proyecto>-docs.git
git push -u origin main
```

### 1.4. Conectar las carpetas como workspaces de Cowork

**Esto es lo que destraba que Claude pueda editar archivos directamente.** Sin esto, Claude puede leer pero no escribir, y el flujo se reduce a "te paso un texto, copialo a mano".

En Cowork (la app desktop de Claude): menú lateral → conectar carpetas → seleccionar tanto la carpeta del código como la carpeta de los Docs. Las dos quedan listadas como "Folder: <ruta>" en el contexto de cada sesión.

Si una sesión arranca sin las carpetas conectadas, Claude tiene que avisarlo de entrada y pedir que se conecten antes de cualquier trabajo.

---

## 2. Reglas de comportamiento de Claude

Estas reglas las podés copy-paste en las custom instructions del proyecto o en `00-ARRANQUE.md`:

### Edita archivos directo, NO pide copy/paste

Claude tiene acceso al filesystem (Read, Write, Edit, bash). Cuando hay que modificar un archivo del proyecto, **lo hace directamente** — no le pasa snippet al operador para que copie y pegue. Si Claude pide "copiá este archivo a tal ruta", está ignorando lo que ya se acordó. Excepción: cuando el archivo es **muy grande** (>2000 líneas) y el cambio es muy puntual, puede entregar solo el snippet con la línea exacta.

**La misma regla aplica para Docs.** Si Claude tiene acceso de escritura al repo de Docs (carpeta conectada como workspace), las actualizaciones al cierre de sesión las edita directamente en los archivos — `00b-MAESTRO-HISTORIAL.md`, `01-ESTADO-ACTUAL.md`, lo que corresponda — y al operador solo le pasa los comandos de `git add` + commit + push para que él empuje a GitHub. **No le pasa un resumen estructurado para que copie y pegue a mano cuando puede escribir él mismo.** Esa interpretación literal de "generá un resumen" delega trabajo manual evitable. El resumen como texto en chat es la excepción (sesión sin acceso al repo), no la regla.

### Guía Git paso a paso, con comandos en bloque de código

El operador no es dev Git-fluent. Para cualquier flujo (commit, push, branch, merge, tag, etc.), Claude **pega el comando completo en bloque de código**, una línea por instrucción, con explicación breve de qué hace cada una. No vale decir "hacé un commit" a secas.

**Un comando por mensaje cuando se guía Git Bash.** Pegar el comando, esperar la salida del operador, después pasar al siguiente. **No listar 4-5 pasos consecutivos** — se acumula error y se vuelve imposible debuggear cuando algo no sale como se esperaba. Está OK avisar el plan al inicio ("vamos a hacer 5 pasos: status, add, commit, tag, push") pero ejecutar uno por uno.

Comandos chainables con `&&` (ej. `git add X && git status`) son OK como "un comando" porque son una sola unidad lógica con una sola salida.

Si el operador omite un paso obvio (branch antes de tocar código, push después de commit, tag al cerrar feature), Claude lo avisa y sugiere el comando.

### Inicio de sesión con "vamos a trabajar en X"

La primera respuesta de Claude empieza con *"Antes de tocar nada, en Git hacé esto:"* y los comandos para preparar el repo (typically `git status` para confirmar punto de partida limpio). **Nunca codear directo sobre `main` sin checkear el estado primero.**

### Commit baseline antes de tocar varios archivos seguidos

Si la sesión va a involucrar editar varios archivos (refactor, feature multi-componente, cambio cromático en CSS+templates, etc.), Claude **sugiere hacer un commit del estado actual antes de arrancar**. Es 30 segundos de trabajo del operador que crean un punto de retorno seguro: si algo se corrompe o se confunde, `git checkout -- <archivo>` o `git reset --hard HEAD` devuelven todo a un estado conocido bueno.

Especialmente importante cuando se van a tocar archivos con caracteres no-ASCII (ver sección 6, trampa del FS).

### Cierre de sesión proactivo

Cuando una sesión cierra con cambios commiteables, Claude sugiere activamente:
1. Commit + push del repo de código (+ tag si la sesión cerró un release).
2. **Edita los Docs directamente** (HISTORIAL, ESTADO-ACTUAL, MAESTRO si corresponde) y después pasa los comandos de `git add` + commit + push del repo de docs.
3. Si aplica, recuerda el deploy manual a producción (FTP, sftp, lo que corresponda al hosting).

No espera a que el operador lo pida.

### Mantiene contexto entre sesiones

Editor del operador, terminal, sistema operativo, hosting de producción, rutas del proyecto: todo eso **queda en memoria persistente de Claude** después de la primera sesión. No se re-pregunta cada vez.

### Stack stock antes de cualquier override

Si una funcionalidad se puede resolver con lo que ya existe en el proyecto (frameworks, helpers, hooks, partials canónicos), usar eso primero. Crear archivos nuevos / módulos nuevos / overrides nuevos es la **última** opción, no la primera.

### Pregunta antes de avanzar cuando hay ambigüedad real

Mejor una pregunta corta que una hora de trabajo desviado. Especialmente cuando el operador dice "ya está en X" sin aclarar a qué capa se refiere.

### Cuando entregás algo dos veces y "sigue mal" → PARÁ

Si después de dos intentos el operador dice "sigue mal" o "no entiendo qué hacés", **parar**. Releer el código canónico desde cero, dibujar el flujo, verificar el estado real (qué archivos están subidos, qué CSS aplica, qué dice DevTools). No entregar otra "fix" inmediata.

---

## 3. Convención de Git

### 3.1. Dos repos separados: código y docs

| Repo | Qué contiene | Por qué separado |
|---|---|---|
| `<proyecto>` (código) | Plugins, themes, scripts, configuración versionable | Cambia con cada feature, se puede deployar a producción |
| `<proyecto>-docs` (docs) | MAESTRO, HISTORIAL, ESTADO, GUIAS, ROADMAP | Cambia post-sesión, NO va al servidor de producción, puede tener visibilidad distinta |

Ambos en GitHub (privados o públicos según preferencia). Cada uno con su `main`, su `origin`, su flujo de commits independiente.

### 3.2. Versionado de componentes (semver)

Cada componente versionable (plugin, theme, paquete) declara su versión en un header estándar del stack:

```php
/**
 * Plugin Name: Mi Plugin
 * Version: 1.2.3
 */
```

Reglas de bump:

| Cambio | Bump | Ejemplo |
|---|---|---|
| Fix de bug, ajuste visual chico, refactor invisible | PATCH | `1.2.3 → 1.2.4` |
| Feature nuevo que NO rompe nada existente | MINOR | `1.2.3 → 1.3.0` |
| Cambio que rompe compatibilidad (schema, API, contrato) | MAJOR | `1.2.3 → 2.0.0` |

### 3.3. Tags de Git por componente

Cada release (cualquier bump) se marca con un **tag anotado** (`git tag -a`, no lightweight). Convención de nombres:

| Componente | Prefix | Ejemplo |
|---|---|---|
| Componente A | `a-v` | `a-v1.0.1` |
| Componente B | `b-v` | `b-v2.3.0` |
| Theme principal | `theme-v` | `theme-v1.0.0` |
| **Snapshot del bundle entero** | `baseline-YYYY-MM-DD` | `baseline-2026-05-19` |

El prefix permite filtrar después con `git tag --list "a-*"`. El sufijo `vX.Y.Z` matchea el `Version:` del header para mapear tag ↔ versión instalada sin abrir el código.

**Cuándo crear `baseline-`:** al iniciar un período de desarrollo grande o al cierre de una sesión densa, para tener ancla del estado funcional completo.

### 3.4. Flujo por feature — versión simplificada (recomendada para solo dev)

Sin branches. Trabajás directo en `main`. Cinco comandos por feature:

```bash
# 1) Editar código + bumpear Version: en el header del componente tocado

# 2) Verificar qué se va a commitear
git status --short

# 3) Stage + commit
git add .
git commit -m "<Componente> X.Y.Z: <descripción corta>"

# 4) Tag del release (anotado, con mismo mensaje del commit)
git tag -a <prefix>-vX.Y.Z -m "<Componente> X.Y.Z: <descripción>"

# 5) Push commit + tag en uno solo
git push origin main --follow-tags
```

El flag `--follow-tags` pushea `main` y cualquier tag anotado en el mismo comando — ahorra el `git push origin <tag>` separado.

### 3.5. Flujo por feature — versión con branches (recomendada cuando son varios devs)

Más ceremonia, más seguro cuando hay riesgo de pisar cambios de otros:

```bash
git checkout -b feat/descripcion
# editar + bumpear Version:
git add .
git commit -m "<Componente> X.Y.Z: <descripción>"
git checkout main
git merge feat/descripcion
git tag -a <prefix>-vX.Y.Z -m "..."
git push origin main --follow-tags
git branch -d feat/descripcion
```

Si un experimento sale mal: `git checkout main && git branch -D feat/descripcion` y nunca tocó `main`.

### 3.6. Volver a un release pasado

```bash
# Inspección no-destructiva (detached HEAD)
git checkout <prefix>-vX.Y.Z
git checkout main          # volver

# Volver main al estado del tag (destructivo)
git reset --hard <prefix>-vX.Y.Z
git push --force-with-lease origin main
```

### 3.7. Locks huérfanos: cómo destrabar

Síntoma: `fatal: Unable to create '.git/index.lock': File exists.`

Causa típica: una operación git se cortó a mitad y dejó el lock. También puede pasar si Claude corrió git desde su sandbox sobre el repo real.

Fix:

```bash
# 1) Verificar que NO haya otra herramienta git activa
# 2) Borrar el lock
rm -f .git/index.lock
# 3) Reintentar
```

### 3.8. Convención de mensaje de commit

Plantilla: `<Componente> X.Y.Z: <descripción corta>`.

Ejemplos:
- `Plugin Foo 1.0.1: fix de validación en formulario de contacto`
- `Theme Bar 1.1.0: nueva variante de hero con video de fondo`
- `chore: actualización de .gitignore`
- `docs: HISTORIAL §X.Y — caso del bug Z`

Si un commit toca varios componentes (raro), listarlos: `Plugin Foo 1.1.0 + Theme Bar 1.0.2: refactor del helper compartido`.

### 3.9. Path correcto cuando el repo está dentro de una subcarpeta

A veces el `.git` no vive en la raíz "obvia" del proyecto. Ejemplo Landing+: el repo está inicializado en `C:\xampp\htdocs\landing\wp-content\`, NO en `C:\xampp\htdocs\landing\`. Si corrés `git status` desde la raíz `landing/` falla con "not a git repository".

Cuando Claude guíe comandos git, **siempre incluir el `cd` absoluto al path correcto** al inicio del primer comando de la sesión:

```bash
cd /c/xampp/htdocs/landing/wp-content && git status --short
```

Es defensivo: Git Bash no preserva cwd entre ventanas y el operador puede haber abierto una nueva sin darse cuenta.

---

## 4. Convención de documentación

### 4.1. Archivos canónicos del repo de docs

| Archivo | Para qué sirve |
|---|---|
| `00-ARRANQUE.md` | Instrucciones de bootstrap para Claude al inicio de cada sesión |
| `00-MAESTRO.md` | Contexto general: qué es el proyecto, arquitectura, entidades, decisiones tomadas |
| `00b-MAESTRO-HISTORIAL.md` | Bitácora cronológica de decisiones, trampas encontradas, reglas derivadas |
| `01-ESTADO-ACTUAL.md` | Qué está implementado y validado, qué falta |
| `02-GUIAS-TECNICAS.md` | Procedimientos operativos detallados (deploy, configuración, helpers, etc.) |
| `03-INVENTARIO-TECNICO.md` | Mapa de archivos, funciones, hooks (opcional, si el proyecto es grande) |
| `05-ROADMAP.md` | Próximas mejoras acordadas, prioridades |

Para proyectos chicos, MAESTRO + HISTORIAL alcanza. Para proyectos grandes (Landing+ tipo), todos los archivos justifican.

### 4.2. Formato de entrada del HISTORIAL

Cada decisión importante de cada sesión genera una entrada numerada §X.Y (X = número de sección reservado para historial, Y = número incremental). Estructura sugerida:

```markdown
## §X.Y — <Título corto>

**Sesión:** YYYY-MM-DD.

**Caso.** Qué pasó. El bug, el pedido del operador, la situación que disparó la decisión.

**Decisión / Hipótesis de mecanismo.** Qué se decidió hacer y por qué. Incluir alternativas descartadas y motivo. Para trampas, incluir la hipótesis de por qué pasa.

**Regla derivada.** Una o más reglas reusables para el futuro. Empezar con "Cuando X, hacer Y" o "Antes de X, verificar Y".

**Estado al cierre.** Qué quedó hecho, qué queda pendiente.
```

### 4.3. Numeración

Antes de agregar una entrada, buscar la última en el HISTORIAL. Hay dos métodos:

**Método A — chequear el final del índice del HISTORIAL** (más confiable si el HISTORIAL está particionado en archivos temáticos como en Landing+):

```bash
tail -20 00b-MAESTRO-HISTORIAL.md
```

Buscar la línea tipo "El último §X.Y usado fue **§X.N**. La próxima entrada sería §X.N+1.".

**Método B — grep mecánico**:

```bash
grep -oE "§[0-9]+\.[0-9]+" 00b-MAESTRO-HISTORIAL.md | sort -V | tail -1
```

Sumar 1 al número final.

---

## 5. Protocolo de cierre de sesión

Cada sesión que cierre con cambios importantes ejecuta este checklist:

1. **Verificar repo de código limpio.** `git status --short` desde `<carpeta-codigo>`. Si hay archivos sin commitear, decidir si entran al commit final o se descartan.

2. **Si hay cambios de código:** commit + tag + push (flujo de §3.4 o §3.5 según preferencia).

3. **Actualizar HISTORIAL — editando directamente los archivos.** Sumar entrada §X.Y nueva con caso + decisión + regla derivada. Bonus: actualizar ESTADO-ACTUAL si la sesión cerró algo que estaba pendiente. Si el MAESTRO tiene secciones afectadas (forms cubiertos, decisiones, módulos), actualizar también esas secciones.

4. **Commit + push del repo de docs.** Mensaje tipo `docs: HISTORIAL §X.Y + <secciones del MAESTRO modificadas> — <tema de la sesión>`.

5. **(Si aplica) Deploy a producción.** Recordar el método del proyecto: FTP manual, sftp, rsync, git push automático, etc. Si requiere FTP manual, pasarle la lista exacta de archivos modificados a subir.

6. **(Opcional) Smoke test en producción del feature crítico** si la sesión tocó algo expuesto al público (forms, pagos, autenticación). Reportar qué se debería ver si funciona y qué se ve en `error.log` si falla.

7. **(Opcional) Generar baseline tag** si la sesión cerró un hito significativo: `git tag -a baseline-YYYY-MM-DD -m "..."` + push.

---

## 6. Trampas del entorno de trabajo conocidas

Sección nueva (2026-05-20). Las trampas acá son **del entorno** (filesystem de Cowork, interacción Git + autocrlf en Windows, editores que truncan al guardar), no del proyecto en sí. Conocerlas antes de tropezarlas ahorra horas.

### 6.1. El FS de Cowork puede truncar archivos al editarlos (heredado de §22.167 de Landing+)

**Síntoma.** Después de editar un archivo, el contenido en disco aparece más corto que lo esperado. Típicamente termina en mitad de palabra o expresión (`text-decoration: no`, `<main id="pri`, `'anch`). El parser de PHP/JS no se queja porque el truncado es físico, no semántico. Worst case: una función crítica desaparece y un módulo deja de funcionar sin error visible.

**Mecanismo.** El filesystem montado en Cowork (FUSE) tiene un cache virtual que puede diverger del disco real visto por el shell del operador y por el stack (XAMPP, Node, etc.). Las escrituras que NO persisten correctamente al disco real:

- `Edit` y `Write` del File API de Claude sobre archivos **existentes** con caracteres no-ASCII (tildes, eñes, em-dash) o tamaño mediano-grande.
- `cat >> $archivo` (append) en bash — el archivo "crece" en `wc -l` virtualmente pero las líneas appendeadas no están en disco real.
- `python open(F, 'w').write(...)` SIN `os.fsync` explícito.
- Algunos editores del operador (TopStyle entre otros) pueden truncar al guardar archivos con caracteres no-ASCII grandes.

**Métodos seguros confirmados (todos persisten en disco real):**

- **`cat > $F <<'EOF' ... EOF`** (heredoc bash) — escritura completa, atómica.
- **`cp $SRC $DST`** — copia entre archivos.
- **`sed -i 's/old/new/' $F`** — reemplazo in-place atómico (perfecto para cambios chicos tipo bump de versión).
- **`python3` con `os.open(F, os.O_WRONLY|os.O_TRUNC|os.O_CREAT)` + `os.write(fd, src)` + `os.fsync(fd)` + `os.close(fd)`** — write completo a nivel kernel con flush forzado.
- **`git show HEAD:<path> > <path>`** — útil para restaurar al estado del último commit cuando el archivo se corrompió.

**Regla operativa.** Para cualquier edit a archivo existente con caracteres no-ASCII o tamaño mediano:

1. **No usar `Edit` ni `Write` del File API.** Usar bash + heredoc, `sed -i`, o `python` con `os.fsync`.
2. **Validar SIEMPRE post-write:**
   - `wc -c <archivo>` — los bytes deben coincidir con lo esperado.
   - `tail -c 300 <archivo>` — el final del archivo debe ser lo esperado (no truncado, no mid-property).
   - `file <archivo>` — debe decir el tipo correcto ("PHP script, Unicode text, UTF-8 text" para PHP/MD con UTF-8). Si dice "data" o menciona "binary characters", hay null bytes o bytes raros.
   - `tr -d -c '\000' < <archivo> | wc -c` — debe dar `0`. Si da un número > 0, hay null bytes que probablemente rompen el parser. **No confiar en `grep -q $'\0'` para esto — falso positivo conocido en bash** (ver §22.179 si el proyecto lo importa).

### 6.2. `git checkout -- <file>` puede no restaurar a disco con `autocrlf=true` y EOLs mixtos (§22.177)

**Síntoma.** Después de `git checkout -- <archivo>` desde Git Bash en Windows, el comando devuelve salida vacía (éxito aparente) pero el archivo en disco sigue corrupto/modificado. `tail` y `wc -c` muestran que NO se restauró.

**Mecanismo.** Si el blob HEAD tiene una mezcla de line endings (parte CR-only legacy + parte LF agregada en sesiones nuevas), `core.autocrlf=true` (default de Git Bash en Windows) parece fallar silenciosamente al normalizar — no escribe el blob al disco si considera que el archivo en disco "ya matchea" la normalización.

**Workaround.** Bypaseado la conversión de autocrlf escribiendo los bytes exactos del blob:

```bash
git show HEAD:<path> > <path>
```

Desde Git Bash funciona si lanzás con autocrlf desactivado en ese comando puntual:

```bash
git -c core.autocrlf=false show HEAD:<path> > <path>
```

O desde el sandbox de Cowork (que no aplica autocrlf por default) si Claude lo hace por vos.

**Regla operativa.** Después de cualquier `git checkout` para restaurar un archivo dañado, **validar con `wc -c` y `tail`** que el contenido coincide con el HEAD esperado. Comparar contra `git cat-file -s HEAD:<path>` (que da el tamaño del blob). Si difieren, usar el workaround.

### 6.3. `git status` puede mostrar archivos modificados que para el operador no existen (§22.178)

**Síntoma.** Claude desde su sandbox Linux reporta 60+ archivos modificados en `git status`. El operador desde Git Bash en Windows ve solo 3 (los reales) — los otros 57 no existen en su lado.

**Mecanismo.** El sandbox Linux NO aplica `core.autocrlf`. Cuando lee el working tree, ve los bytes literales del disco (que ya están en CRLF o LF según se guardaron). Cuando compara contra el index, ve diferencias de EOL que en el lado del operador (autocrlf=true) git normaliza al vuelo y reporta como "sin cambios". Misma copia de trabajo, dos visiones radicalmente distintas según el `core.autocrlf` del shell.

**Regla operativa.** El "qué está realmente modificado" es **siempre el lado del operador** (Windows + autocrlf=true). Cuando Claude opere desde el sandbox y vea cambios masivos no esperados, hipótesis 1 a chequear: discrepancia de autocrlf. Confirmar **antes de actuar** pidiéndole al operador un `git status --short` desde Git Bash. Si en su lado los archivos aparecen limpios, el "ruido" es de visibilidad y no hay que tocar nada. Si en su lado también aparecen, recién ahí hay un problema real.

### 6.4. El editor del operador puede truncar archivos al guardar

**Síntoma.** El operador modifica un archivo en su editor (TopStyle, Notepad++, otros), guarda, y avisa "listo". Al revisar el diff, el archivo está truncado al final — perdió un bloque grande de contenido. El editor no avisó nada, guardó "exitosamente".

**Causa probable.** Algunos editores manejan mal archivos con line endings mixtos o caracteres no-ASCII grandes y truncan silenciosamente al guardar.

**Hábito recomendado para el operador.** Después de guardar en el editor, antes de avisarle a Claude:

1. Mirá rápido el final del archivo. Si el último bloque no está como esperabas, recargá desde disco (no desde el editor) o restaurá con `git checkout -- <archivo>`.
2. Si vas a tocar varios archivos seguidos, hacé un commit baseline primero (sección 2 de este manual). Así si algo se trunca, tenés a dónde volver.

**Hábito recomendado para Claude.** Cuando el operador avisa "modifiqué X archivo", antes de procesar:

1. Pedirle `wc -c <archivo>` o `tail -c 300 <archivo>` desde Git Bash para confirmar tamaño/final.
2. Si lo que aparece está cortado, parar y restaurar antes de seguir.

---

## 7. Cómo arrancar una sesión nueva con Claude

### Lo que vos hacés

1. Abrir Cowork / Claude.ai.
2. Si el proyecto está conectado (las carpetas de código + docs están en el sidebar), saltar al paso 4.
3. Si no, conectar las carpetas del proyecto.
4. Empezar con la consulta concreta, ej: *"Vamos a trabajar en X."*

### Lo que Claude tiene que hacer

1. Leer `00-ARRANQUE.md` antes de responder a la primera consulta.
2. Leer las secciones específicas del MAESTRO / HISTORIAL según el tema (no leer todo).
3. Responder al pedido empezando con *"Antes de tocar nada, en Git hacé esto:"* + comandos de Git para preparar el repo.
4. Trabajar sobre el código (editando directo en disco) y guiar paso a paso en Git con un comando por mensaje.
5. Al cierre, editar Docs directamente + pasar comandos de commit + push sin esperar pedido.

---

## 8. Changelog del manual

**2026-05-20 (rev 2):**

- Sección 1 (Setup): sumada §1.4 "Conectar las carpetas como workspaces de Cowork" — crítico, sin esto Claude no puede editar.
- Sección 1.1 (Custom instructions): ajustado el texto base — explícito sobre "actualizá los documentos vos directamente, no pases resúmenes".
- Sección 2 (Edita archivos directo): segundo párrafo nuevo aplicando la regla también a Docs.
- Sección 2 (Guía Git): sumada regla "un comando por mensaje" y excepción de comandos chained con `&&`.
- Sección 2: bloque nuevo "Commit baseline antes de tocar varios archivos seguidos".
- Sección 2: bloque nuevo "Cuando entregás algo dos veces y 'sigue mal' → PARÁ".
- Sección 3.3: aclarado que los tags son anotados (`git tag -a`).
- Sección 3.4: paso 2 ahora usa `git status --short` (más compacto).
- Sección 3 nueva §3.9: path correcto cuando el `.git` vive en una subcarpeta (caso Landing+).
- Sección 4.3: dos métodos para encontrar el próximo §X.Y libre (método A más confiable con HISTORIAL particionado).
- Sección 5 (Cierre de sesión): paso 3 reescrito — "editando directamente los archivos". Paso 5 nuevo (deploy manual). Paso 6 nuevo (smoke test en producción).
- **Sección 6 NUEVA: "Trampas del entorno de trabajo conocidas"** — cubre FS de Cowork (§22.167), `git checkout` + autocrlf (§22.177), discrepancia `git status` sandbox vs operador (§22.178), editor que trunca al guardar.

**2026-05-19 (rev 1):**

- Versión inicial.

---

## Anexo A — Plantilla mínima de `00-ARRANQUE.md`

Pegar este contenido al archivo `Docs/00-ARRANQUE.md` del proyecto nuevo, ajustando los placeholders:

```markdown
# INSTRUCCIONES DE ARRANQUE — <PROYECTO>

*Este archivo se sube al inicio de cada sesión. Contiene las instrucciones para que Claude se autoconfigure y trabaje bien con el proyecto.*

## Para Claude: cómo arrancar esta sesión

### Paso 1 — Leer el contexto mínimo

Leer del proyecto, en este orden:

1. `00-MAESTRO.md` → secciones §1 (qué es el proyecto) y §2 (arquitectura).
2. Nada más por ahora — el resto se lee on-demand según el tema.

### Paso 2 — Leer secciones específicas según el tema

Cuando el operador haga la primera consulta concreta, identificar el tema y leer solo lo necesario del MAESTRO. Si el tema toca trampas conocidas, ir al HISTORIAL.

### Paso 3 — Otros archivos del proyecto, on-demand

- `00b-MAESTRO-HISTORIAL.md` → solo si una situación recuerda a una trampa de §X.
- `01-ESTADO-ACTUAL.md` → si la consulta es "¿qué falta hacer en X?".
- `02-GUIAS-TECNICAS.md` → si la consulta requiere procedimientos operativos.
- `05-ROADMAP.md` → si la consulta es sobre planificación.

## Reglas de comportamiento durante la sesión

### Forma de trabajar con el operador

- **Editar archivos directo** en disco. No pedir copy/paste.
- **Guiar Git paso a paso**: comando completo en bloque de código, **un comando por mensaje**, esperar la salida del operador antes del siguiente.
- **Si el operador omite un paso obvio en Git**, avisar y sugerir.
- **Inicio de sesión con "trabajemos en X"** → primera respuesta empieza con "Antes de tocar nada, en Git hacé esto:" + comandos de preparación.
- **Cierre de sesión**: editar los Docs directamente (HISTORIAL, ESTADO-ACTUAL, secciones del MAESTRO afectadas) y pasar comandos de commit + push. No pasar resúmenes para que el operador pegue a mano.

### Stack y decisiones tomadas

- **Respetar las decisiones de arquitectura del MAESTRO.** Si una parece equivocada, plantear antes de actuar.
- **Stack stock antes de overrides.** Usar lo que ya existe primero.
- **Versionado**: cada componente con `Version: X.Y.Z` en su header. Bump según semver. Tag anotado por release con prefix por componente.

### Trampas del entorno conocidas

- **Cuando edites archivos con caracteres no-ASCII desde el sandbox**, no usar Edit/Write del File API. Usar bash con heredoc, `sed -i`, o `python` con `os.fsync`. Validar siempre post-write con `wc -c`, `tail`, `file`, y `tr -d -c '\000' | wc -c` para null bytes.
- **`git checkout -- <file>` con `autocrlf=true` puede no restaurar a disco** si el blob tiene EOLs mixtos. Si pasa, validar con `tail`/`wc -c` que el archivo realmente cambió.
- **`git status` desde el sandbox puede mostrar archivos modificados que para el operador no existen** (autocrlf normaliza). La verdad es el lado del operador.

### Cuándo parar y preguntar

- Ante ambigüedad real, preguntar antes de avanzar.
- Si el operador dice "ya está en X", confirmar a qué capa se refiere.
- Si entregás dos veces y "sigue mal" → parar, releer el código, no entregar otra fix inmediata.

## Identidad del proyecto (resumen)

- **Proyecto:** <NOMBRE>
- **Operador:** <NOMBRE OPERADOR>
- **Stack:** <PHP/Node/Python/etc>
- **Hosting:** <DONDE SE DEPLOYA>
- **Editor:** <EDITOR>
- **Terminal:** <TERMINAL>

**Fin del archivo de arranque.** Próxima acción: leer `00-MAESTRO.md` §1 y §2, y esperar la primera consulta del operador.
```

---

## Anexo B — Plantilla mínima de `00-MAESTRO.md`

```markdown
# <PROYECTO>
## Documento Maestro de Contexto

**Última actualización: YYYY-MM-DD.** <Una línea con el último cambio mayor.>

## ÍNDICE NAVEGABLE

| Tema | Secciones a leer |
|---|---|
| Contexto general | §1 |
| Arquitectura | §2 |
| <Tema específico A> | §3 |
| <Tema específico B> | §4 |
| Aprendizajes / trampas | §22 (en HISTORIAL) |

## §1. QUÉ ES ESTE PROYECTO

<Descripción de 2-3 párrafos: qué es, para quién, qué problema resuelve.>

## §2. ARQUITECTURA DEL SISTEMA

<Stack, capas, componentes principales, dependencias.>

## §3. <Sección específica>

...
```

---

## Anexo C — Plantilla de primera entrada del HISTORIAL

```markdown
# HISTORIAL — <PROYECTO>

*Bitácora de decisiones, trampas y reglas derivadas. Una entrada §22.X por situación. Nunca borrar entradas — corregir con entradas posteriores que se refieran a la previa.*

---

## §22.1 — <Título de la primera decisión>

**Sesión:** YYYY-MM-DD.

**Caso.** <Qué pasó.>

**Decisión.** <Qué se decidió y por qué.>

**Regla derivada.** <Una o más reglas para el futuro.>

**Estado al cierre.** <Qué quedó hecho, qué queda pendiente.>

---

## Recordatorio para sesiones nuevas

- El último §22.X usado fue **§22.1**. La próxima entrada sería §22.2.
```

(El recordatorio del final es importante mantenerlo actualizado: es la forma confiable de que la próxima sesión encuentre el número correcto.)

---

## Anexo D — `.gitignore` mínimo recomendado

Para el repo de código (ajustar según stack):

```gitignore
# Dependencias
node_modules/
vendor/
.venv/
__pycache__/

# Builds
dist/
build/
*.pyc

# Editor / OS
.vscode/
.idea/
*.swp
*.swo
Thumbs.db
.DS_Store

# Logs
*.log
logs/

# Carpetas autogestionadas por el stack
uploads/
cache/
backup/
backups/

# Convenciones del operador
@*
**/@*
estructura.bat
estructura.txt
test_*.txt
```

Para el repo de docs:

```gitignore
Thumbs.db
.DS_Store
.vscode/
*.swp
*.swo
```

---

**Fin del manual.** Mantener actualizado: cuando aprendas una mejora del workflow en un proyecto, sumala al changelog (sección 7) y los demás proyectos heredan.
