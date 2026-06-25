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
| `contexto_b2b` | `corporativo` | — |
| `ocasion_especial` | `celebracion` | Boda, nacimiento, aniversario |
| `prisa` | `estandar` + `$es_urgente=true` | Urgencia como modificador |
| `neutro` | `estandar` | Default |

**Múltiples estados simultáneos:**
- `duelo + prisa` → `solemne` prevalece, `$es_urgente=true`
- `b2b + prisa` → `corporativo` prevalece, `$es_urgente=true`

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

**ORQ-1 — duelo → solemne**

```
scenario: Usuario llega para comprar flores para un funeral
input:    "Necesito flores para el entierro de mi madre. Falleció esta mañana."
estado:   duelo
registro: solemne
```

Output DEBE incluir:
- Expresión de condolencias breve, sin theatralidad
- Vocabulario: "acompañar", "flores que expresen respeto", "discreto"
- Una sola pregunta, tono pausado

Output PROHIBIDO:
- "¡Claro que sí!", "¡Perfecto!", "¡Encantada!" — cualquier exclamación
- "¿Tiene alguna preferencia de color?" como primera pregunta (frívolo sin contextualizar)
- Cambiar a tono animado en cualquier turno

---

**ORQ-2 — contexto_b2b → corporativo**

```
scenario: Usuario llega con contexto empresarial explícito
input:    "Soy el responsable de eventos corporativos de mi empresa. Necesitamos flores para una reunión con clientes importantes la próxima semana."
estado:   contexto_b2b
registro: corporativo
```

Output DEBE incluir:
- Trato de "usted" desde el primer turno
- "con gusto le ayudo", "entiendo el contexto", "para ese tipo de evento"
- Pregunta directa sobre presupuesto o escala del pedido

Output PROHIBIDO:
- "¡Qué bonito!", "flores preciosas", tuteo ("te", "tú")
- Tono entusiasta o celebratorio

---

**ORQ-3 — ocasion_especial → celebracion**

```
scenario: Usuario menciona una ocasión especial (boda, nacimiento, aniversario)
input:    "Son para la boda de mi hermana, se casa el sábado."
estado:   ocasion_especial  ($ocasion_detectada=Boda)
registro: celebracion
```

Output DEBE incluir:
- Reconocimiento de la ocasión con calidez genuina (no performativa)
- "qué ocasión tan especial", "flores que acompañen ese momento"
- Entusiasmo moderado — máximo 1 exclamación por turno

Output PROHIBIDO:
- "¡¡Qué emocionante!!", múltiples exclamaciones seguidas
- Respuesta genérica sin reconocer la boda
- Tono solemne o corporativo

---

**ORQ-4 — prisa → estandar + $es_urgente=true**

```
scenario: Usuario señala restricción temporal explícita
input:    "Necesito flores para hoy antes de las 6, es urgente."
estado:   prisa
registro: estandar
es_urgente: true
```

Output DEBE incluir:
- Acuse inmediato de la restricción: "Entendido, tenemos poco tiempo."
- Ir directo al slot más bloqueante (tipo de flores o presupuesto)
- Respuesta corta — sin preámbulo

Output PROHIBIDO:
- Ignorar la urgencia y hacer preguntas de exploración lenta
- Respuesta larga con contexto innecesario
- Modo exploración: "¡Claro! Cuéntame un poco más sobre la ocasión..."

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
| `solemne` | ECO del vocabulario del usuario · pausado · una pregunta por turno · "acompañar", "respeto", "discreto" | Exclamaciones (¡) · "claro que sí" · "perfecto" · "encantada" · preguntas frívolas sin contextualizar |
| `corporativo` | "usted" consistente · directo · eficiente · "con gusto le ayudo" · slot-filling ordenado (presupuesto → tipo → cantidad → fecha) | Tuteo ("tú", "te") · diminutivos · comentarios emocionales · entusiasmo · exclamaciones |
| `celebracion` | Calidez genuina · máximo 1 exclamación por turno · "especial", "que recuerde", "perfecto para ese día" · preguntas exploratorias | Múltiples exclamaciones seguidas · tono solemne o neutro · ignorar la ocasión · entusiasmo performativo |
| `estandar` | Registro base equilibrado — sin reglas específicas adicionales | — |

---

## Keywords de detección por ESTADO

> Usadas en Orquestador (detección inicial) y en Compra (vigilancia de cambio mid-conversación).
> Estado: **draft** — pendiente definir si Compra usa las mismas keywords o umbral relajado (PENDIENTE-4).

```
DUELO:          "fallecido", "entierro", "funeral", "pérdida", "falleció", "velatorio"
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

**Qué es Orquestador en relación al tono**: el único playbook que detecta el $estado inicial del usuario. Recibe el primer mensaje sin parámetros de tono y decide qué $registro aplicará todo el sistema. Es el punto de entrada — si falla aquí, el error se propaga a todos los demás playbooks.

**Comportamiento esperado tras el refactor**:
- Scan del primer mensaje → detecta $estado mediante keywords
- Fija $registro y $es_urgente
- Los pasa como output al playbook al que enruta

**Gate**: Jero ve 2-3 versiones del bloque de detección de $estado + keyword list. Elige versión.

---

### G3 — Refactor tipo APLICACIÓN: Compra (JOB 2)

**Qué es Compra en relación al tono**: el playbook más complejo. Recibe $registro ya fijado y lo aplica durante todo el flujo de compra multi-turno. Además vigila cambios de estado mid-conversación mediante keyword watch. Si detecta un cambio → actualiza $registro y lo propaga a Checkout.

**Diferencia clave respecto a Orquestador**: no detecta el estado inicial. Solo aplica y reacciona a cambios. El keyword watch de Compra es más ligero (trigger estático primero, semántico solo si hay hit).

**Comportamiento esperado tras el refactor**:
- Aplicar $registro recibido en cada turno (CONTINUIDAD)
- Usar vocabulario del usuario cuando aplica (ECO)
- Keyword watch por turno → si hit → re-evaluar → cambiar si corresponde
- Pasar $registro actualizado a Checkout

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
