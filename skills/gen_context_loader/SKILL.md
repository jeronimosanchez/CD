---
name: gen_context_loader
version: 0.1
línea: GEN
scope: ag
estado: 🔴 No existe
descripción: >
  Lee el brief del diseñador y determina qué KBs son relevantes para el
  playbook a generar. Carga y estructura el contexto que recibirá el
  generador (gen_ag_generator). Es el puente entre el brief humano y
  el conocimiento acumulado del sistema.
kbs:
  - kb_plat_cx           # estructura técnica de playbooks en Dialogflow CX
  - kb_proj_<proyecto>   # tono, restricciones y patrones del proyecto
  - kb_ag_global         # principios universales de diseño conversacional
  - kb_sys_core          # anti-patrones confirmados (output de Sistema B)
input:
  - brief del diseñador:
      objetivo: "<qué debe hacer el playbook>"
      casos_cubiertos: "<lista de escenarios>"
      restricciones: "<lo que no debe hacer>"
      proyecto: "<nombre del proyecto, e.g. Petal>"
output:
  - contexto estructurado para gen_ag_generator:
      brief: <reproducido íntegro>
      kb_plataforma: <extracto relevante de kb_plat_cx>
      kb_proyecto: <extracto relevante de kb_proj_<proyecto>>
      kb_universal: <principios aplicables de kb_ag_global>
      anti_patrones: <entradas relevantes de kb_sys_core>
modelo_recomendado: claude-sonnet-4-6
razon_modelo: >
  El trabajo es leer el brief, razonar qué partes del KB son relevantes
  y componer un contexto coherente. Es selección y síntesis — no generación
  creativa. Sonnet es suficiente y el coste importa porque este skill corre
  antes de cada sesión de generación.
sistema_eval: seleccion_contexto
paso: 1  # primera etapa de GEN
trigger: bajo demanda (Jero inicia una sesión de generación)
---

# gen_context_loader

> ⚠️ SKILL EN CONSTRUCCIÓN — este archivo es el placeholder. El contenido real se escribe al construir la skill.

## Cuándo invocar

Primera etapa de GEN. Se invoca bajo demanda cuando Jero necesita generar
un playbook nuevo. No hay cron — GEN responde a necesidad puntual.

## Flujo

```
brief del diseñador
       ↓
gen_context_loader
  ├── identifica qué KBs aplican al brief
  ├── extrae las secciones relevantes (no el KB completo)
  └── estructura el contexto en formato listo para el generador
       ↓
contexto enriquecido → gen_ag_generator
```

## Criterio de selección de KB

No se carga el KB completo — solo las secciones relevantes al brief.
Ejemplo: si el brief es sobre un playbook de compra urgente, se cargan:
- De kb_plat_cx: estructura de playbook, parameters, state handlers
- De kb_proj_petal: tono, restricciones de inventario, flujo de compra
- De kb_ag_global: principios de urgencia y priorización
- De kb_sys_core: anti-patrones sobre urgencia ya confirmados

## Coste

## DoD
