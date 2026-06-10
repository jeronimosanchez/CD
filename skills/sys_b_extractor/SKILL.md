---
name: sys_b_extractor
version: 0.1
línea: Sistema B
scope: ag
estado: 🔴 No existe
descripción: >
  Lee los logs de salida de un ciclo de Sistema A y convierte cada resultado
  en un registro de experimento estructurado. Produce la materia prima que
  sys_b_classifier y sys_b_writer procesarán aguas abajo.
kbs:
  - kb_sys_core   # estructura del ciclo de Sistema A y formato de logs
input:
  - logs de un ciclo de Sistema A (JSON o texto estructurado):
      - hipótesis validadas + fixes aplicados + verificación OK
      - hipótesis rechazadas + motivo del rechazo
      - casos con regresión (fix funcionó pero rompió otro TC)
output:
  - array JSON de registros de experimento:
      contexto: "<qué flow/playbook/patrón fue afectado>"
      hipotesis: "<qué cambio se probó>"
      resultado: "✅ | ❌ | ⚠️"
      por_que: "<razón del resultado>"
      ciclo: "<fecha del ciclo>"
modelo_recomendado: claude-sonnet-4-6
razon_modelo: >
  Extracción estructurada sobre texto semiestructurado. No requiere juicio
  semántico complejo — es parseo con comprensión. Sonnet es suficiente y
  más económico para un volumen de N registros por ciclo.
sistema_eval: extraccion_estructurada
paso: 1  # primera etapa de Sistema B
cadencia: cada 7 ciclos de Sistema A (o inmediato si hay contradicción detectada)
---

# sys_b_extractor

> ⚠️ SKILL EN CONSTRUCCIÓN — este archivo es el placeholder. El contenido real se escribe al construir la skill.

## Cuándo invocar

Primera etapa de Sistema B. Se invoca al cierre de cada N ciclos de Sistema A
(valor por defecto: 7) o de forma inmediata cuando otro componente detecta
una posible contradicción con el KB existente.

## Flujo

```
logs Sistema A (JSON/texto)
       ↓
sys_b_extractor
       ↓
array de registros de experimento (JSON)
       ↓
sys_b_classifier  →  sys_b_writer
```

## Tipos de resultado

| Resultado | Qué representa | Por qué capturar |
|---|---|---|
| ✅ | Hipótesis validada, fix aplicado y verificado | Patrón positivo — candidato a KB |
| ❌ | Hipótesis rechazada por el juez | Anti-patrón — evita repetir el camino fallido |
| ⚠️ | Fix funcionó pero generó regresión | Caso límite — patrón condicionado, no escalar sin restricción |

## Coste

## DoD
