---
name: qap_plat_cx_playbook_expert
version: 0.1
línea: QAP
scope: plat_cx
estado: 🔴 No existe
descripción: >
  Experto del comité en la capa de comportamiento (Capa 1).
  Lee el YAML del playbook involucrado e identifica qué instrucción, bloque o regla
  es responsable del fallo. Produce un informe con cita exacta para el hypothesis-generator.
  También crea y mantiene kb_proj_petal_playbook con heurísticas finas de auditoría.
kbs:
  - kb_ag_global
  - kb_plat_cx             # formato YAML de CX, estructura de playbooks, parámetros entrada/salida
  - kb_proj_petal_playbook # estado actual de cada playbook de Petal, comportamiento esperado
modelo: Gemini Flash (Google AI Studio free tier)
razon_modelo: >
  Análisis estructurado de YAML — reconocimiento de patrones en la propia plataforma de Gemini.
  No requiere razonamiento complejo: lee el archivo, identifica el bloque causante, cita la línea.
  Gemini conoce el formato CX nativamente. Coste $0.
input:
  - cluster de FAILs (del cluster-analyzer)
  - YAML del playbook involucrado
output:
  - informe de capa: estructura, params, reglas, tools — con cita exacta de la instrucción causante
---

# qap_plat_cx_playbook_expert

> ⚠️ SKILL EN CONSTRUCCIÓN — este archivo es el placeholder. El contenido real se escribe al construir la skill.

## Cuándo invocar

Paso 6 del ciclo autónomo — miembro del comité de expertos.

## Flujo

## Coste

## DoD
