---
name: qap_sys_plan_generator
version: 0.1
línea: QAP
scope: sys
estado: 🔴 No existe
descripción: >
  Paso 1 del ciclo autónomo. Decide qué testear cada noche:
  lee el historial de ciclos anteriores, los FAILs sin resolver y la matriz de cobertura,
  y produce un coverage_plan.yaml con áreas a atacar, volumen de conversaciones
  e hipótesis a priorizar.
kbs:
  - kb_ag_global
  - kb_sys_core       # estructura del motor, métricas del ciclo, modelo de madurez
  - kb_proj_<proyecto> # qué componentes tiene el agente, qué FAILs hay abiertos
input:
  - log JSONL de ciclos anteriores         # ~/CD/proyectos/<proyecto>/_durable/log_hypotheses.jsonl
  - coverage_matrix.yaml                   # ~/CD/proyectos/<proyecto>/_durable/coverage_matrix.yaml — escrito por log+KB al final de cada ciclo
  - lista de TCs en FAIL                   # del runner QA del proyecto
output:
  - coverage_plan.yaml                     # qué areas atacar, volumen de conversaciones, hipótesis a priorizar
---

# qap_sys_plan_generator

> ⚠️ SKILL EN CONSTRUCCIÓN — este archivo es el placeholder. El contenido real se escribe al construir la skill.

## Cuándo invocar

Paso 1 del ciclo autónomo — siempre el primero.

## Flujo

## Coste

## DoD
