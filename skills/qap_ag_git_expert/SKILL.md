---
name: qap_ag_git_expert
version: 0.1
línea: QAP
scope: ag
estado: 🔴 No existe
descripción: >
  Experto del comité en el historial de cambios (Capa 3).
  Analiza el git del proyecto para responder: ¿ya se intentó este fix?
  ¿Se revirtió? ¿Cuándo y por qué falló? Evita que el ciclo repita
  direcciones ya descartadas.
kbs:
  - kb_ag_global
  - kb_ag_git               # metodología de navegación git (agnóstica)
  - kb_proj_<proyecto>      # ruta del repo del proyecto, estructura de archivos
input:
  - hipótesis actual (qué sección del playbook se propone cambiar)
  - ruta del archivo afectado
output:
  - informe: intentos previos sobre esa sección, resultados, direcciones descartadas
modelo_recomendado: Claude API
razon_modelo: >
  Requiere razonar sobre historial de decisiones — interpretar por qué
  un cambio previo fue revertido y si la hipótesis actual es sustancialmente
  distinta o repite el mismo error.
  La lógica de análisis es agnóstica (cualquier repo git).
  Lo específico del proyecto (ruta, estructura) viene de kb_proj.
gits_que_consulta:
  - git del proyecto (cx-automation-template para Petal) → historial de playbooks
  - NO el git del sistema (~/CD/) → ese es el motor, no el contenido
sistema_eval: eval_semantic
paso: 6
---

# qap_ag_git_expert

> ⚠️ SKILL EN CONSTRUCCIÓN — este archivo es el placeholder. El contenido real se escribe al construir la skill.

## Cuándo invocar

Paso 6 del ciclo autónomo — miembro del comité de expertos.

## Nota sobre repos

git-expert distingue dos gits:
- Git del proyecto (Petal: ~/cx-automation-template/) → busca aquí cambios en playbooks
- Git del sistema (~/CD/) → NO lo consulta para análisis de Petal

Para Farma: misma skill, kb_proj_farma proporciona la ruta del repo de Farma.

## Flujo

## Coste

## DoD
