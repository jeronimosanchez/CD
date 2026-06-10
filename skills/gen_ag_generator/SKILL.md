---
name: gen_ag_generator
version: 0.1
línea: GEN
scope: ag
estado: 🔴 No existe
descripción: >
  Motor de generación de playbooks. Recibe el contexto enriquecido de
  gen_context_loader y produce 5 variantes completas del playbook solicitado.
  Cada variante es un candidato válido — estructuralmente correcto y coherente
  con los KBs cargados. La diversidad entre variantes es deliberada: cubren
  enfoques distintos para el mismo objetivo.
kbs: []  # no lee KBs directamente — recibe el contexto ya preparado por gen_context_loader
input:
  - contexto enriquecido (output de gen_context_loader):
      brief, kb_plataforma, kb_proyecto, kb_universal, anti_patrones
output:
  - array de 5 variantes de playbook (YAML):
      cada variante incluye:
        - nombre
        - description
        - instruction.steps (el cuerpo completo)
        - enfoque: <qué estrategia de diseño usa esta variante>
modelo_recomendado: Qwen 32B (Ollama local)
razon_modelo: >
  Generación creativa de texto estructurado en español. Qwen 32B supera
  a Gemma en esta tarea y corre a $0 en el M4 local. La diversidad de
  variantes requiere capacidad generativa real — un modelo pequeño produce
  5 variantes demasiado similares entre sí.
  Ollama endpoint: http://localhost:11434
sistema_eval: generacion_playbook
paso: 2  # segunda etapa de GEN
n_variantes: 5
---

# gen_ag_generator

> ⚠️ SKILL EN CONSTRUCCIÓN — este archivo es el placeholder. El contenido real se escribe al construir la skill.

## Cuándo invocar

Segunda etapa de GEN, después de gen_context_loader.
Recibe el contexto ya preparado — no lee KBs directamente.

## Flujo

```
contexto enriquecido (de gen_context_loader)
       ↓
gen_ag_generator (Qwen 32B local)
  ├── variante 1: enfoque directo / mínimo
  ├── variante 2: enfoque conversacional / empático
  ├── variante 3: enfoque robusto / manejo de excepciones
  ├── variante 4: enfoque eficiente / pasos reducidos
  └── variante 5: enfoque híbrido / balance
       ↓
5 playbooks candidatos (YAML) → gen_ag_reviewer
```

## Por qué 5 variantes con enfoques distintos

Una sola variante limita el espacio de soluciones. 5 variantes con
enfoques deliberadamente distintos maximizan las probabilidades de que
al menos una cubra el caso mejor que el diseñador hubiera anticipado.
gen_ag_reviewer filtra las 2 peores — pero las 3 que pasan a QAP deben
ser genuinamente distintas, no paráfrasis entre sí.

## Restricciones del output

- Cada variante debe ser structuralmente válida para Dialogflow CX
- No puede violar ningún anti-patrón listado en el contexto
- El YAML de cada variante debe ser desplegable por ACT sin modificaciones

## Coste

$0 — Qwen local.

## DoD
