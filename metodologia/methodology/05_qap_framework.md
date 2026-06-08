# 05 — QAP Framework

## Propósito

Framework de análisis y validación de calidad de agentes conversacionales construidos. Es la metodología que usa el skill `qa-tc-analyzer`.

## Arquitectura del sistema QAP — 4 capas

```
01. Capa conceptual — 9 dimensiones de análisis
02. Capa de validación — empírica con base de datos
03. Capa de gestión del KB — anti-drift
04. Capa de calidad del output — 5 estrategias transversales
```

---

## Capa 01 — Las 9 dimensiones de análisis

Cada bug se evalúa contra 9 preguntas estructurales:

| # | Capa | Pregunta estructural |
|---|---|---|
| 1 | Comportamiento | ¿La instrucción al agente es la causa? |
| 2 | Routing | ¿La conversación va al módulo equivocado? |
| 3 | Parámetros / Slots | ¿Se pierde información entre componentes? |
| 4 | Integración | ¿Falla una llamada a tool o backend? |
| 5 | Datos | ¿La fuente de datos es coherente? |
| 6 | Infraestructura | ¿El deploy y la versión son los correctos? |
| 7 | Modelo / LLM | ¿Gemini se comporta de forma no determinista? |
| 8 | Histórico | ¿Hay regresión previa o cambio reciente? |
| 9 | Test | ¿El test está bien calibrado? |

Cada capa recibe marca + cita de fuente obligatoria:
- 🔴 verificada · causa del bug + fuente
- 🟢 verificada · descartada con evidencia
- 🟡 supuesta · no comprobable con fuente directa
- ⚪ N/A · no aplica al tipo de bug

---

## Capa 02 — Validación empírica con KB

**Closed-loop empirical validation**: el sistema se valida con sus propios outcomes.

### Bucle

```
1. Bug detectado (QA suite marca FAIL)
2. Skill analiza + recomienda (9 capas + N soluciones)
3. Fix aplicado al sistema (rama efímera, deploy automático)
4. Outcome medido (TC pasa / no pasa)
5. KB refuerza (PASS → +1 acierto · FAIL → anti-precedente)
6. Iteración si no funcionó (Sol#2, Sol#3 también alimentan KB)
```

### Qué guarda el KB

| Archivo | Contenido |
|---|---|
| `system_knowledge.md` | Mapa del agente analizado |
| `mechanisms_library.yaml` | Mecanismos de fix verificados con efectividad |
| `patterns_catalog.yaml` | Patrones de bug recurrentes cross-TC |
| `gemini_behaviors.md` | Comportamientos verificados del LLM |
| `decisions.log` | Audit trail completo de análisis |
| `confidence_track.yaml` | Métricas agregadas de precisión por capa |

### Honestidad

No hay validación académica clásica (sin anotadores expertos). La validación es operativa, continua y automatizable.

---

## Capa 03 — Gestión anti-drift del KB

El KB no es estático: el sistema analizado evoluciona constantemente. Sin gestión activa, el KB se vuelve obsoleto silenciosamente (concept drift).

### 4 técnicas combinadas

| # | Técnica | Qué hace |
|---|---|---|
| 1 | Snapshotting | Cada entrada al KB lleva el hash del sistema cuando se registró |
| 2 | Decay temporal | Outcomes recientes pesan más; el KB "olvida" lo viejo |
| 3 | Detección de drift | Cambios en componentes invalidan outcomes asociados |
| 4 | Re-validación periódica | Cron mensual re-corre outcomes antiguos contra sistema actual |

---

## Capa 04 — 5 estrategias transversales de calidad

| # | Estrategia | Qué hace | Retos que cubre |
|---|---|---|---|
| E1 | Multi-modelo cross-validation (solo en marcas 🔴) | Varios LLMs confirman las marcas críticas | G1, G2, G7 |
| E2 | Verificación empírica > teórica | Citas re-verificadas, recomendaciones probadas, KB con outcomes | G3, G8, G10 |
| E3 | Inputs estructurados (activación futura) | Retrieval semántico cuando el volumen supere context window | G3, G9 |
| E4 | Rediseño del framework | Multi-capa, capa UX, score continuo | G4, G5, G11 |
| E5 | Inversión en coste de validación | Más runs cuando hay varianza alta | G6, G7 |

---

## Los 11 retos del framework

| # | Reto | Estrategia que lo aborda |
|---|---|---|
| G1 | Verificador = analizador | E1 |
| G2 | Confirmation bias | E1 |
| G3 | Alucinación en citas | E2, E3 |
| G4 | Granularidad mono-capa vs multi-capa | E4 |
| G5 | Cobertura incompleta del framework | E4 |
| G6 | Sample size pequeño | E5 |
| G7 | No determinismo del análisis | E1, E5 |
| G8 | No validación empírica pre-recomendación | E2 |
| G9 | Lost in the middle / context window | E3 |
| G10 | KB poisoning | E2 |
| G11 | Regla binaria fuerza mal categorizaciones | E4 |

**Techo realista colectivo: ~85-90% de reducción de riesgo. Production-grade**.
