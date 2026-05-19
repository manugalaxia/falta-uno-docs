# Manual de trabajo con Claude en proyectos de código

*Documento maestro para llevar a cualquier proyecto donde trabajemos juntos con Claude (Cowork / claude.ai). Pegá la sección que corresponda en las custom instructions del proyecto, y subí este archivo entero como referencia al inicio.*

---

## Por qué este manual existe

En el proyecto Landing+ aprendimos una forma de trabajar entre Manu (operador, aprendiendo Git) y Claude (asistente con acceso a filesystem y bash) que reduce fricción y deja trazabilidad de todo. Este manual condensa esas reglas para poder replicarlas en otros proyectos sin re-aprenderlas.

Las reglas cubren cuatro frentes: **cómo Claude se comporta**, **cómo se versiona en Git**, **cómo se documenta** y **cómo se cierra cada sesión**.

---

## 1. Setup inicial del proyecto (one-time, primera vez)

### 1.1. Custom instructions del proyecto

En Claude.ai (web) o Cowork, cada proyecto admite *custom instructions* que se cargan automáticamente. Pegar esto:

```
Estás trabajando en <NOMBRE DEL PROYECTO>. Siempre leé el archivo 00-ARRANQUE.md antes de responder. Respetá las decisiones de arquitectura ya tomadas. Al finalizar sesiones con decisiones importantes, generá un resumen para actualizar los documentos.
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

---

## 2. Reglas de comportamiento de Claude

Estas reglas las podés copy-paste en las custom instructions del proyecto o en `00-ARRANQUE.md`:

### Edita archivos directo, NO pide copy/paste

Claude tiene acceso al filesystem (Read, Write, Edit, bash). Cuando hay que modificar un archivo del proyecto, **lo hace directamente** — no le pasa snippet al operador para que copie y pegue. Si Claude pide "copiá este archivo a tal ruta", está ignorando lo que ya se acordó. Excepción: cuando el archivo es **muy grande** (>2000 líneas) y el cambio es muy puntual, puede entregar solo el snippet con la línea exacta.

### Guía Git paso a paso, con comandos en bloque de código

El operador no es dev Git-fluent. Para cualquier flujo (commit, push, branch, merge, tag, etc.), Claude **pega el comando completo en bloque de código**, una línea por instrucción, con explicación breve de qué hace cada una. No vale decir "hacé un commit" a secas.

Si el operador omite un paso obvio (branch antes de tocar código, push después de commit, tag al cerrar feature), Claude lo avisa y sugiere el comando.

### Inicio de sesión con "vamos a trabajar en X"

La primera respuesta de Claude empieza con *"Antes de tocar nada, en Git hacé esto:"* y los comandos para preparar el repo (typically `git status` para confirmar punto de partida limpio). **Nunca codear directo sobre `main` sin checkear el estado primero.**

### Cierre de sesión proactivo

Cuando una sesión cierra con cambios commiteables, Claude sugiere activamente:
1. Commit + push del repo de código (+ tag si la sesión cerró un release).
2. Commit + push del repo de docs si se editó documentación.
3. Actualización del HISTORIAL con caso + decisión + regla derivada.

No espera a que el operador lo pida.

### Mantiene contexto entre sesiones

Editor del operador, terminal, sistema operativo, hosting de producción, rutas del proyecto: todo eso **queda en memoria persistente de Claude** después de la primera sesión. No se re-pregunta cada vez.

### Stack stock antes de cualquier override

Si una funcionalidad se puede resolver con lo que ya existe en el proyecto (frameworks, helpers, hooks, partials canónicos), usar eso primero. Crear archivos nuevos / módulos nuevos / overrides nuevos es la **última** opción, no la primera.

### Pregunta antes de avanzar cuando hay ambigüedad real

Mejor una pregunta corta que una hora de trabajo desviado. Especialmente cuando el operador dice "ya está en X" sin aclarar a qué capa se refiere.

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

Cada release (cualquier bump) se marca con un tag anotado. Convención de nombres:

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
git status

# 3) Stage + commit
git add .
git commit -m "<Componente> X.Y.Z: <descripción corta>"

# 4) Tag del release
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

**Decisión.** Qué se decidió hacer y por qué. Incluir alternativas descartadas y motivo.

**Regla derivada.** Una o más reglas reusables para el futuro. Empezar con "Cuando X, hacer Y" o "Antes de X, verificar Y".

**Estado al cierre.** Qué quedó hecho, qué queda pendiente.
```

### 4.3. Numeración

Antes de agregar una entrada, buscar la última en el HISTORIAL:

```bash
grep -oE "§[0-9]+\.[0-9]+" 00b-MAESTRO-HISTORIAL.md | sort -V | tail -1
```

Sumar 1 al número final.

---

## 5. Protocolo de cierre de sesión

Cada sesión que cierre con cambios importantes ejecuta este checklist:

1. **Verificar repo de código limpio.** `git status` desde `<carpeta-codigo>`. Si hay archivos sin commitear, decidir si entran al commit final o se descartan.

2. **Si hay cambios de código:** commit + tag + push (flujo de §3.4 o §3.5 según preferencia).

3. **Actualizar HISTORIAL.** Sumar entrada §X.Y nueva con caso + decisión + regla derivada. Bonus: actualizar ESTADO-ACTUAL si la sesión cerró algo que estaba pendiente.

4. **Commit + push del repo de docs.** Mensaje tipo `Actualiza HISTORIAL §X.Y: <tema de la sesión>`.

5. **(Opcional) Generar baseline tag** si la sesión cerró un hito significativo: `git tag -a baseline-YYYY-MM-DD -m "..."` + push.

---

## 6. Cómo arrancar una sesión nueva con Claude

### Lo que vos hacés

1. Abrir Cowork / Claude.ai.
2. Si el proyecto está conectado (las carpetas de código + docs están en el sidebar), saltar al paso 4.
3. Si no, conectar las carpetas del proyecto.
4. Empezar con la consulta concreta, ej: *"Vamos a trabajar en X."*

### Lo que Claude tiene que hacer

1. Leer `00-ARRANQUE.md` antes de responder a la primera consulta.
2. Leer las secciones específicas del MAESTRO / HISTORIAL según el tema (no leer todo).
3. Responder al pedido empezando con *"Antes de tocar nada, en Git hacé esto:"* + comandos de Git para preparar el repo.
4. Trabajar sobre el código (editando directo en disco) y guiar paso a paso en Git.
5. Al cierre, sugerir commits + docs sin esperar pedido.

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
- **Guiar Git paso a paso**: comando completo en bloque de código, una línea por instrucción, explicación breve.
- **Si el operador omite un paso obvio en Git**, avisar y sugerir.
- **Inicio de sesión con "trabajemos en X"** → primera respuesta empieza con "Antes de tocar nada, en Git hacé esto:" + comandos de preparación.
- **Cierre de sesión**: sugerir activamente commit + push + actualización de docs.

### Stack y decisiones tomadas

- **Respetar las decisiones de arquitectura del MAESTRO.** Si una parece equivocada, plantear antes de actuar.
- **Stack stock antes de overrides.** Usar lo que ya existe primero.
- **Versionado**: cada componente con `Version: X.Y.Z` en su header. Bump según semver. Tag por release con prefix por componente.

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
```

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

**Fin del manual.** Mantener actualizado: cuando aprendas una mejora del workflow en un proyecto, sumarla acá y los demás proyectos heredan.
