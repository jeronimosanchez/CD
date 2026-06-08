---
name: qap_proj_petal_inventory_expert
version: 0.1
línea: QAP
scope: proj
estado: 🔴 No existe
descripción: >
  Experto del comité en la capa de inventario/catálogo (Capa 2).
  Analiza los FAILs desde la perspectiva de la Sheet de Petal y del petal-sheet-api.
  Identifica si el fallo viene de un producto inexistente, una tool call fallida,
  o una interpretación incorrecta de la respuesta del inventario.
kbs:
  - kb_ag_global
  - kb_proj_petal           # estructura de la Sheet, categorías de productos, formato petal-sheet-api
input:
  - cluster de FAILs (del cluster-analyzer)
  - logs de tool calls del petal-sheet-api en las conversaciones fallidas
output:
  - informe de capa: disponibilidad, tool calls, interpretación — con cita exacta del fallo
modelo_recomendado: Gemini Flash (Google AI Studio free tier)
razon_modelo: >
  Análisis de datos estructurados (respuestas JSON del petal-sheet-api).
  Reconocimiento de patrones en las tool calls — no requiere razonamiento complejo.
  Coste $0 (free tier).
  Para Farma: crear qap_proj_farma_inventory_expert con su propio KB.
sistema_eval: eval_semantic
paso: 6
---

# qap_proj_petal_inventory_expert

> ⚠️ SKILL EN CONSTRUCCIÓN — este archivo es el placeholder. El contenido real se escribe al construir la skill.

## Cuándo invocar

Paso 6 del ciclo autónomo — miembro del comité de expertos.

## Flujo

## Coste

## DoD
