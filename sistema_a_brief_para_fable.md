# Briefing completo — Sistema A (motor autónomo de optimización conversacional)
## Para revisión crítica por un modelo externo (Fable 5)

> **Versión 10-jun-2026 · Actualizado 12-jun-2026** (ver **§7.5** + preguntas 6-8): se ha empezado a
> construir y medir el prerequisito ADK, pero su **fidelidad SIGUE EN TEST, sin concluir** — los números
> de fidelidad de este documento son PROVISIONALES y están contaminados (fuga de andamiaje, ~22% INVALID).

> **Instrucción para el evaluador:** lee todo el documento — es largo a propósito,
> porque quiero una valoración con los detalles reales, no sobre una abstracción.
> Al final hay 5 preguntas concretas y una petición de cierre. No valides la idea
> por cortesía: busca activamente fallos lógicos, supuestos no comprobados, riesgos
> operativos, errores de coste/escala y alternativas mejores. Prefiero un "esto no
> funciona porque X" a un "buena idea". Si una sección no cierra, dilo.

---

## 0. Quién soy y qué construyo

Soy diseñador conversacional. Construyo un sistema de automatización continua (CD)
que **optimiza agentes conversacionales de forma autónoma, de noche, sin humano en
el bucle** salvo el gate final de despliegue.

El sistema se organiza en 4 líneas:
- **ACT** — despliegue de artefactos a la plataforma (operativo, CI/CD real con
  GitHub Actions + Workload Identity Federation). Son "las tuberías".
- **GEN** — generación de playbooks y artefactos (en definición).
- **QAP** — quality assurance y validación. **El Sistema A vive aquí.**
- **RES** — investigación y fuentes de conocimiento (en definición).

El caso de prueba es **Petal**, floristería online española con agente en
**Dialogflow CX**. Arquitectura **NLU + LLM**: Dialogflow enruta con intents/flows
(capa determinista) y Gemini genera las respuestas dentro de playbooks (capa LLM).
Tiene un backend de inventario en Cloud Run (FastAPI) que lee una Google Sheet.
El playbook de compra pesa ~11.4k tokens. Hoy hay una suite de ~29-49 TCs de QA
real corriendo en el pipeline.

**Petal es el primer punto de datos, no el producto.** El objetivo real es un
**método agnóstico** de optimización conversacional que sirva para cualquier
arquitectura, y que sea defendible como IP/portfolio en una entrevista técnica
(objetivo inmediato: conseguir un puesto, no comercializar).

---

## 1. Conceptos base (necesarios para entender el pipeline)

**Corpus fijo de TCs.** Un set de Test Cases predefinido que NO cambia entre runs.
Cubre el diseño conversacional end-to-end. Cada TC activa el sistema entero (no
testea una unidad aislada, sino una conversación completa con sus tool calls). La
reproducibilidad viene de que el corpus es fijo; la dificultad viene de que el
adversarial genera variaciones difíciles dentro de cada TC.

**Niveles N (eje de restricción de conocimiento).** Barremos la MISMA arquitectura
con distinto grado de KB inyectado:
- **N1** — sin KBs. Máxima libertad del LLM, máxima improvisación. Baseline crudo.
- **N2-N4** — se van añadiendo capas de KB progresivamente.
- **N5** — todos los KBs + anti-patrones confirmados. Mínima improvisación.
- Las **reglas de negocio** son base invariable desde N1 (no son un "nivel", son
  el suelo).
La hipótesis: medir el % de resolución por nivel dice cuánto aporta el conocimiento
acumulado frente a la capacidad cruda del modelo. Si N1 ya resuelve el 90%, el KB
aporta poco para ese cliente. Si N1 resuelve 40% y N5 resuelve 85%, el KB es el
diferencial.

**Dos aproximaciones (modos de ejecución):**
- **Aproximación 1 — `--validate-only`:** corre los pasos 2 a 11 SIN tocar CX ni
  desplegar nada. Solo mide métricas por nivel. Se pueden correr 30 runs × 5
  niveles = 150 ejecuciones en una noche, en local, a coste casi nulo. El output
  es puramente analítico: "con esta config, el agente resuelve X% del corpus".
- **Aproximación 2 — `--apply`:** ciclo completo de 12 pasos. Genera un fix real,
  lo valida, y lo deja en un paquete para gate humano antes de desplegar a CX.

La clave de diseño: **los pasos 2-11 son los mismos en ambos modos.** La diferencia
es solo si el paso 11 escribe-y-para (validate) o si continúa al gate y deploy
(apply). Eso permite extraer una "ventana automatizada sin intervención" reutilizable.

**Niveles de coste por modelo** (relevante para evaluar viabilidad):
| Componente | Modelo | Coste |
|---|---|---|
| Generación adversarial | Qwen 32B local (Ollama, M4) | $0 |
| Comité de expertos | Gemini Flash (free tier) | ~$0 |
| Fixer | Gemini Flash (free tier) | ~$0 |
| Plan / hipótesis / juez | Claude Opus 4.8 | coste real |
| Clustering | sentence-transformers local | $0 |
| Runner | Google ADK local | $0 |
| **Total estimado / noche** | | **~$0.45** |

(Una estimación anterior más conservadora, con validación en CX staging real para
top candidatos, daba ~$3.50/noche y ~$105/mes — dentro de $500/mes de créditos GCP,
out-of-pocket $0.)

**Prerequisito crítico, aún NO resuelto:** el ADK runner debe tener fidelidad
> 85% (tool_trajectory_avg_score) respecto al CX real. Si el simulador local no
predice el comportamiento de producción, todo el filtrado barato es ruido y el
sistema no sirve. Esto es lo primero que hay que construir y validar. **(ACTUALIZACIÓN 12-jun: se ha
EMPEZADO a construir y medir — ver §7.5. Sigue SIN concluir: el número actual está contaminado por fuga
de andamiaje, ~22% INVALID. La fidelidad de ADK aún NO está establecida.)**

---

## 2. El pipeline de 12 pasos (detalle completo)

| Paso | Nombre | Herramienta | Qué hace | Input | Output |
|---|---|---|---|---|---|
| 1 | plan_generator | Opus 4.8 | • Cruza historial de intentos con cobertura actual<br>• Identifica TCs sin cobertura reciente<br>• Prioriza por ROI para esta noche | log_hypotheses.jsonl · coverage_matrix.yaml · lista TCs | coverage_plan.yaml |
| 2 | adversarial | Qwen 32B local | • Genera conversaciones diseñadas para romper el agente<br>• Cubre los TCs del plan con variaciones difíciles<br>• Genera SOLO el input; el esperado sale de rúbricas (RES), no del adversarial | coverage_plan.yaml · kb_ag_global · kb_proj_petal | N conversaciones .json |
| 3 | ADK runner | google-adk | • Ejecuta cada conversación contra el agente local<br>• Captura transcript completo por turno<br>• Registra PASS/FAIL por TC | conversaciones .json · agente CX local | {tc_id, resultado, transcript} |
| 4 | metrics + split | script .py | • Calcula % PASS y % FAIL sobre el corpus<br>• Separa fails.json y passes.json<br>• Añade metadatos (nº run, nivel N, timestamp)<br>• Solo fails.json continúa; passes.json se archiva | array de resultados del runner | run_summary.json {total, pass_pct, fail_pct, run, nivel_N} · fails.json · passes.json |
| 5 | cluster_analyzer | embeddings local (sentence-transformers) | • Calcula embeddings semánticos de cada FAIL<br>• Agrupa FAILs por similitud (k-means)<br>• Estima ROI por cluster (nº de TCs que comparten causa) | fails.json | clusters + ROI estimado |
| 6a | playbook_expert | Gemini Flash | • Lee el playbook activo completo<br>• Identifica la instrucción causante del fallo<br>• Cita el texto exacto del playbook | transcript · playbook de definitions/ | informe: instrucción causante + cita |
| 6b | inventory_expert | Gemini Flash | • Analiza las tool calls del turno fallido<br>• Contrasta con la respuesta real del backend<br>• Determina si el fallo viene de la capa de datos | transcript · tool calls · Sheet | informe: análisis backend |
| 6c | git_expert | Sonnet | • Lee el historial de hipótesis para este TC<br>• Detecta si el fix ya se intentó<br>• Detecta si fue revertido y por qué | transcript · log_hypotheses.jsonl | informe: intentos previos + reversiones |
| 7 | hypothesis_generator | Opus 4.8 | • Sintetiza los 3 informes del comité<br>• Propone 3-5 hipótesis de fix distintas<br>• Rankea por % de éxito predicho<br>• Excluye hipótesis ya intentadas sin éxito | informes 6a+6b+6c · kb_sys_core · kb_ag_global | 3-5 hipótesis rankeadas con % predicho |
| 8 | hypothesis_fixer | Gemini Flash | • Toma la hipótesis #1<br>• Localiza la sección exacta del playbook<br>• Genera SOLO el delta, no el playbook entero | hipótesis #1 · playbook completo · kb_plat_cx | patch: sección + contenido nuevo |
| 9 | hypothesis_validator | google-adk | • Aplica el patch al agente local<br>• Ejecuta el corpus fijo completo contra él<br>• Registra PASS/FAIL de la hipótesis<br>• (Fase A = ADK local; Fase B opcional = staging CX real) | patch · corpus fijo · agente | PASS/FAIL por hipótesis + transcript |
| 10a | juez | Opus 4.8 | • Lee la rúbrica del TC (criterios de evaluación)<br>• Evalúa la respuesta real criterio a criterio<br>• Veredicto sí/no/parcial con evidencia textual | transcript validado · rúbrica del TC | JSON {criterion: {verdict, evidence}} |
| 10b | scorer | script .py | • Agrega los veredictos del juez<br>• Calcula % de criterios cumplidos<br>• Clasifica en PASS/PARTIAL/FAIL | JSON del juez | PASS/PARTIAL/FAIL + % |
| 11 | log | script .py | • Registra resultado e hipótesis en el historial<br>• Actualiza la matriz de cobertura<br>• En --validate-only: solo escribe, no despliega | resultado · hipótesis · patch · métricas | log_hypotheses.jsonl · coverage_matrix.yaml |
| 12 | gate humano → deploy | Jero + pipeline ACT | • Revisa el fix validado (impacto, riesgo)<br>• Aprueba o descarta<br>• Despliega a CX vía CI/CD (git push → GitHub Actions → WIF → CX) | paquete de cambio | deploy en producción |

**Decisiones de diseño ya cerradas (no las cuestiones salvo que veas un fallo):**
1. El adversarial genera solo el INPUT difícil; el output esperado sale de rúbricas
   verificadas (RES), no del propio adversarial — si no, validas contra una
   alucinación.
2. El comité (paso 6) **analiza** (aporta el *por qué*: dominio, contexto, riesgo);
   la validación **binaria** la hace ADK (aporta el *cuánto*: PASS% medible). Un
   comité votando binario sería opinión; ADK midiendo es evidencia.
3. GEN genera el fix (paso 8) ANTES de testar en ADK (paso 9) — no puedes testar
   algo que no existe. El orden es: comité analiza → GEN genera diff → ADK testa.

---

## 3. Ejemplo trabajado del paso 5 (cluster_analyzer) — para que veas la mecánica real

Cinco TCs fallan en una noche. Sus transcripts (resumidos):

```
TC-U01: "quiero rosas para esta tarde"  → agente ignoró "esta tarde"
TC-U03: "necesito flores para hoy, urgente" → no preguntó hora de entrega
TC-C06: "es para el cumpleaños de mi madre, que es hoy" → no verificó mismo día
TC-C08: "lo necesito antes de las 6, ¿es posible?" → no confirmó horario de corte
TC-I02: "quiero 20 girasoles" → Sheet devolvió stock=0 pero el agente dijo que había
```

**Paso A — embeddings.** `paraphrase-multilingual-MiniLM-L12-v2` convierte cada
transcript en un vector de 384 dimensiones.

**Paso B — matriz de similitud coseno** (compara todos con todos, N×N):

```
       U01    U03    C06    C08    I02
U01  [ 1.00   0.92   0.89   0.86   0.21 ]
U03  [ 0.92   1.00   0.91   0.85   0.20 ]
C06  [ 0.89   0.91   1.00   0.88   0.22 ]
C08  [ 0.86   0.85   0.88   1.00   0.23 ]
I02  [ 0.21   0.20   0.22   0.23   1.00 ]
```

similitud = (A·B)/(|A|·|B|) — mide el ángulo entre vectores. Los 4 primeros hablan
de restricción temporal; el quinto es un fallo de inventario, semánticamente lejano.

**Paso C — k-means agrupa en el espacio 384D:**

```
Cluster 0: U01 · U03 · C06 · C08   (tamaño 4, ROI 4)  → prioridad 1
Cluster 1: I02                      (tamaño 1, ROI 1)  → prioridad 2
```

**El punto crítico:** el cluster_analyzer agrupa por **similitud textual**, no
entiende el porqué. Solo en el paso 6 el playbook_expert lee el cluster 0 y dice
"estos 4 fallan porque el agente no detecta restricción temporal" — ese es el primer
momento en que el sistema entiende la causa. El clustering solo prioriza dónde mirar.

**Por qué esto importa para la evaluación:** con 5 FAILs, el comité podría leerlos
directo sin clustering. El paso 5 solo aporta valor con volumen (~20-30+ FAILs),
donde sin agrupar el comité gastaría tokens analizando síntomas repetidos de la
misma causa raíz. Esto lo hace un paso **condicional**, no fijo (ver sección 5).

---

## 4. La tesis central — por qué el sistema es agnóstico

El sistema es agnóstico **no porque sea idéntico para todos**, sino porque su
**esqueleto es constante y sus piezas son adaptables**.

- **Lo constante (el método):** el pipeline de 12 pasos. No cambia entre clientes.
- **Lo adaptable (los adapters enchufados):** cada paso se activa, desactiva o
  reconfigura según la arquitectura conversacional del cliente:

| Arquitectura | Qué cambia en el sistema |
|---|---|
| **NLU solo** (Dialogflow ES, LUIS, Watson legacy) | El comité no tiene capa LLM (no hay playbook_expert); el fixer opera sobre intents/entities, no playbooks. El juez evalúa clasificación, no generación. |
| **NLU + LLM** (CX + Gemini — Petal) | Dos capas: clasificación determinista + generación LLM. El fix puede tocar cualquiera de las dos. Es lo que se construye ahora. |
| **LLM solo** (Rasa Pro, agentes puros) | plan_generator tiene más libertad (no hay intents fijos que cubrir); cluster_analyzer gana valor (mayor varianza de respuesta); el comité se centra en prompt/política, no en routing. |

---

## 5. Los dos ejes ortogonales de adaptación

La adaptabilidad tiene dos ejes que NO deben mezclarse conceptualmente:

- **Eje arquitectura** (NLU / NLU+LLM / LLM): decide *qué adapters* se enchufan.
  Decisión de diseño, se toma una vez por cliente.
- **Eje nivel N** (N1→N5): decide *cuánto KB/restricción* se aplica dentro de una
  arquitectura. Barrido experimental, se corre cada noche.

Su intersección produce una **matriz paso × arquitectura × nivel** que sería el
manual de configuración del método por cliente. La hipótesis fuerte: cada paso,
analizado por separado y por nivel, permite **medir empíricamente si aporta valor**
para un cliente/arquitectura dados — convirtiendo la configuración en data-driven,
no en intuición.

Caso testigo: el cluster_analyzer aporta más en arquitectura LLM-pura **y** en
niveles bajos (N1 genera más varianza). Prescindible con <20 FAILs, necesario con
alta varianza. Es el primer paso donde ambos ejes se cruzan.

---

## 6. Bucle de conocimiento (Sistema B — cómo aprende el conjunto)

Lo que el sistema aprende no es solo "este fix funcionó", sino patrones del tipo
"para arquitectura NLU+LLM, los FAILs de urgencia se resuelven en la capa LLM el
80% de las veces, no en la capa NLU".

KB de 4 capas:
- **kb_ag_** — universal, transferible entre clientes (la IP de verdad).
- **kb_sys_** — sistema (anti-patrones confirmados).
- **kb_plat_** — plataforma (ej. cómo se estructura un playbook en CX).
- **kb_proj_** — cliente (ej. tono y reglas de Petal).

Un patrón **sube** de kb_proj_ a kb_ag_ cuando se confirma en 2+ proyectos.
El Sistema B (3 skills: extractor → classifier → writer) lee los registros de
experimento del log, los clasifica por capa aplicando 3 tests, y los escribe al KB
tras gate humano. Cadencia: cada 7 ciclos de Sistema A, o inmediato si detecta una
contradicción con conocimiento existente.

---

## 7. Estado real de construcción (qué existe, qué falta)

**Ya existe y funciona:**
- ACT completo: 12 recursos de CX desplegables vía CI/CD, WIF, idempotencia.
- Suite QA real (~29-49 TCs) corriendo en pipeline contra Default Environment,
  con reportes en GitHub Pages.
- 3 skills QA heredadas: qa-tc-analyzer (9 capas de causa raíz), qa-fix, qa-revert.
- KB de 4 capas con gobierno propio (_index, _politica, _mapa), cerrada.

**Falta construir (todo en estado 🔴):**
- El prerequisito ADK (instalar, conectar tool calls, **validar fidelidad >85%**,
  runner batch) — bloquea todo lo demás.
- Las ~10 skills del motor (plan_generator, adversarial, cluster_analyzer, los 3
  expertos del comité, hypothesis_generator, fixer, validator, juez).
- RES: las rúbricas del juez y el golden set (~50 conversaciones) — prerequisito
  del juez y el validator.
- Las skills de Sistema B y GEN (placeholders escritos, lógica sin construir).

---

## 7.5 ACTUALIZACIÓN 12-jun-2026 — El prerequisito ADK está EN TEST (sin concluir)

> Desde el 10-jun se ha EMPEZADO a construir y MEDIR el prerequisito crítico (fidelidad del runner ADK).
> **AÚN NO ESTÁ RESUELTO.** Los números de abajo son PROVISIONALES y el instrumento se está endureciendo
> ahora mismo. Se incluye porque cambia cómo leer la pregunta 1 (sigue siendo el cuello de botella).

**Qué se construyó.** Reconstrucción de Petal en ADK, arquitectura MULTI-AGENTE (orquestador → sub-agentes,
cada uno con SOLO su playbook) corriendo Qwen2.5-14B-q4 vía Ollama, webhook real, rúbrica regex. Mide
acuerdo vs CX sobre 51 TCs.

**Qué se midió (PROVISIONAL — no fiarse del número todavía):** acuerdo bruto ~82-88%, sesgo PESIMISTA
limpio (0 falsos negativos en ambas corridas — nunca traga un fallo real). PERO el número está
CONTAMINADO en las dos direcciones → todavía NO es fidelidad real.

**El hallazgo que invalida el número.** La reconstrucción FUGA andamiaje interno al output
(`$var`, `${PLAYBOOK:...}`, `PASO N`, `sourceMapping`, JSON de routing) — vocabulario que está
LITERALMENTE en los prompts: son directivas que el MOTOR de CX ejecuta y un LLM pelado lee como texto y
vomita. La rúbrica regex a veces PREMIA esa fuga (matchea keywords por casualidad → PASS falso) y a veces
penaliza respuestas limpias (FAIL falso). Medido: **~22% de turnos INVALID** (fuga). Con eso, el 82/88
mide en parte basura. → Es el **TERCER auto-envenenamiento del instrumento** (ground truth caduco →
truncación de contexto → fuga+regex). El patrón ya es tesis del método: **el harness se audita ANTES de
creerle un solo número.**

**Reproducibilidad (esto sí resuelto).** Misma config en dos hardware (Mac Metal vs Kaggle CUDA-P100):
82% vs 88%, discrepan en 9/51 TCs. Digest del modelo IDÉNTICO → drift DESCARTADO; la divergencia es
coma-flotante no asociativa entre backends = FÍSICA, no bug. Decisión: entorno canónico = Mac;
cross-hardware se abandona; baseline POR hardware (fingerprint con hardware+backend+versión).

**Plan de endurecimiento EN CURSO (no terminado, y en pausa hasta que cierre la pata de velocidad):**
1. Veredicto de 3 estados PASS/FAIL/**INVALID** (fuga/degeneración/tool-error/timeout → INVALID, NO FAIL;
   su % = salud del harness; >5-10% → run no vale). [parcial: ya midió el 22%]
2. Sanear las directivas de CX en los prompts (traducir `${PLAYBOOK}`/`PASO N` a lenguaje natural o
   constructos ADK reales — el LLM no puede fugar lo que no ve). [PENDIENTE — cambio significativo, re-baselinea]
3. Routing como tool-call (separado del canal de texto), no como texto.
4. Lexicón anti-fuga AUTOGENERADO desde el prompt compilado (pre-gate de validez, cero mantenimiento, agnóstico).
5. Auto-reproducibilidad ×2 en Mac (mismo run debe dar 100%) → baseline limpio v3 SOLO cuando 1-4 estén.

**Método de validación del INSTRUMENTO que emerge (agnóstico, es IP):** mutation testing (inyectar defectos
conocidos en copias de playbooks → tasa de detección = métrica reina; NO necesita TCs nuevos) · holdout
(casos que el optimizador nunca ve, anti-overfitting) · recall@k (el sesgo pesimista es el lado seguro) ·
tasa INVALID = salud del harness. Dos capas de veredicto: binario (contrato comparable con CX) + score
0-1 (orden interno de candidatos, nunca claim de calidad).

**Aprendizaje ADK capturado (kb_plat_adk, ADK-29):** distinguir lenguaje natural (pasa tal cual) de
directivas ejecutables del motor de CX (se traducen) — un LLM local no tiene el motor que las ejecuta.

**Estado honesto: la fidelidad de ADK NO está establecida.** No hay aún un número de fidelidad de fiar.
El cribador se ganará el puesto POR CLASE de TC vía gates mecánicos (INVALID<10%, falsas alarmas≤10-15%,
mutación≥80%, wall-clock < staging CX) con timebox; las clases que no pasen → CX directo (regla del embudo).

---

## 8. Preguntas concretas para el evaluador

1. **Fidelidad ADK (el cuello de botella):** todo el ahorro de coste depende de que
   el runner local prediga el CX real con >85% de fidelidad. ¿Es razonable ese
   umbral? ¿Qué pasa con fixes que pasan en local pero fallarían en producción —
   cómo los detecto sin desplegar 50 candidatos? ¿Hay un sesgo sistemático en
   simular localmente (ej. ADK no reproduce la latencia/timeout del webhook real,
   o usa una versión distinta del modelo Gemini) que invalide la métrica?

2. **El juez LLM como ground truth (circularidad):** el paso 10 usa Opus 4.8 para
   juzgar si la respuesta cumple la rúbrica. La respuesta la generó otro LLM (Gemini).
   ¿Es circular? ¿Cómo evito que el juez comparta puntos ciegos con el generador?
   ¿Qué método de calibración del juez recomendarías (golden set humano, doble juez
   con desacuerdo, rúbricas más atómicas)?

3. **La tesis de adaptabilidad (secciones 4-5):** ¿se sostiene que el mismo esqueleto
   de 12 pasos sirve para NLU-solo, NLU+LLM y LLM-solo, o estoy forzando una
   abstracción que en la práctica obligaría a rediseñar medio pipeline por
   arquitectura? ¿En qué paso concreto se rompe la analogía? ¿La matriz
   paso × arquitectura × nivel es un artefacto real y útil, o una racionalización
   a posteriori?

4. **Validez estadística del barrido por nivel:** 30 runs × 5 niveles por noche, y
   el adversarial genera inputs DISTINTOS en cada run. ¿El "% de resolución por
   nivel" es una métrica estadísticamente válida, o estoy mezclando varianza del
   agente con varianza del input? ¿Cómo separo ambas? ¿30 runs es suficiente
   potencia estadística para afirmar "N5 > N1"? ¿Debería fijar el input y variar
   solo el nivel para aislar el efecto?

5. **Sobreajuste al corpus fijo:** si optimizo siempre contra el mismo corpus de TCs,
   ¿no acabo sobreajustando el agente a esos TCs y degradando su comportamiento en
   conversaciones reales no cubiertas? ¿Cómo equilibro reproducibilidad (corpus fijo)
   con generalización? ¿Necesito un holdout de TCs que el optimizador nunca ve?

6. **Veredicto de 3 estados (ver §7.5):** ¿es correcto tratar la fuga de andamiaje como **INVALID**
   (muestra inválida, fuera del acuerdo y del ranking) en vez de FAIL? Razón: contarla como FAIL infla
   las falsas alarmas y entierra fixes buenos por ruido del harness. ¿El umbral de salud (INVALID <5-10%)
   es razonable? ¿Qué categorías de INVALID falta contemplar?

7. **Sanear vs reconstruir fiel (ver §7.5):** la fuga viene de inlinear directivas EJECUTABLES de CX
   (`${PLAYBOOK}`, `PASO N`) como texto. ¿Traducirlas a lenguaje natural basta, o el arreglo correcto es
   modelar el routing/estado como constructos REALES de ADK (transfer, state)? ¿Dónde está el punto de
   retorno decreciente entre "barato (sanear)" y "fiel (machinery ADK)"?

8. **Cross-hardware abandonado (ver §7.5):** mismo modelo/digest, distinto backend (Metal vs CUDA) →
   82% vs 88%, 9 TCs voltean. ¿Es correcto declarar la reproducibilidad cross-hardware imposible (física)
   y pinear UNA máquina canónica, o hay una normalización que se nos escapa? ¿Los 9 TCs que voltean son
   un "set de fragilidad" útil (candidatos a TCs discriminantes) o ruido a descartar?

**Cierre obligatorio:** por cada pregunta, dame tu veredicto + el fallo más grave
que veas + 1-2 alternativas concretas. Y al final, nombra los **3 supuestos del
sistema que, si resultan falsos, lo derrumban entero** — ordenados por probabilidad
de que sean falsos.
