# 04 — Architecture Derivation methodology

## Propósito

Convertir el mapping de Query Analysis + Layer Assignment en una arquitectura técnica concreta deployable.

## Output esperado

Lista de:
- Intents necesarios (con training phrases iniciales)
- Entity Types necesarios
- Flows + Pages para state management
- Playbooks LLM (solo donde justifica)
- Tools / integraciones con backend

## Reglas de derivación

### Por cada query asignado a NLU clásica

→ Crea un **Intent** con:
- Display name claro
- 5-10 training phrases iniciales (a refinar con uso)
- Respuesta fija o lookup desde Sheet
- Si captura datos → Entity Types asociados

### Por cada query asignado a NLU + lookup

→ Crea un **Intent** + **Webhook/Tool** que consulta el backend.

### Por cada query asignado a LLM Task

→ Crea un **Playbook** especializado con:
- Goal claro y específico
- Instructions concisas
- Examples representativos
- Input/output parameter definitions

### Por cada flujo multi-turno con captura de datos

→ Crea un **Flow** + **Pages** con slot-filling determinístico.

### Rutas críticas para el negocio

→ NLU + reglas explícitas, NUNCA LLM puro.

Ejemplos: pagos, registro, escalación a humano, cancelaciones.

## Arquitectura resultante (ejemplo Petal 2)

```
INTENTS NLU clásicos:
- horarios, zonas_envio, devoluciones, plazo_entrega (G1 business)
- saludo, despedida, agradecimiento (G4)
- confirmacion_si, confirmacion_no
- email, telefono, direccion (entity extraction)
- cancelacion, hablar_con_humano (G6/G7)

FLOWS + PAGES:
- Slot-filling estructurado: email, teléfono, dirección
- State management de carrito

PLAYBOOKS LLM (Tasks pequeñas):
- Compra (refinamiento de producto)
- Refinamiento (filtros progresivos)
- Multi_Producto (carrito de varios items)
- Frustracion (manejo emocional)
- Manejo_Objeciones
```

## Principios de diseño arquitectónico

1. **Playbook delgado**: cada playbook hace UNA cosa bien. Si crece >5K tokens, descomponer.
2. **Slot-filling determinístico**: state management va en Pages, no en Playbooks.
3. **Tools antes que LLM**: si una decisión es determinística, ponla en una Tool.
4. **Composición sobre herencia**: agrupar playbooks pequeños vs un monolito.
5. **Orquestador delgado**: el orquestador solo enruta; no razona sobre el dominio.

## Anti-patrones arquitectónicos

| Anti-patrón | Síntoma | Cómo corregir |
|---|---|---|
| Mega-playbook | >10K tokens, hace todo | Descomponer en Tasks |
| Orquestador gordo | Tiene lógica de negocio | Mover lógica a playbook especializado |
| LLM para queries determinísticos | Coste alto, latencia alta | Mover a NLU |
| NLU para casos creativos | Fallback constante | Promover a LLM |
| Slots tracking en LLM | State management frágil | Mover a Flows + Pages |

## Validación de la arquitectura

Antes de construir, revisar:
- ¿Cada playbook tiene un único propósito claro?
- ¿Los slots están explícitos en input/output parameters?
- ¿Los tools cubren las integraciones necesarias?
- ¿Las rutas críticas para el negocio NO dependen 100% del LLM?
- ¿La arquitectura es testeable componente a componente?
