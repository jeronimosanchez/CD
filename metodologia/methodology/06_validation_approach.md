# 06 — Validation Approach

## Propósito

Definir cómo se valida que el sistema hace lo que dice que hace, sin depender de equipos de anotadores que no se tienen.

## Las dos aproximaciones reconocidas

| Approach | Cómo funciona | Quién lo usa |
|---|---|---|
| **Expert-annotated validation** | Equipos de anotadores etiquetan datasets. Inter-rater agreement. Métricas estadísticas. | Google, Anthropic, OpenAI, DeepMind |
| **Closed-loop empirical validation** | El sistema actúa, mide outcomes, valida y aprende. Sin anotadores. | Sistemas autónomos de producción |

**Ambos son científicos. Ambos son empíricos**. La diferencia es de dónde sale la ground truth (humanos vs realidad operativa).

## Por qué elegir closed-loop

No es elección "inferior" — es la apropiada para el contexto:

- **No requiere equipos de anotación** (no los tienes ni los tendrás)
- **Escala desde día 1** sin infraestructura previa
- **Continuo en producción** vs puntual pre-release
- **Auto-validable** según opera

**Lo que pierdes en rigor académico, lo ganas en bucle de retroalimentación real**.

## Cómo funciona el closed-loop

```
Skill propone análisis + recomendación
   ↓
Recomendación aplicada al sistema
   ↓
TC re-corre automáticamente
   ↓
PASS → diagnóstico era útil (KB +1 acierto)
FAIL → diagnóstico era erróneo o incompleto (KB anti-precedente)
   ↓
Próxima iteración: el skill prioriza mecanismos con mayor efectividad
```

## Fuentes de validación (en orden de potencia)

1. **TCs del QA suite** (51 hoy en Petal): definen "qué debe pasar"
2. **Outcomes históricos** (~100+ FAILs en gh-pages): universo de bugs reales
3. **Git history** (commits que cerraron bugs): ground truth implícita de qué capa era el problema
4. **Logs de interacciones reales del cliente** (cuando aplique): la fuente más rica
5. **Señales implícitas en conversaciones reales**: abandono, escalación, repetición, sentiment negativo

## Señales implícitas extraíbles de logs conversacionales

| Categoría | Ejemplos | Capa afectada |
|---|---|---|
| Fallos de comprensión | Usuario repite, dice "no entiendes" | 1 / 2 |
| Fallos de flujo | Usuario vuelve atrás, bucle | 1 / 2 |
| Fallos de resolución | Escalada a humano, abandono | Multi-capa |
| Fallos de slot | Mismo dato 2 veces, slot vacío al final | 3 |
| Fallos de tono | Sentiment negativo creciente | 1 / UX |
| Fallos de integración | "Error técnico" repetido | 4 |
| Fallos de abandono | Sesión termina en flujo crítico | UX / 1 |
| Fallos de coherencia | Agente se contradice | 1 |

**Detección automatizable con NLP estándar + reglas**.

## Métricas derivadas para el cliente

| KPI | Cómo se calcula |
|---|---|
| % conversaciones exitosas | Sin señales de error / Total |
| % escaladas a humano | Escaladas / Total |
| Tiempo medio a resolución | Turnos hasta estado terminal |
| Tasa de abandono por flujo | Abandonos / Inicios por flujo |
| Tasa de frustración detectada | Conversaciones con sentiment_neg > umbral / Total |

**Vendibles a cliente enterprise como evidencia objetiva del valor**.

## Honestidad sobre la curva de aprendizaje

| Mes | Accuracy esperado del skill |
|---|---|
| Mes 1 (KB vacío) | ~50-60% |
| Mes 3 (KB con ~50 outcomes) | ~70-75% |
| Mes 6 (KB con ~200 outcomes) | ~80-85% |
| Mes 12 (KB maduro) | ~85-90% |

**El sistema arranca con accuracy modesta y mejora con uso. Trade-off explícito**.

## Anti-patrón

Prometer accuracy alta desde día 1. Es deshonesto y no se sostiene. **Acepta accuracy modesta inicial a cambio de mejora continua medible**.
