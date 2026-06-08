---
name: qap_ag_juez
version: 0.1
línea: QAP
scope: ag
estado: 🔴 No existe
descripción: >
  Evalúa la respuesta real del agente contra los criterios de la rúbrica.
  Por cada criterio produce un veredicto (sí / no / parcial) con evidencia textual.
  No agrega ni puntúa — eso es responsabilidad del scorer (infra).
kbs:
  - kb_ag_global
  - kb_proj_<proyecto>     # rúbricas y comportamiento esperado por TC
input:
  - respuesta real del agente (capturada en staging)
  - rúbrica del TC (criterios de evaluación)
output:
  - JSON estructurado por criterio:
      criterion_1: {verdict: "sí/no/parcial", evidence: "..."}
      criterion_2: {verdict: "sí/no/parcial", evidence: "..."}
modelo_recomendado: Claude API
razon_modelo: >
  Evaluación semántica que requiere comprensión de matices.
  Ejemplo crítico: una respuesta correcta sintácticamente que ignora
  una restricción temporal ("para esta tarde") es FAIL — un regex la daría PASS.
  Claude detecta estos fallos semánticos con mayor fiabilidad.
  Es el punto donde una evaluación incorrecta contamina todo el ciclo.
par_infra: scorer
descripcion_scorer: >
  Recibe el JSON de qap_ag_juez y agrega los veredictos:
  PASS (100%), PARTIAL (50-99%), FAIL (<50%).
  Determinista — no requiere LLM.
sistema_eval: eval_semantic
paso: 10
---

# qap_ag_juez

> ⚠️ SKILL EN CONSTRUCCIÓN — este archivo es el placeholder. El contenido real se escribe al construir la skill.

## Cuándo invocar

Paso 10 del ciclo autónomo — después de hypothesis-validator, antes de log+KB.

## Arquitectura juez + scorer

```
qap_ag_juez (skill)  →  veredicto por criterio (JSON)
      ↓
scorer (infra)       →  PASS / PARTIAL / FAIL + % numérico
```

El PARTIAL es clave: indica que la hipótesis iba en la dirección correcta
pero incompleta. Esa información alimenta el siguiente ciclo.

## Flujo

## Coste

## DoD
