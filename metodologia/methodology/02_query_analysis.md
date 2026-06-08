# 02 — Query Analysis methodology

## Propósito

Inventariar los queries que el agente va a recibir y clasificarlos por frecuencia y complejidad. Es la base para decidir la arquitectura.

## Por qué importa

La arquitectura de un agente conversacional moderno NO es monolítica:
- **Capa NLU clásica** (Intents/Flows): rápida, barata, determinística
- **Capa LLM** (Playbooks/Tasks): flexible, costosa, no determinística
- **Capa de datos** (Sheet/backend): contenido dinámico

Decidir QUÉ va en cada capa es la decisión arquitectónica más importante. Define coste, latencia, calidad y mantenibilidad del agente entero.

**El error común**: o construir 100% LLM (caro y no determinístico) o 100% NLU clásico (rígido). La arquitectura correcta es híbrida, con distribución calculada por caso de uso.

## Los 6 pasos del Query Analysis

```
1. INVENTARIO de queries esperados
   Fuentes: TCs existentes + intuición de negocio + conversaciones reales si hay
   Output: lista de 50-100 queries representativos

2. CLASIFICACIÓN por 2 ejes
   - Frecuencia: Alta / Media / Baja
   - Complejidad: Determinística / Variable / Creativa

3. ASIGNACIÓN NLU vs LLM según criterio (ver 03_layer_assignment.md)

4. ARQUITECTURA DERIVADA del mapping (ver 04_architecture_derivation.md)

5. VALIDACIÓN EMPÍRICA con uso real
   - % queries que caen en cada capa según lo esperado
   - Detección de patterns que forzaron LLM innecesariamente
   - Coste real por turno + latencia

6. RE-AJUSTE iterativo
   - Patrones repetidos en LLM → promocionar a NLU
   - Intents mal clasificados → degradar a LLM o reentrenar
   - El QAP detecta automáticamente cuándo un query mal asignado debería moverse entre capas
```

## Versión ligera (pragmática)

Versión rigurosa = 1.5 días.
Versión ligera = 2-3 horas:
- Listar los 20 queries más obvios
- Clasificar mentalmente con regla de pulgar
- 2-3 bullets por categoría
- Esbozo arquitectónico + dejar margen para ajustar

**La versión ligera es mejor que el 80% de proyectos en producción**. No exijas perfección.

## Conexión con QAP

El QAP retroalimenta el Query Analysis:
- Detecta patterns de bugs que sugieren queries mal asignados
- Propone re-asignación entre capas
- Acumula evidencia empírica para refinar la distribución

**Ciclo virtuoso**: Query Analysis pre-arquitectura → construcción → QAP detecta desajustes → Query Analysis se ajusta. El sistema converge a la arquitectura óptima.

## Honestidad sobre la práctica real

| Tipo profesional | Cómo lo hace en realidad |
|---|---|
| Consultor junior | Copia plantilla, adapta sobre la marcha |
| Senior pragmático | Mapeo mental de 30 min, no documentado |
| Enterprise con tiempo | Workshops 2-3 semanas, doc largo, resultado mixto |
| Producto in-house | Iterativo: prototipo + ajustar |
| Founder bootstrapped | Build first, ajustar después |

**El 80% de proyectos arranca sin mapping riguroso. Por eso fracasan o cuestan 3x lo previsto. La versión ligera es la frontera realista para diferenciarse**.

## Output

`templates/query_inventory.yaml` rellenado con los queries del proyecto.
