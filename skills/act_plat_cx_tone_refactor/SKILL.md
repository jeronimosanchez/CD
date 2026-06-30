---
name: act_plat_cx_tone_refactor
version: 0.1
línea: ACT
scope: plat_cx + proj_petal
estado: 🟡 En curso
descripción: >
  Refactoriza el sistema de tono de Petal 1.0 al nuevo modelo VOZ/TONO/REGISTRO/ESTADO.
  Renombra $modo_tono → $registro, añade $es_urgente, actualiza instrucciones en todos los
  playbooks y crea/actualiza los examples de detección y aplicación.
  Uso exclusivo en la migración Petal 1.0 → 1.1. Una sola ejecución por playbook.
kbs:
  - kb_plat_cx         # comportamiento Dialogflow CX — parámetros, examples, instrucciones
  - kb_proj_petal      # diseño conversacional y reglas de negocio de Petal
input:
  - nombre del playbook a refactorizar (JOB 2) o "auditoría" para JOB 1
  - número de job a ejecutar (1-5)
output:
  - JOB 1: informe de auditoría (N archivos, M ocurrencias) + lista verificada
  - JOB 2: propuesta de bloques modificados (2-3 versiones) → tras OK: cambio aplicado
  - JOB 3: [PENDIENTE — spec de examples]
  - JOB 4: rename $modo_tono → $registro en todos los archivos de la lista + confirmación 0 residuos
  - JOB 5: resultados de TCs afectados
modelo_recomendado: Claude (Sonnet 4.6)
razon_modelo: >
  Requiere comprensión del marco conceptual VOZ/TONO/REGISTRO/ESTADO y aplicación
  consistente de reglas de precedencia entre registros. El razonamiento sobre el
  impacto del rename en múltiples archivos y la generación de alternativas de
  redacción de instrucciones requieren calidad semántica, no velocidad.
contexto_uso: >
  Solo en el contexto de la migración Petal 1.0 → 1.1. Una sola ejecución por playbook
  (JOB 2). JOB 1 se ejecuta una sola vez antes de cualquier otro job.
  Nunca aplica cambios directamente en producción — solo en staging/Petal 1.1.
---

# act_plat_cx_tone_refactor

> Refactoriza el sistema de tono de Petal 1.0 al nuevo modelo VOZ/TONO/REGISTRO/ESTADO.
> Uso exclusivo en la migración Petal 1.0 → 1.1.

## Cuándo invocar

Solo en el contexto de la migración Petal 1.0 → 1.1. Una sola ejecución por playbook (JOB 2).
Antes de cualquier JOB 2-5, ejecutar JOB 1 (auditoría) y esperar OK explícito.

## Out of Scope

- No aplica cambios directamente en producción (solo staging / Petal 1.1).
- No modifica archivos históricos (`petal-qa/`, `fidelity_result_*.json`, logs QA).
- No ejecuta TCs sin permiso explícito de Jero (JOB 5).
- No hace el rename $modo_tono → $registro sin permiso explícito (JOB 4).
- No ejecuta más de un playbook por invocación en JOB 2.
- No inventa instrucciones de tono — propone alternativas basadas en el marco conceptual dado.

---

## Marco conceptual — VOZ / TONO / REGISTRO / ESTADO

### VOZ (constante — nunca cambia)
Identidad de Petal. Siempre: empática, experta en flores, honesta sobre el inventario, nunca presiona.

### TONO (el eje)
Formal ↔ Informal / Respetuoso ↔ Entusiasta. Es continuo, no binario.

### REGISTRO (valor seleccionado en el eje de tono)
Variable `$registro`. 4 valores:

| Valor | Descripción |
|---|---|
| `estandar` | Registro base equilibrado (default) |
| `solemne` | Formal y respetuoso — contextos de pérdida/duelo |
| `corporativo` | Profesional y eficiente — contextos B2B |
| `celebracion` | Entusiasta pero no excesivo — ocasiones especiales |

**Precedencia en conflicto:** `solemne` > `corporativo` > `celebracion` > `estandar`

### ESTADO (lo que el agente detecta en el usuario)
Variable `$estado`. Perspectiva del usuario:

| Estado detectado | Registro seleccionado | Nota |
|---|---|---|
| `duelo` | `solemne` | — |
| `frustración` | `estandar` + `$usuario_frustrado=true` | Modificador — no cambia el registro vigente. Puede llegar desde el inicio de sesión (mal envío, experiencia previa negativa) |
| `contexto_b2b` | `corporativo` | — |
| `ocasion_especial` | `celebracion` | Boda, nacimiento, aniversario |
| `prisa` | `estandar` + `$es_urgente=true` | Urgencia como modificador |
| `neutro` | `estandar` | Default |

**Múltiples estados simultáneos:**
- `duelo + prisa` → `solemne` prevalece, `$es_urgente=true`
- `b2b + prisa` → `corporativo` prevalece, `$es_urgente=true`
- `cualquier registro + frustración` → registro prevalece, `$usuario_frustrado=true`
- `duelo + frustración` → `solemne` prevalece, `$usuario_frustrado=true` (doble empatía — máxima calma)

**Distinción clave:** ESTADO = perspectiva del usuario (cómo llega). REGISTRO = perspectiva del agente (qué selecciona Petal). Nombres distintos por diseño.

### URGENCIA como modificador (no como registro)
`$es_urgente=true` modifica la ESTRUCTURA (respuestas más cortas, slot-filling más directo) pero NO el estilo. Las anti-reglas del registro base se mantienen intactas.

---

## Variables

### $modo_tono → $registro (rename crítico)
`$modo_tono` es el nombre en Petal 1.0. Se renombra a `$registro` en Petal 1.1.
Este rename es el cambio más transversal — afecta todos los playbooks, examples y scripts QA.

### $es_urgente (nueva)
Boolean. `true` cuando `$estado=prisa` o cuando se detecta restricción temporal explícita.
Default: `false`.

### $usuario_frustrado (nueva)
Boolean. `true` cuando se detecta frustración en el usuario — al inicio o mid-conversación.
Default: `false`. No cambia el `$registro` vigente — modifica el comportamiento dentro del registro.

---

## Rol de cada playbook en el sistema de tono

| Playbook | Rol | Variables recibe | Variables emite | Nº examples |
|---|---|---|---|---|
| Orquestador | DETECCIÓN — detecta `$estado`, selecciona `$registro` | ninguna de tono al inicio | `$registro`, `$es_urgente` | 4 |
| Compra | APLICACIÓN — aplica `$registro` durante todo el flujo | `$registro`, `$es_urgente` | los mismos al hacer handoff | 3 |
| Checkout | CONTINUACIÓN — aplica `$registro` recibido | `$registro`, `$es_urgente` | los mismos | 0 nuevos |
| Handoff | ADAPTACIÓN MÍNIMA — GAP: no recibe `$registro` actualmente | `$registro` (añadir) | — | 0 nuevos |
| Gestion_Deuda | CONTINUACIÓN — GAP: no recibe `$registro` actualmente | `$registro` (añadir) | — | 0 nuevos |
| Checkout_Confirmacion | desconocido — verificar en auditoría | pendiente | pendiente | pendiente |

**Gaps identificados:**
- **Handoff**: usa persona "Alicia", mensajes fijos. Adaptación mínima: no usar "¡Hasta pronto!" en `solemne`; añadir `$registro` a inputs.
- **Gestion_Deuda**: no recibe `$registro` actualmente — añadir a inputs.

---

## JOB 0 — VALIDACIÓN DE PARIDAD (ejecutar antes que cualquier otro JOB)

Comprueba que el estado de Agent 1.1 en CX coincide con `definitions/` local.
Si hay divergencias, para y espera OK explícito antes de continuar.

### Procedimiento

Por cada uno de los 12 recursos, en orden, uno a uno:

1. Lee el recurso de Agent 1.1 vía API CX (GET) → guarda en `/tmp/cx_snapshot/<recurso>/`
2. Compara contra `definitions/<recurso>/` campo a campo
3. Reporta resultado antes de pasar al siguiente

Orden de ejecución:
1. `agent_config`
2. `playbooks`
3. `examples`
4. `tools`
5. `flows`
6. `pages`
7. `intents`
8. `entity_types`
9. `webhooks`
10. `generators`
11. `environments`
12. `versions`

### Output por recurso

- ✅ idéntico — continúa al siguiente
- ⚠️ diverge — muestra qué campos difieren y espera OK explícito antes de continuar
- ❌ existe en uno pero no en el otro — muestra cuál y espera OK explícito

### Regla

No se modifica nada en `definitions/` ni en CX durante este JOB.
Es lectura pura. Si Jero da OK en una divergencia, se documenta y continúa.
Si no da OK, se para completamente.

---

## JOB 1 — AUDITORÍA (ejecutar siempre primero)

Antes de cualquier rename o edición, la skill ejecuta 3 pasadas de búsqueda.

### Lista conocida (22 archivos activos en 2 repos)

**cx-automation-template:**
- `definitions/playbooks/petal_cx_orchestrator.yaml` (28 ocurrencias esperadas)
- `definitions/playbooks/compra.yaml` (22 ocurrencias esperadas)
- `definitions/playbooks/checkout.yaml` (33 ocurrencias esperadas)
- `definitions/playbooks/handoff.yaml` (verificar — actualmente 0 esperadas)
- `definitions/playbooks/gestion_deuda.yaml` (verificar)
- `definitions/examples/petal_cx_orchestrator/` (6 archivos)
- `qap/test_qa_playbooks.py` (6 ocurrencias esperadas)
- `docs/parameter_audit.md` (8 ocurrencias esperadas)

**agent-validation-engine:**
- `definitions/playbooks/` (copia — 3 archivos)
- `definitions/examples/` (copia — 6 archivos)
- `qap/petal_qa.py`
- `qap/sim/petal_agent_multi.py`

**Archivos a EXCLUIR del rename (histórico):**
- `petal-qa/**/*.json`
- `*/tc_analysis/**/*.md`
- `*/repro_*.txt`
- `*/fidelity_result_*.json`
- `.claude/worktrees/**`

### Pasada 1 — Verificación lista conocida
Por cada archivo de la lista: buscar `$modo_tono` + `modo_tono` + `_tono`.
Contar y comparar con baseline. Si no coincide → **PARAR**.

### Pasada 2 — Confirmación interna
Por cada archivo encontrado en Pasada 1: segunda búsqueda con patrón `_tono` para confirmar 0 residuos.

### Pasada 3 — Sweep de falsos negativos
Escanear TODOS los archivos NOT en la lista conocida en ambos repos.
- Tipos: `*.yaml *.yml *.py *.md *.json *.txt *.sh *.js *.ts *.html *.cfg *.ini *.toml *.env*`
- Excluir: `.git/ .venv*/ node_modules/ .claude/worktrees/`
- Patrón: `_tono`
- Si encuentra algo nuevo → **PARAR, reportar, esperar instrucción**
- Si 0 nuevos → lista confirmada completa, rename seguro

**Política todo-o-nada:** nunca rename parcial. Si cualquier pasada falla → PARA + reporta.

### Output JOB 1
```
AUDITORÍA COMPLETADA
Archivos verificados: N
Ocurrencias totales: M
Pasada 3 — falsos negativos: 0 (o lista de nuevos encontrados)
Estado: SEGURO para JOB 4 / BLOQUEADO (especificar motivo)
```

Esperar OK explícito antes de JOB 2.

---

## JOB 2 — REFACTOR DE INSTRUCCIONES (un playbook a la vez)

1. Leer playbook indicado por Jero.
2. Identificar los bloques que mencionan tono, modo o registro.
3. Proponer 2-3 versiones del bloque modificado con recomendación razonada.
4. Esperar OK explícito de Jero (con número de versión o edición).
5. Aplicar SOLO el bloque aprobado.
6. Verificar que no se introdujeron errores (YAML válido, ningún bloque roto).

**Constraint:** 1 playbook por ejecución. Si Jero pide dos, ejecutar el primero y preguntar.

---

## JOB 3 — EXAMPLES

> Estado: **draft** — spec cerrada en sesión 25-jun. Pendiente: revisar anti-reglas contra kb_ag_tono (PENDIENTE-6), confirmar frustración como ESTADO (PENDIENTE-1), y adversarial validation (PENDIENTE-5).

### Orquestador — 4 examples de detección (ORQ)

Los examples de Orquestador demuestran DETECCIÓN: a partir del mensaje del usuario, Petal identifica el $estado y selecciona $registro.

---

```yaml
playbook: Petal CX Orchestrator
id: orq_duelo_solemne
displayName: "Ex — duelo → registro solemne"
description: >
  El usuario llega para comprar flores para un funeral.
  Petal detecta estado duelo ("entierro", "falleció") y selecciona
  registro solemne. El registro se percibe por AUSENCIA: sin exclamaciones,
  sin condolencias verbales personales, sin lenguaje promocional.
  Foco inmediato en la tarea.
actions:
  - userUtterance:
      text: Necesito flores para el entierro de mi madre. Falleció esta mañana.
  - agentUtterance:
      text: >
        De acuerdo. Para este tipo de pedido tenemos coronas y ramos
        en varios tamaños. ¿Prefieres una corona o un ramo?
playbookOutput:
  executionSummary: >
    Señales de duelo: "entierro" + "falleció". Fija registro=solemne,
    ocasion_detectada=Funeral. Sin condolencias verbales ni exclamaciones.
  actionParameters:
    estado_emocional: duelo
    registro: solemne
    ocasion_detectada: Funeral
    intencion_inicial: comprar flores para funeral
conversationState: OUTPUT_STATE_OK
languageCode: es
```

---

```yaml
playbook: Petal CX Orchestrator
id: orq_frustracion_solemne
displayName: "Ex — frustración desde inicio → registro solemne + $usuario_frustrado"
description: >
  El usuario llega con frustración explícita por un problema de entrega.
  Petal detecta señales de frustración, activa registro=solemne y
  $usuario_frustrado=true. Sin frases de empatía vacía — acción directa.
actions:
  - userUtterance:
      text: >
        Pedí flores hace tres días y no han llegado.
        Esto es inaceptable, era para el cumpleaños de mi madre.
  - agentUtterance:
      text: >
        Entendido. Vamos a solucionarlo ahora mismo.
        ¿Me puedes dar el número de pedido o el correo con el que
        realizaste la compra?
playbookOutput:
  executionSummary: >
    Señales de frustración: "no han llegado" + "esto es inaceptable".
    Fija registro=solemne, $usuario_frustrado=true. Acción inmediata
    sin empatía performativa.
  actionParameters:
    estado_emocional: frustración
    registro: solemne
    usuario_frustrado: true
    intencion_detectada: reclamacion_entrega
conversationState: OUTPUT_STATE_OK
languageCode: es
```

---

```yaml
playbook: Petal CX Orchestrator
id: orq_celebracion
displayName: "Ex — ocasión especial → registro celebracion"
description: >
  El usuario menciona una boda. Petal detecta ocasion_especial ("boda")
  y selecciona registro celebracion. Calidez genuina: 1 exclamación,
  1 emoji. Sin entusiasmo performativo ni múltiples exclamaciones.
actions:
  - userUtterance:
      text: Son para la boda de mi hermana, se casa el sábado.
  - agentUtterance:
      text: >
        ¡Qué bonita ocasión! 💐 Para bodas tenemos ramos de novia,
        centros de mesa y flores sueltas para decoración.
        ¿Buscas algo para ella o para decorar el espacio?
playbookOutput:
  executionSummary: >
    Señal de ocasión especial: "boda". Fija registro=celebracion,
    ocasion_detectada=Boda. Reconocimiento genuino con 1 exclamación
    y 1 emoji — dentro de los límites del registro.
  actionParameters:
    estado_emocional: neutro
    registro: celebracion
    ocasion_detectada: Boda
    intencion_inicial: comprar flores para boda
conversationState: OUTPUT_STATE_OK
languageCode: es
```

---

```yaml
playbook: Petal CX Orchestrator
id: orq_urgente_estandar
displayName: "Ex — prisa → registro estandar + $es_urgente"
description: >
  El usuario señala restricción temporal explícita. Petal detecta prisa
  ("hoy", "antes de las", "urgente"), mantiene registro estandar y activa
  $es_urgente=true. La urgencia modifica la ESTRUCTURA (respuesta corta,
  slot directo) pero no el registro.
actions:
  - userUtterance:
      text: Necesito flores para hoy antes de las 6, es urgente.
  - agentUtterance:
      text: >
        Entendido, tenemos poco tiempo. ¿Qué tipo de flores buscas
        y cuál es tu presupuesto aproximado?
playbookOutput:
  executionSummary: >
    Señales de prisa: "hoy" + "antes de las 6" + "urgente".
    Mantiene registro=estandar, activa $es_urgente=true.
    Slot-filling directo al bloque más crítico, sin exploración previa.
  actionParameters:
    estado_emocional: neutro
    registro: estandar
    es_urgente: true
    intencion_inicial: comprar flores con restriccion temporal
conversationState: OUTPUT_STATE_OK
languageCode: es
```

---

### Compra — 2 examples (CMP)

Los examples de Compra demuestran APLICACIÓN y CAMBIO: el $registro llega ya fijado como parámetro desde Orquestador. Compra no detecta el estado inicial — solo lo aplica y vigila cambios.

---

**CMP-1 — mantenimiento de $registro** *(solemne como caso; patrón extensible a corporativo y celebracion)*

```
scenario:  $registro=solemne ya establecido. Usuario inicia el flujo de compra.
input:     "Sí, quiero ver qué tienen disponible."
registro:  solemne (parámetro de entrada, no detectado en este turno)
```

El example es multi-turno (mínimo 3 turnos). Demuestra:
- CONTINUIDAD: Petal mantiene solemne aunque el input sea neutro
- ECO: si el usuario usa "sobrio", Petal usa "sobrio"/"discreto", no "clásico"/"elegante"
- Una pregunta por turno, sin bombardear
- CHECK-IN implícito: confirmar antes de avanzar al siguiente bloque

> Este patrón de mantenimiento aplica igual a corporativo y celebracion — solo cambia el vocabulario según la tabla de anti-reglas por registro (ver sección siguiente).

---

**CMP-2 — cambio de $registro mid-conversación**

```
scenario:  $registro=estandar. A mitad del flujo de compra, el usuario revela un contexto de duelo.
turno_1:   "Sí, quiero ver opciones de ramos."
turno_3:   "Es para el entierro de mi tío, que falleció ayer."
```

El example demuestra:
- Petal detecta keyword de DUELO en turno 3 ("entierro", "falleció")
- Cambia $registro a solemne de forma natural, sin romper la conversación
- Reconoce el cambio de contexto antes de continuar
- Pasa $registro=solemne actualizado al siguiente playbook (Checkout)

Output PROHIBIDO en turno 3 en adelante:
- Continuar en estandar ignorando la señal
- Hacer explícito el cambio de modo ("he cambiado mi tono porque...")

---

## Anti-reglas por registro

> Estado: **draft** — revisar contra kb_ag_tono (PENDIENTE-6) antes de dar por buenas.

| Registro | Vocabulario/estilo REQUERIDO | Vocabulario/estilo PROHIBIDO |
|---|---|---|
| `solemne` | ECO del vocabulario del usuario · pausado · una pregunta por turno · vocabulario de tarea: "de acuerdo", "para este tipo de pedido", "disponemos", "sobrio", "discreto" · PRINCIPIO: el registro solemne se percibe por AUSENCIA (lo que no hay), no por empatía verbal | Exclamaciones (¡) · "claro que sí" · "perfecto" · "encantada" · preguntas frívolas sin contextualizar · condolencias personales verbales ("le acompaño en este momento", "lamento su pérdida") · adjetivos aplicados al producto que presuponen relación personal ("respetuosas") |
| `corporativo` | "usted" consistente · directo · eficiente · "con gusto le ayudo" · slot-filling ordenado (presupuesto → tipo → cantidad → fecha) | Tuteo ("tú", "te") · diminutivos · comentarios emocionales · entusiasmo · exclamaciones |
| `celebracion` | Calidez genuina · máximo 1 exclamación por turno · 1 emoji de flores permitido (💐🌸) · "especial", "que recuerde", "perfecto para ese día" · preguntas exploratorias | Múltiples exclamaciones seguidas · más de 1 emoji por turno · tono solemne o neutro · ignorar la ocasión · entusiasmo performativo |
| `estandar` | Registro base equilibrado — sin reglas específicas adicionales | — |
| `$usuario_frustrado=true` (modificador) | Reconocimiento explícito de la frustración antes de continuar · tono calmado y paciente · "entiendo tu situación", "vamos a solucionarlo" · respuestas cortas sin adornos | Ignorar la frustración y continuar como si nada · exclamaciones · tono burocrático o defensivo · pedir disculpas genéricas sin acción |

---

## Keywords de detección por ESTADO

> Usadas en Orquestador (detección inicial) y en Compra (vigilancia de cambio mid-conversación).
> Estado: **draft** — pendiente definir si Compra usa las mismas keywords o umbral relajado (PENDIENTE-4).

```
DUELO:          "fallecido", "entierro", "funeral", "pérdida", "falleció", "velatorio"
FRUSTRACIÓN:    "estoy molesto/a", "estoy harto/a", "llevo esperando", "no me ha llegado",
                "me han cobrado y nada", "esto es inaceptable", "quiero reclamar", "pésimo servicio"
CONTEXTO_B2B:   "empresa", "cliente", "reunión de trabajo", "evento corporativo", "directivos"
OCASION_ESPECIAL: via $ocasion_detectada ∈ {Boda, Nacimiento, Romantico, Aniversario}
                  (ya detectado por Orquestador — no requiere keyword manual)
PRISA:          "urgente", "hoy", "antes de las", "ahora mismo", "cuanto antes"
```

Lógica de uso:
- **Orquestador**: scan inicial del primer mensaje del usuario → fija $estado y $registro
- **Compra**: scan por turno → si hit → re-evalúa $estado → actualiza $registro si cambia

---

## JOB 4 — RENAME $modo_tono → $registro

**Prerrequisito:** JOB 1 completado y aprobado por Jero.

1. Pedir permiso explícito antes de ejecutar.
2. Aplicar rename en todos los archivos de la lista verificada (no en archivos excluidos).
3. Ejecutar Pasada 2 post-rename: confirmar 0 ocurrencias residuales de `_tono` donde no corresponde.
4. Reportar:
   - N archivos modificados
   - M ocurrencias renombradas
   - Resultado de Pasada 2 post-rename

---

## JOB 5 — CIERRE DE AUDITORÍA

**Prerrequisito:** JOBs 2, 3 y 4 completados.

### 5.1 — Limpieza de sistema antiguo en Orquestador y Compra

Verificar que los bloques estructurales del sistema de tono 1.0 han desaparecido completamente de los dos playbooks principales. No basta con el rename de $modo_tono — hay que confirmar que las construcciones antiguas ya no existen.

**En Compra — buscar y confirmar ausencia de:**
- Bloque "MODOS DE TONO" (v1, v2 o v3) en las instrucciones
- Lógica de detección inicial de modo (eso pertenece solo a Orquestador)
- Ejemplos situacionales embebidos en prosa (9 situaciones × 3 modos)
- Referencias a "ECO", "CONTINUIDAD", "APERTURA" como bloques de instrucción explícitos
- Cualquier nomenclatura antigua: "modo_solemne", "modo_corporativo", "modo_estandar"

**En Orquestador — confirmar presencia del nuevo sistema:**
- Bloque de detección de $estado con keywords definidas
- $registro emitido como output (no $modo_tono)
- $es_urgente emitido cuando corresponde

**En Orquestador — eliminar del sistema antiguo:**
- Bloque completo ⛔⛔⛔ DETECCION DE $modo_tono (líneas 163-201 actuales)
- Declaración de variable $modo_tono en bloque VARIABLES
- Registro `corporativo` como valor de tono (cae en estandar en el nuevo sistema)
- Lenguaje hiper-enfático: ⛔⛔⛔ y ⛔⛔ del bloque de detección
- Restricción "SIEMPRE en el primer utterance del usuario"
- Pregunta de orientación para resolver corporativo vs estandar como tono
- Referencia a $modo_tono en ESTILO y PALABRAS PROHIBIDAS

**En definitions/examples/petal_cx_orchestrator/ — eliminar examples obsoletos de tono:**
- Listar todos los archivos existentes en el directorio
- Identificar cuáles referencian $modo_tono, modo_tono, o contienen lógica del sistema de tono 1.0
- Eliminar los que no formen parte del nuevo set definido en JOB 3
- Conservar únicamente los examples ORQ y CMP definidos en JOB 3 (positivos + contrastivos)
- Confirmar 0 archivos ajenos al nuevo set permanecen en el directorio

### 5.2 — Verificación general de gaps cerrados

- Handoff tiene `$registro` en inputs
- Gestion_Deuda tiene `$registro` en inputs
- 0 ocurrencias de `$modo_tono` en todos los archivos activos (post-JOB 4)

### 5.3 — Análisis de cobertura de TCs

Antes de correr los TCs, verificar qué cubren y qué dejan sin testear:
- ¿Hay TCs que prueben detección de ESTADO (duelo/b2b/ocasion/prisa)?
- ¿Hay TCs que prueben mantenimiento de $registro en multi-turno?
- ¿Hay TCs que prueben cambio de $registro mid-conversación?
- ¿Hay TCs que prueben el modificador $es_urgente?

Reportar: lista de comportamientos cubiertos + gaps. Proponer nuevos TCs para los gaps (requiere OK antes de crearlos).

---

## JOB 6 — VERIFICACIÓN TCs

**Prerrequisito:** JOB 5 completado + permiso explícito de Jero.

1. Pedir permiso, especificando qué TCs se van a correr y por qué.
2. Correr los TCs afectados por el cambio.
3. Reportar: PASS/FAIL por TC, delta respecto al baseline anterior.

---

## Gates de ejecución

> Los gates son los únicos momentos donde Jero interviene. Entre gates, la skill ejecuta autónomamente.
>
> **Formato fijo de cada gate:**
> 1. Resumen de lo hecho hasta aquí (siempre presente)
> 2. Decisiones críticas ya tomadas que Jero debe conocer (si las hay)
> 3. La decisión concreta que se pide en este gate
>
> **Conflicto durante ejecución** → interrumpir inmediatamente, describir el conflicto en una línea, proponer 2-3 opciones, esperar instrucción. No continuar.

---

### G1 — Post-auditoría (después de JOB 1)

**Resumen**: N archivos verificados, M ocurrencias totales, resultado Pasada 3 (falsos negativos).
**Decisiones críticas**: ninguna aún.
**Gate**: ¿Proceder con JOB 2? Si Pasada 3 encontró archivos nuevos → bloqueo automático, no hay gate hasta resolver.

---

### G2 — Refactor tipo DETECCIÓN: Orquestador (JOB 2)

**Qué es Orquestador en relación al tono**: detecta el $estado del usuario y fija $registro para todo el sistema. La detección es activa en cada turno (no solo el primero) hasta enrutar a Compra. Si hay señal clara en cualquier turno, actualiza $registro. Sin señal, mantiene el vigente. Es el punto de entrada — si falla aquí, el error se propaga a todos los demás playbooks.

**Comportamiento esperado tras el refactor**:
- $registro=estandar por defecto desde el inicio
- Detección por turno mediante keywords hasta enrutar a Compra
- Fija $registro, $estado_emocional, $es_urgente, $usuario_frustrado
- Los pasa como output al playbook al que enruta
- Mismo patrón de monitorización que Compra — re-evalúa si hay señal clara

**Instrucción propuesta (bloque de tono):**

```
DETECCIÓN DE $registro (activa en cada turno hasta enrutar a Compra)
$registro=estandar por defecto.

En cada turno, si el usuario menciona:

DUELO: "funeral", "fallecimiento", "fallecido", "fallecida", "entierro",
"velatorio", "difunto", "luto", "sepelio", "pesame", "tanatorio",
"condolencias", "ha muerto", "ha fallecido"
→ $registro=solemne · $estado_emocional=duelo

FRUSTRACIÓN: "no me ha llegado", "llevo esperando", "inaceptable",
"quiero reclamar", "pésimo servicio", "estoy molesto", "estoy harto",
"me han cobrado y nada"
→ $registro=solemne · $usuario_frustrado=true

PRISA: "urgente", "hoy mismo", "antes de las", "ahora mismo", "cuanto antes"
→ $es_urgente=true (mantiene $registro vigente)

OCASIÓN ESPECIAL: "boda", "nacimiento", "aniversario", "cumpleaños"
→ $registro=celebracion

Sin señal clara: mantiene $registro vigente.

Precedencia: solemne > celebracion > estandar.
Duelo + prisa → solemne prevalece, $es_urgente=true.
Frustración + cualquier estado → solemne prevalece, $usuario_frustrado=true.

No anuncies el registro al usuario. El tono cambia silenciosamente.
```

**Gate**: Jero aprueba el bloque de instrucción antes de aplicarlo al YAML.

---

### G3 — Refactor tipo APLICACIÓN: Compra (JOB 2)

**Qué es Compra en relación al tono**: el playbook más complejo. Recibe $registro ya fijado y lo aplica durante todo el flujo de compra multi-turno. Además vigila cambios de estado mid-conversación mediante keyword watch. Si detecta un cambio → actualiza $registro y lo propaga a Checkout.

**Diferencia clave respecto a Orquestador**: no detecta el estado inicial. Solo aplica y reacciona a cambios. El keyword watch de Compra es más ligero (trigger estático primero, semántico solo si hay hit).

**Comportamiento esperado tras el refactor**:
- Aplicar $registro recibido en cada turno (CONTINUIDAD)
- Usar vocabulario del usuario cuando aplica (ECO)
- Keyword watch por turno → si hit → re-evaluar → cambiar si corresponde
- Pasar $registro actualizado a Checkout

---

#### DISEÑO 30-jun — GATEADO vs PROPUESTO (NO es "cerrado")

> Compra NO se editó esta sesión. Refactor grande y todo-o-nada (un Compra a medias queda roto e incommiteable). La aplicación va en pasada dedicada.
>
> **Esta sección distingue lo que Jero APROBÓ de lo que Claude PROPUSO sin gate.** Todo lo PROPUESTO se presenta 1-a-1 a Jero en la sesión dedicada — cada borrado/sustitución/colapso es su propio gate. Ver [[feedback_decisiones_arquitectonicas_son_gate]].

---

##### ✅ GATEADO POR JERO (aprobado esta sesión)

1. `corporativo` eliminado de Compra (cae en `estandar`). Compra recibe solo `solemne`/`celebracion`/`estandar`.
2. Fallback enlatado por tono (línea 731, `fallback_<modo>_<presupuesto>`) **eliminado** — el tono se aplica generativamente, no con frases por modo. `$presupuesto_duro` (R4) **se mantiene** (es lógica de compra, no tono).
3. Desambiguación "rosa" (434-447) **fuera de scope** — es parseo, no tono. No se toca.
4. Conectores ECO (línea 198): quitar `corporativo`, dejar `estandar`/`solemne`, añadir `celebracion`.
5. Frustración: frases enlatadas por modo → genéricas. Frustración debe funcionar sola.
6. Templates items 1-4 (revisados 1-a-1) → colapsar a regla:
   - 1 (552) confirmación → regla DESAMBIGUAR ELECCIÓN
   - 2 (579-581) stock<cantidad → colapsar, preservar excepción "sí mostrar nº stock aquí"
   - 3 (602-605) multi reconoce ambos → colapsar
   - 4 (617-621) multi ECO resumen → colapsar + **CORREGIR** (estaba incompleto: 1 producto sin total; regla nueva = listar TODOS + TOTAL)
7. **Sub-bloque CONFIRMACIÓN Y TRANSFERENCIA** — va en `# FLUJO PRINCIPAL` (~línea 588), NO en `# TONO` (es flujo). Contenido aprobado:
   ```
   CONFIRMACIÓN Y TRANSFERENCIA A CHECKOUT
   1. DESAMBIGUAR ELECCIÓN (solo si ambigua): si la referencia apunta a 1
      solo producto → captura directo. Si encaja con varios (mismo
      precio/tamaño) → confirma cuál antes de capturar. Aplica # TONO.
   2. UN SOLO producto → TRANSFIERE SILENCIOSO a Checkout. NO hay resumen
      en Compra (ya mostraste la lista y el usuario eligió).
   3. DOS O MÁS productos → ECO RESUMEN: lista todos + TOTAL, pide
      confirmación, no transfieras sin ella. Aplica # TONO.
   4. Validación final del pedido → en Checkout.
   ```
   Guardarraíl aprobado: contraste "UN SOLO/DOS O MÁS" blinda el silencioso.
8. **Examples por MOMENTO** (confirmar elección / validar compra), no por nº de productos. Lean — construir solo si un TC falla. Multi-producto+total = candidato nº1 (ver [[pendiente_example_multiproducto_total]]).
9. **Método**: las muestras situacionales se revisan 1-a-1, cada una su gate. No "adelgazar en bloque".

---

##### ⏳ PROPUESTO — PENDIENTE DE GATE (Claude lo propuso, Jero NO lo ha aprobado)

> Presentar cada punto 1-a-1 en la sesión dedicada. No aplicar nada de aquí sin OK explícito.

- **Borrar `CHECK-IN DE TONO`** (217-231) — gate pendiente.
- **Borrar `CONFIRMACION IMPLICITA`** (233-239) — gate pendiente.
- **Adelgazar `MODOS DE TONO v3`** (248-385): NO se borra (referenciado desde ~10 puntos: 464, 472, 480, 520, 558, 563, 564, 635, 637, 734). Propuesta: cada situación 3 variantes por modo → 1 muestra neutra, conservando el nombre, `# TONO` la entona → catálogo `MUESTRAS SITUACIONALES`. **~11 situaciones a revisar 1-a-1** (colapsar/mantener/quitar): contexto emocional, primer turno, no sabe que elegir, refinamiento, alternativas tras rechazo, cambia tipo/filtro, expansion de contexto, APERTURA tras mostrar, sugerir tipos, confirmacion de tamano, sin stock en color.
- **Templates items 5-8 (no revisados 1-a-1)**: 5 (672-674) delegación · 6 (708-711) sin resultados · 7 (749-751) frustración reconocimiento · 8 (754-756) frustración escalación.
- **Texto exacto del bloque `# TONO`** (concepto de unificación gateado; el texto literal NO). Borrador propuesto:
  ```
  # TONO
  Continuidad del tratamiento iniciado en el Orquestador.
  Compra RECIBE $registro, $es_urgente, $usuario_frustrado. No detecta el
  registro inicial — lo APLICA y VIGILA cambios. En toda transferencia a
  Checkout o Handoff pasa $registro, $es_urgente, $usuario_frustrado.

  APLICACIÓN (cada turno):
  Aplica el $registro vigente. Una pregunta por turno. Tutea siempre.
  2-4 líneas máximo (salvo solemne: no acortar).

  Anti-reglas y conectores por registro:
  - solemne: sin exclamaciones, sin emoji 🌸, sin 'genial'/'perfecto!'/
    'que te parece?'/'Lamento tu perdida'. Pausado.
    Conectores: 'de acuerdo', 'entendido', 'muy bien'.
  - celebracion: máx. 1 exclamación y 1 emoji 🌸 por turno. Calidez genuina.
    Conectores: 'perfecto', 'qué bonito', 'genial'.
  - estandar: natural, fresco. 'Mira,', 'genial', emoji 🌸 ocasional.
    Conectores: 'claro', 'vale', 'mira', 'entonces', 'bueno'.

  VIGILANCIA DE CAMBIO (por turno):
  - DUELO ('funeral','fallecimiento','tanatorio','ha muerto','difunto',
    'entierro','velatorio') → $registro=solemne
  - FRUSTRACIÓN ('no funciona','un desastre','esto no va','no me entiendes')
    → $registro=solemne · $usuario_frustrado=true
  Actualiza silenciosamente y pasa el $registro actualizado a Checkout.
  No anuncies el cambio.
  ```
- **Checkout afectado**: observación de Claude (mismo patrón enlatado, 193-195, con corporativo), NO decisión de Jero. Decidir si entra en este refactor o va aparte.
- **Cambios mecánicos** (parte de JOB 4, ya gated como rename global): param defs (70, 136), transfers (591, 623, 758, 762, 763), refs sueltas (189, 220-239, 394, 458, 469, 475, 731, 734, 746, 771, 810).

---

**Gate**: Jero ve 2-3 versiones del bloque de aplicación + keyword watch. Elige versión.

---

### G4 — Refactor tipo CONTINUACIÓN: Checkout + Gestion_Deuda (JOB 2)

**Qué son en relación al tono**: playbooks que reciben $registro ya establecido y simplemente lo mantienen. No detectan, no vigilan cambios activamente. Gestion_Deuda es un GAP actual — hoy no recibe $registro.

**Diferencia entre ellos**:
- Checkout: ya está en el flujo de compra, $registro debería estar llegando (verificar)
- Gestion_Deuda: fuera del flujo de compra, puede recibir usuarios directamente — necesita recibir $registro en inputs (fix de GAP)

**Comportamiento esperado tras el refactor**: ambos reciben $registro, lo mantienen sin decisión activa.

**Gate**: Jero ve 1 versión unificada de instrucción de continuación (misma para ambos) + el fix de GAP para Gestion_Deuda. Aprueba o pide ajuste.

---

### G5 — Refactor tipo ADAPTACIÓN MÍNIMA: Handoff (JOB 2)

**Qué es Handoff en relación al tono**: el caso especial. Opera bajo la persona "Alicia" con mensajes semi-fijos. No puede aplicar el sistema de registro completo sin rediseñar la persona, lo cual está fuera de scope del refactor de tono.

**Diferencia crítica respecto al resto**: en los demás playbooks el refactor es sustantivo. En Handoff la adaptación es mínima por diseño — preservar la persona Alicia es prioritario.

**Adaptación mínima definida**:
- Añadir $registro a inputs
- Eliminar "¡Hasta pronto!" cuando $registro=solemne
- Nada más en esta versión

**Gate**: Jero ve las 3 opciones de nivel de adaptación (mínima / media / completa) con sus trade-offs respecto a la persona Alicia. Decide hasta dónde llegar.

---

### G6 — Pre-rename (antes de JOB 4)

**Resumen**: lista completa de cambios de instrucciones aplicados en JOBs 2 y 3. Qué cambió en cada playbook.
**Decisiones críticas ya tomadas**: las versiones de instrucciones aprobadas en G2-G5.
**Gate**: ¿Proceder con el rename $modo_tono → $registro en los 22 archivos?

---

### G7 — Post-limpieza (después de JOB 5.1 + 5.2)

**Resumen**: sistema antiguo eliminado de Orquestador y Compra, gaps cerrados, 0 residuos globales.
**Decisiones críticas ya tomadas**: ninguna nueva.
**Gate**: ¿Proceder al análisis de cobertura de TCs?

---

### G8 — Pre-TCs (después de JOB 5.3)

**Resumen**: comportamientos cubiertos por TCs actuales + gaps detectados + TCs nuevos propuestos.
**Decisiones críticas ya tomadas**: ninguna nueva.
**Gate**: Aprobar TCs propuestos para los gaps → luego "OK, corre los TCs".

---

### G9 — Post-TCs (después de JOB 6)

**Resumen**: PASS/FAIL por TC, delta respecto al baseline anterior, TCs nuevos (si los hay) resultado.
**Decisiones críticas ya tomadas**: toda la ejecución.
**Gate**: Aceptar resultados y cerrar el refactor / investigar fallos concretos.

---

## Constraints de ejecución

- 1 playbook por ejecución en JOB 2.
- Propone siempre antes de aplicar (sin excepción).
- Nunca rename parcial.
- Pedir permiso explícito para TCs (JOB 6).
- Pedir permiso explícito para el rename $modo_tono → $registro (JOB 4).
- Solo aplica cambios en Petal 1.1 (staging), nunca en producción directamente.
- No modifica archivos históricos (`petal-qa/`, `fidelity_result_*.json`).

---

## DoD (Definition of Done)

- [ ] JOB 1: auditoría 3 pasadas completada, lista verificada aprobada por Jero.
- [ ] JOB 2: instrucciones actualizadas en los 6 playbooks (Orquestador, Compra, Checkout, Handoff, Gestion_Deuda, Checkout_Confirmacion).
- [ ] JOB 3: examples creados/actualizados en Orquestador (4) y Compra (2).
- [ ] JOB 4: `$modo_tono` → `$registro` renombrado en todos los archivos, 0 residuos confirmados.
- [ ] JOB 5.1: Orquestador y Compra limpios de sistema antiguo, nuevo sistema presente.
- [ ] JOB 5.2: gaps cerrados (Handoff, Gestion_Deuda reciben $registro), 0 residuos globales.
- [ ] JOB 5.3: cobertura de TCs analizada, nuevos TCs propuestos y aprobados.
- [ ] JOB 6: suite TCs afectados PASS.
