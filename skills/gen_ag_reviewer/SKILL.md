---
name: gen_ag_reviewer
version: 0.1
línea: GEN
scope: ag
estado: 🔴 No existe
descripción: >
  Evalúa las 5 variantes generadas por gen_ag_generator contra los criterios
  del brief y los KBs cargados. Descarta las 2 peores y devuelve las 3 mejores
  con justificación. Actúa como revisor externo — distinto del generador para
  evitar sesgo hacia el output propio.
kbs: []  # no lee KBs directamente — evalúa contra el contexto ya cargado
input:
  - 5 variantes de playbook (YAML, de gen_ag_generator)
  - contexto enriquecido original (brief + KBs, de gen_context_loader)
output:
  - top 3 variantes seleccionadas con:
      ranking: 1 / 2 / 3
      puntuacion: 0-10
      fortalezas: ["..."]
      debilidades: ["..."]
      razonamiento: "<por qué esta variante supera a las descartadas>"
  - variantes descartadas con motivo
modelo_recomendado: claude-sonnet-4-6
razon_modelo: >
  Evaluación semántica de calidad: ¿esta variante cumple los criterios de
  diseño conversacional del brief? ¿Viola algún anti-patrón? ¿Es coherente
  con el tono del proyecto? Sonnet como revisor externo al generador (Qwen)
  evita el sesgo de auto-evaluación. No se necesita Opus — la evaluación
  es sobre criterios explícitos en el contexto, no juicio abierto.
sistema_eval: revision_playbook
paso: 3  # tercera etapa de GEN
---

# gen_ag_reviewer

> ⚠️ SKILL EN CONSTRUCCIÓN — este archivo es el placeholder. El contenido real se escribe al construir la skill.

## Cuándo invocar

Tercera etapa de GEN, después de gen_ag_generator.
Recibe las 5 variantes + el contexto original.

## Flujo

```
5 variantes (Qwen) + contexto original
       ↓
gen_ag_reviewer (Sonnet)
  ├── evalúa cada variante contra: brief · kb_plataforma · kb_proyecto · anti_patrones
  ├── detecta violaciones de anti-patrones → descalifica
  ├── puntúa coherencia, completitud, eficiencia
  └── selecciona top 3 con justificación
       ↓
3 variantes rankeadas → gen_ag_adversarial (stress test)
```

## Criterios de evaluación

| Criterio | Peso | Descripción |
|---|---|---|
| Coherencia con el brief | Alto | ¿Cubre los casos declarados? ¿Respeta las restricciones? |
| Corrección técnica CX | Alto | ¿Estructura válida para Dialogflow CX? |
| Tono y voz del proyecto | Medio | ¿Coherente con kb_proj? |
| Sin anti-patrones | Bloqueante | Si viola un anti-patrón confirmado → descalificada |
| Diversidad respecto a otras | Medio | Las 3 seleccionadas deben ser genuinamente distintas |

## Por qué Sonnet y no Qwen aquí

Qwen generó las variantes — evaluarlas con el mismo modelo introduce sesgo
(tiende a favorecer su propio estilo). Sonnet como revisor externo produce
evaluaciones más críticas e independientes.

## Coste

## DoD
