---
name: qap_ag_hypothesis_generator
version: 0.1
línea: QAP
scope: ag
estado: 🔴 No existe
descripción: >
  Sintetiza los informes de N expertos en 3-5 hipótesis concretas y accionables,
  cada una con predicción de mejora en % y riesgos identificados.
  La skill no analiza el agente directamente — razona sobre los análisis ya hechos
  para producir hipótesis de CAUSA (abducción). Es diagnóstico (QAP), no generación de fixes:
  el fix concreto lo genera gen_plat_cx_hypothesis_fixer (GEN).
kbs:
  - kb_ag_global      # principios universales de diagnóstico y diseño
  - kb_sys_core       # estructura del motor: qué niveles puede tocar, restricciones, qué ya se intentó
  - kb_plat_<plataforma>  # qué soluciones son factibles en la plataforma del cliente (CX, Lex, etc.)
  - kb_proj_<proyecto>    # objetivo actual, estado del agente, métricas, contexto del cliente
modelo_recomendado: Claude API
alternativa_local: Qwen 2.5 32B Q3 (~13GB RAM) o DeepSeek R1 14B (~9GB)
razon_modelo: >
  Es el paso de razonamiento más exigente del ciclo. Requiere síntesis cross-domain:
  combinar evidencia de playbook + datos + historial en una sola hipótesis causal coherente.
  Claude conecta piezas de dominios distintos con mayor fiabilidad que modelos más pequeños.
  Gap crítico: si la hipótesis es errónea, el ciclo nocturno completo no produce valor.
  Estrategia: empezar con Qwen 32B local (Q3, ~13GB, $0) y medir tasa de acierto.
  Si > 70% hipótesis correctas → mantener local.
  Si no → escalar a Claude API (~$0.05-0.10/noche).
  DeepSeek R1 14B es alternativa ligera (~9GB) diseñada para chain-of-thought.
input:
  - informes del comité (N expertos — extensible, mínimo 3 en Fase 1)
  - log JSONL de hipótesis anteriores (para no repetir lo ya intentado)
output:
  - 3-5 hipótesis ordenadas por score
  - cada hipótesis incluye: descripción, mecanismo de fix, predicción de mejora %, riesgos
---

# qap_ag_hypothesis_generator

> ℹ️ Genera hipótesis de CAUSA (abducción) a partir de los informes del comité → es diagnóstico (QAP). El fix concreto lo genera `gen_plat_cx_hypothesis_fixer` (GEN).

> ⚠️ SKILL EN CONSTRUCCIÓN — este archivo es el placeholder. El contenido real se escribe al construir la skill.

## Decisiones de diseño clave

### Número de hipótesis: 3-5
- Menos de 3 → se pierde la selección competitiva que da valor al sistema
- Más de 5 → el ciclo overnight de prueba en ADK crece demasiado
- Con 3-5 hay variedad real y el ciclo cierra en tiempo razonable

### Comité extensible
El input son N informes, no exactamente 3. En Fase 1 son playbook-expert + inventory-expert + git-expert.
En Fase 2 se añaden test-expert, integration-expert, infra-expert sin cambiar esta skill.

### Por qué el tag es ag_ aunque carga kb_plat_cx
El tag describe la LÓGICA de la skill, no los KBs que carga.
La lógica de sintetizar informes y generar hipótesis es universal.
Lo específico de la plataforma (qué soluciones son factibles) viene del KB, no de la skill.
Si el cliente usa Amazon Lex, se carga kb_plat_lex y la skill genera hipótesis factibles en Lex.

### No repite hipótesis ya intentadas
Lee el log JSONL antes de generar. Si una hipótesis es similar a algo ya probado y descartado, no la regenera.

## Cuándo invocar

## Flujo

## Coste

## DoD
