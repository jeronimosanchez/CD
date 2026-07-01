# _gobierno_kb — Cómo se crean y gobiernan los KBs

> Manual unificado de gobierno del sistema de KBs: nomenclatura, estructura de entradas y proceso de gestión.
> Última actualización: 2026-06-18

---

## 1. Nomenclatura — 4 capas y origen

### Las 4 capas (prefijo)

| Prefijo | Capa | Qué es | ¿IP? |
|---|---|---|---|
| `kb_ag_` | **Agnóstico (dominio)** | CD que vale para cualquier cliente y plataforma | ✅ IP vendible |
| `kb_sys_` | **Sistema (meta)** | Cómo funciona tu máquina por dentro | ✅ IP profunda |
| `kb_plat_` | **Plataforma (adapter)** | Específico de una plataforma | 🟡 Reemplazable |
| `kb_proj_` | **Proyecto (instancia)** | Datos de un cliente concreto | ⚪ Del cliente |

```
¿Vale para cualquier cliente?           → kb_ag_
¿Describe tu sistema de optimización?   → kb_sys_
¿Es de una plataforma concreta?         → kb_plat_
¿Es de un cliente concreto?             → kb_proj_
```

### Origen: Authored vs Grown (siempre explícito, nunca por defecto)

```
Authored  →  el contenido lo escribes tú directamente, o lo trae RES como candidato.
             En ambos casos, tú decides qué entra.
Grown     →  el contenido lo propone el sistema corriendo (patrones de QAP, destilación).
             El sistema PROPONE entradas; tú las revisas y apruebas antes de que entren.
             Arranca vacío. NO tiene item de construcción propio.
```

**Regla de oro de la destilación (para que los Grown agnósticos no se contaminen):**
```
DEFAULT     →  todo patrón aprendido va a la capa ESPECÍFICA (kb_proj_/kb_plat_)
PROMOCIÓN   →  sube a kb_ag_ SOLO cuando re-aparece en ≥2 clientes/plataformas
```

### La lista de KBs vive en `_index.md` (no se duplica aquí)

`_index.md` es la **fuente única** de qué KBs existen (capa, estado, descripción).
Aquí, en el gobierno, se define solo la **convención** (las 4 capas + Authored/Grown); la lista
concreta NO se replica — así no puede divergir. `kb_doctor` valida el disco contra `_index.md`.

> Pendiente: si se quiere el `origen` (Authored/Grown) por-KB, va como columna en `_index.md`.

### Renombrado ejecutado (06-jun-2026)

```
kb_global → kb_ag_global · kb_skills → kb_sys_core · kb_cx → kb_plat_cx · kb_petal → kb_proj_petal
```

### Expertos (resuelto)

El experto es una **skill** (no un KB). Va contra el KB de proyecto del componente
(`kb_proj_petal_playbook`, etc.) y lee `kb_plat_cx` + `kb_ag_global` como referencia.
No hay `kb_ag_behavior` ni KB propio del experto — su conocimiento son los KBs de proyecto y plataforma.

---

## 2. Anatomía de una entrada — cómo se escribe cada entrada

> Norma editorial para CUALQUIER KB. La skill que escribe o mantiene un KB lee esto primero.

### Las tres zonas

| Zona | Para quién | Qué lleva |
|---|---|---|
| **Metadatos** (`clave: valor`) | máquina + LLM | proveniencia, calidad, vigencia, versión |
| **Cuerpo** (prosa) | LLM / humano | QUÉ, POR QUÉ, ejemplos |
| **Bloque-máquina** (` ```static `) | scripts | datos estructurados (umbrales) |

> Sin YAML frontmatter `---`: las entradas son secciones dentro de un archivo, no archivos sueltos.
> Los campos `clave: valor` bajo el título ya son parseables por máquina.

### Plantilla

````
### <CÓDIGO> — <título que enuncia el principio entero>
fuente: <URL o "hallazgo propio">
confianza: Alta | Media | Baja
version_status: current | deprecated | superseded
platform_version / agent_version: <versión a la que aplica>   ← solo plat / proj
last_verified: <YYYY-MM-DD>
instancia de: <A-code>  o  [<A-code>, <A-code>]               ← opcional, acepta lista

QUÉ: qué es y dónde aplica.
POR QUÉ: qué fallo evita.
EJEMPLO ✅: caso correcto.        ← concretas; en agnósticas → "no aplica — <por qué>"
EJEMPLO ❌: caso incorrecto.      ← opcional

```static                         ← opcional, solo si es auditable por script
check: <nombre>
<clave>: <valor>
```
````

### Campos

| Campo | ¿Obligatorio? | Qué hace |
|---|---|---|
| título | ✅ | Enuncia el principio entero |
| `fuente` | ✅ | De dónde sale (proveniencia) |
| `confianza` | ✅ | Calidad: Alta/Media/Baja |
| `version_status` | ✅ | Vigencia: current/deprecated/superseded |
| `last_verified` | ✅ | Cuándo se comprobó (YYYY-MM-DD) |
| `QUÉ` / `POR QUÉ` | ✅ | Qué es / qué fallo evita |
| `platform_version` / `agent_version` | ✅ en su capa | A qué versión aplica |
| `instancia de` | ⚪ | Linaje al/los principio(s) agnóstico(s) — acepta lista |
| `EJEMPLO ✅/❌` | ⚪ | Ilustración (en agnósticas → "no aplica") |
| bloque `static` | ⚪ | Dato auditable por script |

### Clasificación de verificabilidad

Cada entrada debe saber cómo se verifica su principio:

| Tipo | Cuándo | Cómo se verifica |
|---|---|---|
| **static** | Umbral, keyword o estructura auditable sin ejecutar el agente | Bloque ` ```static ` → `static_audit.py` lo lee automáticamente |
| **dynamic** | Comportamiento del LLM, solo verificable ejecutando una conversación real | TC en la suite QA (`test_qa_playbooks.py`) |
| *(ninguna)* | Principio de diseño puro, sin verificación automatizable | Solo revisión humana |

Ejemplos:
- CX-25 (tamaño de playbook) → **static** — umbral numérico medible sin ejecutar nada
- CX-37 (token budget acumulativo) → **dynamic** — el impacto solo se detecta corriendo conversaciones reales; el historial crece y los examples que CX incluye varían por presión de tokens
- A10 (ejemplos valen más que instrucciones) → **ninguna** — principio agnóstico, revisión humana

### Reglas

- **R1 — Una entrada = una idea.** Si dos dicen lo mismo, se fusionan (duplicar dobla tokens sin valor).
- **R2 — Atomicidad y tamaño.** Un principio por entrada; si necesitas explicar dos, son dos entradas. ~300-500 palabras como *alarma* (AWS RAG), no ley — pasarse suele indicar dos principios mezclados. El límite real es el tamaño TOTAL del KB (la skill lo carga entero): pendiente de calibrar con datos propios, sin número inventado.
- **R3 — Ejemplos según la naturaleza del contenido.** Concretos → `EJEMPLO ✅/❌`. Agnósticos/abstractos → `EJEMPLO: no aplica — <por qué>` + puntero a una instancia. Se decide por concreto/abstracto, NO por capa (eso limitaría capas futuras).
- **R4 — El título ya es información.** Enuncia el principio completo; si es claro, el cuerpo puede ser más corto.
- **R5 — Caducidad según fase.** Experimental (sin caducidad) o estable (no referenciada en 90 días → revisar → archivar). Los principios agnósticos son durables: no caducan.
- **R6 — Anatomía.** Orden fijo: título → metadatos → QUÉ → POR QUÉ → ejemplos → bloque. Si una entrada no tiene "por qué", probablemente es un detalle, no un principio.
- **R7 — El agente es siempre el sujeto.** Se escribe como si el agente la leyera y aplicara directamente. Si no puede aplicarla, no pertenece al KB.
- **R8 — Nombra los componentes con precisión.** Cada parámetro, columna, intent o variable con un nombre exacto. Lo ambiguo genera bugs silenciosos.
- **R9 — Valida ejecutando los principios varias veces.** Los LLM no dan siempre el mismo resultado; una ejecución no basta.
- **R10 — Lo mantiene quien construye**, antes de cada ciclo. Un KB desactualizado hace trabajar al agente contra el objetivo equivocado.
- **R11 — Demuestra la mejora** comparando resultados antes y después. Con datos, no por suposición.
- **R12 — Separa el principio agnóstico del dato config-específico.** El principio en entradas numeradas; los números de un modelo/hardware concreto en sección "Perfil" marcada como candidata a SPLIT.
- **R13 — Metadatos obligatorios de trazabilidad:** `fuente` · `confianza` · `version_status` · `last_verified` (+ versión de capa). Confianza: **Alta** = fuente oficial/hard limit · **Media** = anti-patrón externo o umbral propuesto · **Baja** = principio sólido sin benchmark.
- **R14 — Bloque `static`** si la entrada tiene un umbral numérico o lista de keywords que `static_audit.py` puede verificar sin LLM. Es la fuente de verdad que `sync_static_config.py` extrae para generar `static_audit_config.yaml`.
- **R15 — Versionado: vigencia, versión y frescura** (3 campos coordinados):
  - `version_status`: current por defecto → **wip** (principio identificado pero rúbrica/procedimiento sin calibrar; no cargar en skills de producción) → **deprecated** (ya no vale, se conserva por traza) → **superseded** (reemplazada; referenciar la entrada que la sustituye).
  - `platform_version` / `agent_version`: la versión a la que aplica; se fija al crear la entrada.
  - `last_verified`: se actualiza CADA vez que se comprueba la entrada contra la realidad.
  - Al cambiar la versión de la plataforma/agente → revisar sus entradas y actualizar `last_verified` (o marcar deprecated/superseded).

### Meta-instrucción — Aplicación del formato (migración perezosa)

- Entradas **nuevas**: formato completo, obligatorio desde el día 1.
- Entradas **existentes**: se migran al formato nuevo SOLO cuando se tocan por otra razón (corrección, revisión, cambio de contenido). NO hay pasada global — no se toca lo que funciona.
- **Excepción**: si una skill o script nuevo DEPENDE de un campo nuevo (ej. `version_status`), se hace una pasada de ESE campo concreto en todas las entradas ANTES de activar la skill. Mientras nadie dependa del campo, las entradas viejas siguen válidas sin él.

### Ejemplos de referencia

**Concreto (kb_plat_cx):**

````
### CX-13 — Examples como memoria de comportamiento del LLM
fuente: https://cloud.google.com/dialogflow/cx/docs/concept/playbook/best-practices
confianza: Alta
version_status: current
platform_version: Dialogflow CX (v3 general)
last_verified: 2026-06-18
instancia de: A10

QUÉ: los Examples de un Playbook son conversaciones completas (input + expected output) que el
LLM usa como referencia de comportamiento. Aplica a todo playbook de CX. Diseño example-first.
POR QUÉ: determinan la precisión del playbook más que las instrucciones — un playbook sin
examples improvisa formato y se salta pasos.
EJEMPLO ✅: playbook Compra con 4 examples (rosas / tulipanes / sin-stock / error-tool) →
comportamiento estable y previsible.
EJEMPLO ❌: playbook Compra con 0 examples y solo instrucciones largas → el LLM inventa el
formato de respuesta y omite la consulta de inventario.

```static
check: min_examples
min: 4
```
````

**Agnóstico (kb_ag_):** no lleva `platform_version` (no tiene plataforma) y declara `EJEMPLO: no aplica`.

```
### A10 — Los ejemplos de comportamiento valen más que las instrucciones
fuente: hallazgo propio
confianza: Alta
version_status: current
last_verified: 2026-06-18

QUÉ: mostrar ejemplos del comportamiento esperado guía al LLM mejor que describirlo con instrucciones.
POR QUÉ: el modelo generaliza desde patrones concretos mejor que desde reglas abstractas.
EJEMPLO: no aplica — es un principio agnóstico; un caso concreto lo ataría a una plataforma y
rompería su transferibilidad. Las instancias viven en sus capas (ver CX-13).
```

**Dinámico (kb_plat_cx) — sin bloque `static`, verificable solo ejecutando el agente:**

```
### CX-37 — El token budget de un playbook es acumulativo
fuente: https://docs.cloud.google.com/dialogflow/cx/docs/concept/playbook
confianza: Alta (composición documentada) — Baja (valores numéricos no publicados)
instancia de: A23

QUÉ: el budget real por turno = instrucción + examples que CX decide incluir + schemas de tools + historial de conversación + contexto resumido de playbooks anteriores. CX-25 mide solo la instrucción — es proxy útil pero incompleto.
POR QUÉ: un playbook "pequeño" con muchas tools puede agotar el budget igual que uno grande sin tools. El impacto es dinámico: crece turno a turno y CX no expone el consumo real.
EJEMPLO ❌: medir solo la instrucción con tiktoken, dar el playbook por "sano" y no detectar que en el turno 6 los examples empiezan a desaparecer por presión de tokens.
```

---

## 3. Proceso — ciclo de vida, creación y mantenimiento

### El ciclo de vida de un KB (5 fases)

```
CAPTURA → ESTRUCTURA → CURACIÓN → MANTENIMIENTO → RECUPERACIÓN → (vuelta)
```
- **Captura** — de dónde entra el conocimiento: RES (externo) · experiencia propia · QAP (empírico).
- **Estructura** — formato de entrada (sección 2) + capa y nombre (sección 1).
- **Curación** — gate humano: qué entra y qué no (las 2 puertas).
- **Mantenimiento** — revisión, caducidad, versionado, `kb_doctor`.
- **Recuperación** — `_index` dice qué skill carga qué KB.

### Cómo se crea un KB nuevo (archivo)

1. Crear `kb_<capa>_<nombre>.md` con cabecera autodescriptiva: `# KB_<X> — <descripción en una línea>`
2. `python3 kb_doctor.py --fix` → lo añade a `_index` solo, en su sección, con estado 🟡.
3. Completar el **origen** (Authored / Grown) en el registry (sección 1) — único toque manual.
4. `git commit` → el hook `pre-commit` corre `kb_doctor`; si la lista cuadra, pasa.

### Doctrina — decisión semántica vs comprobación mecánica

Dos cosas distintas que no hay que confundir:

| | DECISIÓN (semántica) | COMPROBACIÓN (mecánica) |
|---|---|---|
| Quién | skill LLM (al crear/actualizar) | `kb_doctor.py` (en el commit) |
| Ejemplo | "¿de qué A-code es instancia?" · "¿static o dynamic?" | "¿el A-code que declara existe?" |
| Fiable con `.py` puro | ❌ necesita entender significado | ✅ 100% fiable |

El `.py` **no decide** de qué es instancia ni si encaja con un check — eso es semántico. Solo **valida lo declarado**.

### Recordatorio al crear/tocar entradas (checklist de 3)

`kb_doctor` detecta entradas tocadas y recuerda; la decisión la tomas tú **con el LLM**:

```
□ ¿static, dynamic o ninguna?   → bloque `static` / TC en suite QA / nada
□ ¿de qué A-code(s) es instancia?
□ ¿metadatos obligatorios completos?  (fuente · confianza · version_status · last_verified)
```

- **static** → verificable SIN ejecutar el agente (umbral/keyword) → bloque ` ```static `.
- **dynamic** → solo verificable EJECUTANDO el agente (comportamiento) → TC en la suite QA.
- *(algunas entradas no son ni una ni otra — principio de diseño puro.)*

### El kb_doctor — guardián del gobierno

`~/CD/kb/kb_doctor.py` verifica que el gobierno cuadra con los KBs y entradas en disco.

| Modo | Qué hace |
|---|---|
| `kb_doctor.py` | Detecta y reporta. No toca nada |
| `kb_doctor.py --fix` | Añade a `_index` los KB nuevos (capas ag/sys/plat) leyendo su cabecera |

Comprobaciones (todas mecánicas, fiables):
- **Lista:** KB nuevo (lo añade con `--fix`) · KB borrado (avisa, nunca borra solo).
- **Instancias:** que cada `instancia de: AXX` apunte a un A-code real en `kb_ag_global.md`; A-codes duplicados.
- **Mapa inverso (A-code → instancias):** huérfanos (A-code sin instancia, informativo) · duplicación (mismo A-code instanciado 2+ veces en el mismo KB → posible redundancia, R1/A25).

**Se activa solo** en cada `git commit` de `~/CD/` vía `hooks/pre-commit`. Si algo no cuadra → bloquea el commit (`exit 1`).
No cubre: el **contenido** de cada entrada (eso es la sección 2 + revisión humana/LLM).
En un clon nuevo, reactivar el hook una vez: `git config core.hooksPath hooks`.

### Mapa inverso — propagación de cambios

La relación principio↔instancia es **1:N** (un A-code, muchas instancias) o N:M (una entrada instancia varios A-codes). El mapa inverso que construye `kb_doctor` permite:
- **Propagación** (clave): si un A-code se marca `deprecated` o se reformula, todas sus instancias heredan "revisar" — conecta con R15. Sin el mapa, el cambio se pierde silenciosamente.
- **Huérfanos:** A-code sin ninguna instancia → ¿principio teórico que sobra?
- **Duplicación:** dos entradas del mismo KB instancia del mismo A-code → posible redundancia.

### Regla de oro

Un KB (y un campo, y un tag) se crea **solo cuando una skill o proceso lo consume**. No crear nada "por si acaso".
