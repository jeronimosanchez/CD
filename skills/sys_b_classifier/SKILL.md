---
name: sys_b_classifier
version: 0.1
línea: Sistema B
scope: ag
estado: 🔴 No existe
descripción: >
  Recibe un registro de experimento (de sys_b_extractor) y determina a qué
  nivel de KB pertenece (proyecto / servicio / universal / plataforma) y con
  qué nivel de confianza. Aplica 3 tests semánticos para la clasificación.
kbs:
  - kb_sys_core   # criterios de clasificación por nivel
  - kb_ag_global  # principios universales de diseño conversacional (referencia)
input:
  - registro de experimento (JSON del sys_b_extractor):
      contexto, hipotesis, resultado, por_que, ciclo
output:
  - registro enriquecido (JSON):
      ...campos originales...
      kb_nivel: "proyecto | servicio | universal | plataforma"
      confianza: 0.0-1.0
      razonamiento: "<por qué se asignó ese nivel>"
      tests_aplicados:
        abstraible_sin_dominio: true/false
        valido_fuera_de_cx: true/false
        patron_repetido: true/false
modelo_recomendado: claude-opus-4-8
razon_modelo: >
  Clasificación con juicio semántico: requiere razonar si un patrón es
  específico del dominio (florería) o generalizable, y si depende de
  comportamiento propio de Dialogflow CX o es agnóstico de plataforma.
  Opus minimiza clasificaciones erróneas que contaminarían el KB universal
  con entradas demasiado específicas.
sistema_eval: clasificacion_semantica
paso: 2  # segunda etapa de Sistema B
---

# sys_b_classifier

> ⚠️ SKILL EN CONSTRUCCIÓN — este archivo es el placeholder. El contenido real se escribe al construir la skill.

## Cuándo invocar

Segunda etapa de Sistema B, después de sys_b_extractor.
Procesa un registro a la vez — puede ejecutarse en paralelo para N registros.

## Los 3 tests

### Test 1 — ¿Abstraíble sin mencionar el dominio?
Pregunta: ¿Puedo enunciar este patrón sin mencionar flores, florería, ni ningún
elemento específico del cliente?
- Si no → `proyecto`
- Si sí → candidato a `servicio` o superior

### Test 2 — ¿Válido fuera de Dialogflow CX?
Pregunta: ¿Este aprendizaje seguiría siendo válido si el agente corriera en
Amazon Lex, Rasa, o cualquier otra plataforma conversacional?
- Si no → `plataforma`
- Si sí → no es de plataforma (puede ser servicio o universal)

### Test 3 — ¿Patrón repetido en 2+ ciclos?
Pregunta: ¿Este mismo patrón (o uno equivalente) apareció en al menos 2 ciclos
anteriores de Sistema A?
- Si sí → confianza alta (≥0.8), candidato a subir de nivel
- Si no → confianza baja (≤0.5), se queda en `proyecto` hasta confirmación

## Criterio especial (un solo cliente activo)

Cuando solo existe Petal (sin Farma ni otros), el test de "aparece en 2 clientes"
no aplica. Sustituto: si el LLM puede formular el patrón sin mencionar el dominio
del cliente → candidato a `universal` con confianza media (0.6).

## Niveles de KB

| Nivel | Quién se beneficia |
|---|---|
| proyecto | Solo quien mantiene este agente específico |
| servicio | Cualquiera que construya un bot del mismo vertical |
| universal | Cualquier agente conversacional, cualquier dominio |
| plataforma | Cualquiera que use Dialogflow CX (agnóstico de cliente) |

## Coste

## DoD
