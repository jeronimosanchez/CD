# 03 — Layer Assignment methodology

## Propósito

Decidir qué capa (NLU clásica, LLM, datos) maneja cada query del inventario.

## Criterio de asignación

| Frecuencia | Complejidad | Capa asignada | Razón |
|---|---|---|---|
| Alta | Determinística | **NLU** | Barato y rápido para lo más común |
| Alta | Variable | **NLU + lookup Sheet** | Patrón fijo, contenido dinámico |
| Media | Determinística | **NLU** | Sigue siendo rentable |
| Media | Variable | **LLM (Task pequeña)** | Justifica el coste |
| Baja | Cualquier complejidad | **LLM** | Volumen no permite especialización NLU |
| Cualquiera | Creativa | **LLM** | NLU no aplica |

## Distribuciones típicas por caso de uso

| Caso de uso | NLU / LLM óptimo | Razón |
|---|---|---|
| Customer service alto volumen | 80/20 | Mayoría queries simples repetitivos |
| Banca consultiva | 60/40 | Compliance exige determinismo, complejidad media |
| Salud (triaje) | 50/50 | Compliance + complejidad alta |
| Coaching/educativo | 30/70 | Contextual, sin patrón fijo |
| Agente creativo | 10/90 | Cada query único |
| Venta consultiva (Petal) | 60-70 / 30-40 | Mix FAQs + razonamiento contextual |

## Ejemplo aplicado (Petal)

| Query típico | Frecuencia | Complejidad | Capa asignada |
|---|---|---|---|
| "¿Cuál es vuestro horario?" | Alta | Determinística | NLU + Sheet lookup |
| "¿Hacéis envíos a Murcia?" | Alta | Determinística | NLU + Sheet lookup |
| "Hola/Adiós" | Alta | Determinística | NLU |
| "Sí/No/Vale" | Alta | Determinística | NLU |
| "Email: jero@gmail.com" | Media | Determinística | NLU + entity extraction |
| "Quiero rosas para un cumpleaños" | Alta | Variable | LLM (Task: Compra) |
| "Algo para una boda elegante pero no caro" | Media | Variable | LLM (Task: Refinamiento) |
| "Estoy muy cabreado, me cobraron mal" | Baja | Creativa | LLM (Task: Manejo Frustración) |

## Trade-offs explícitos

| Decisión | Pro | Contra |
|---|---|---|
| Más NLU | Cheaper, faster, determinístico | Rígido, requiere mantenimiento manual de intents |
| Más LLM | Flexible, sin training phrases | Caro, lento, no determinístico |
| Mucho lookup | Contenido siempre fresco | Depende de uptime del backend |

## Anti-patrones

1. **Todo a LLM** — caro innecesariamente, latencia alta
2. **Todo a NLU** — rígido, mucho mantenimiento de training phrases
3. **Asignar sin medir frecuencia** — termina dando a NLU queries que casi no ocurren (overhead)
4. **Asignar sin medir complejidad** — termina dando a LLM queries que tenían respuesta fija

## Output

`templates/layer_assignment_matrix.md` rellenado con la asignación del proyecto.
