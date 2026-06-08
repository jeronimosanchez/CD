# Situaciones de cliente — mapa completo

> Documento de definición de producto.
> Define todos los escenarios reales que el producto puede encontrarse en el mercado.
> Base para diseñar la estructura de KBs y priorizar el backlog.
> Fecha: 2026-06-06

---

## Por qué existe este documento

El backlog del sistema se construyó de dentro hacia afuera (desde Petal).
Este mapa se construye de fuera hacia dentro (desde la realidad del mercado).
La intersección define qué KBs construir, en qué orden, y qué falta en el backlog.

---

## Los 20 escenarios

### A — Evaluación
> El cliente no sabe qué hacer. Necesita evidencia antes de comprometerse.

| ID | Situación | En qué consiste | Decisión clave |
|---|---|---|---|
| A1 | Auditoría / health check | Analizamos el sistema actual para determinar su estado real: calidad, cobertura, puntos débiles y potencial de mejora. | ¿Está bien o necesita intervención? |
| A2 | Benchmarking competitivo | Comparamos el sistema del cliente con el estándar del mercado o un competidor concreto para identificar brechas. | ¿Está por encima o por debajo del estándar? |
| A3 | Due diligence (M&A) | Auditamos los activos conversacionales como parte de un proceso de compraventa de empresa para valorar y documentar riesgos. | ¿Qué vale y qué riesgo tiene? |

### B — Optimización
> El cliente tiene un sistema que funciona pero quiere que rinda más.

| ID | Situación | En qué consiste | Decisión clave |
|---|---|---|---|
| B1 | Optimización reactiva | El cliente identifica conversaciones que fallan y nos las entrega. Diagnosticamos, decidimos si optimizar o cambiar, y ejecutamos. | ¿Optimizo el actual o cambio de sistema? |
| B2 | Optimización proactiva | El sistema funciona sin fallos evidentes pero el cliente quiere superar un techo de métricas (contención, CSAT, resolución). | ¿Dónde está el techo y cómo superarlo? |
| B3 | Escalado | El sistema actual funciona bien en su scope original pero necesita crecer: más canales, idiomas o casos de uso. | ¿Escalo el actual o rediseño? |
| B4 | Cumplimiento normativo | Una nueva regulación (GDPR, accesibilidad, sectorial) obliga a modificar el sistema en un plazo determinado. | ¿Qué hay que cambiar y cuánto impacta? |

### C — Cambio
> El cliente necesita un cambio significativo en lo que ya tiene.

| ID | Situación | En qué consiste | Decisión clave |
|---|---|---|---|
| C1 | Migración planificada | El cliente sabe que su sistema actual ha llegado a su límite. Quiere uno nuevo que cubra lo actual y amplíe funcionalidades. | ¿Qué plataforma y cómo migrar? |
| C2 | Modernización de legacy | El cliente tiene un sistema de primera generación (IVR, Dialogflow ES, LUIS, Watson) que necesita modernizarse manteniendo el caso de uso. | ¿Qué conservo y qué rediseño? |
| C3 | Expansión de caso de uso | El sistema funciona bien en un área y el cliente quiere extenderlo a otra (ej: atención → ventas). | ¿Extiendo el actual o creo uno paralelo? |
| C4 | Consolidación multi-plataforma | El cliente tiene múltiples sistemas en distintas plataformas (Google, Amazon, on-premise) y quiere unificarlos. | ¿Qué plataforma destino y cómo unificar? |

### D — Creación
> El cliente no tiene sistema o quiere uno completamente nuevo.

| ID | Situación | En qué consiste | Decisión clave |
|---|---|---|---|
| D1 | Greenfield con plataforma libre | El cliente quiere un sistema nuevo desde cero y nos da libertad para recomendar la plataforma e implantarla. | ¿Qué plataforma y qué arquitectura? |
| D2 | Greenfield con plataforma impuesta | El cliente quiere un sistema nuevo pero ya tiene contrato con un proveedor. Diseñamos e implantamos dentro de su stack. | ¿Cómo diseñar dentro de sus constraints? |
| D3 | POC / prototipo | El cliente quiere validar si un caso de uso tiene sentido antes de comprometer una inversión completa. | ¿Vale la pena invertir en el caso completo? |

### E — Gestión
> El cliente quiere que operemos o transfiramos lo que ya existe.

| ID | Situación | En qué consiste | Decisión clave |
|---|---|---|---|
| E1 | Soporte post-implantación | Sistema que nosotros construimos. El cliente quiere que lo mantengamos, corrijamos errores y hagamos mejoras menores. | ¿Qué SLA y cómo lo operamos? |
| E2 | Managed service | El cliente nos delega la operación completa: corremos el sistema, lo mejoramos de forma continua y reportamos resultados. | ¿Qué modelo de servicio y precio? |
| E3 | Transferencia de conocimiento | El cliente quiere internalizar la gestión de su propio sistema. Nosotros formamos y capacitamos a su equipo. | ¿Qué capacidades necesita el equipo cliente? |

### F — Excepción
> Situaciones con características únicas que requieren un enfoque diferente al estándar.

| ID | Situación | En qué consiste | Decisión clave |
|---|---|---|---|
| F1 | Emergencia / incidente crítico | El sistema está caído o con fallo grave e impacto de negocio inmediato. Foco en triaje y restauración en horas, no optimización. | ¿Qué falla y cómo lo resuelvo en horas? |
| F2 | QA como servicio externo | El cliente construye internamente pero nos contrata para validar y certificar la calidad antes de cada release. | ¿Qué certifica y qué no? |
| F3 | Estrategia de datos conversacionales | El cliente tiene años de logs de conversaciones sin explotar. Queremos extraer patrones y tomar decisiones con datos reales. | ¿Qué patrones hay y qué decisiones informan? |

---

## Dimensión transversal — modo de entrega

Aplica a cualquier situación. Define qué entregamos y qué hace el cliente.

| Modo | Qué hacemos |
|---|---|
| Advisory only | Analizamos y recomendamos — el cliente ejecuta |
| Co-build | Construimos junto al equipo del cliente |
| Full delivery | Construimos todo nosotros |

---

## Próximo paso

Mapear cada situación contra la estructura de KBs y skills para:
1. Detectar gaps en el backlog
2. Priorizar qué construir primero
3. Definir qué KB carga cada skill en cada situación
