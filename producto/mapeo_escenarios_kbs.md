# Mapeo: 20 Escenarios × KBs del Sistema
> Fecha: 2026-06-06 | Generado por agente de análisis

## Tabla de mapeo
Leyenda: ✅ necesario | ⚡ opcional/parcial | — no aplica

| Escenario | kb_global | kb_diseño | kb_plataformas | kb_cx | kb_diagnostico | kb_optimizacion | kb_migracion | kb_requisitos | kb_cliente_N |
|---|---|---|---|---|---|---|---|---|---|
| A1 Auditoría | ✅ | ⚡ | ✅ | ⚡ | ✅ | ⚡ | — | — | ✅ |
| A2 Benchmarking | ✅ | ✅ | ✅ | ⚡ | ⚡ | — | — | — | — |
| A3 Due diligence | ✅ | ✅ | ✅ | ⚡ | ✅ | — | ⚡ | — | ✅ |
| B1 Opt. reactiva | ✅ | ⚡ | — | ✅ | ✅ | ✅ | — | — | ✅ |
| B2 Opt. proactiva | ✅ | ✅ | — | ✅ | ✅ | ✅ | — | — | ✅ |
| B3 Escalado | ✅ | ✅ | ✅ | ✅ | ⚡ | ✅ | — | — | ✅ |
| B4 Cumplimiento | ✅ | ⚡ | ✅ | ✅ | ✅ | ⚡ | — | ⚡ | ✅ |
| C1 Migración | ✅ | ✅ | ✅ | ✅ | ✅ | ⚡ | ✅ | ⚡ | ✅ |
| C2 Legacy | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ⚡ | ✅ |
| C3 Expansión | ✅ | ✅ | ⚡ | ✅ | ⚡ | ✅ | — | ✅ | ✅ |
| C4 Consolidación | ✅ | ✅ | ✅ | ✅ | ✅ | ⚡ | ✅ | — | ✅ |
| D1 Greenfield libre | ✅ | ✅ | ✅ | ⚡ | — | — | — | ✅ | ✅ |
| D2 Greenfield impuesto | ✅ | ✅ | ⚡ | ✅ | — | — | — | ✅ | ✅ |
| D3 POC | ✅ | ✅ | ✅ | ⚡ | — | — | — | ✅ | ⚡ |
| E1 Soporte | ✅ | ⚡ | — | ✅ | ✅ | ✅ | — | — | ✅ |
| E2 Managed service | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | — | ⚡ | ✅ |
| E3 Transferencia | ✅ | ✅ | ✅ | ✅ | ⚡ | ⚡ | ⚡ | ⚡ | ✅ |
| F1 Emergencia | ✅ | — | — | ✅ | ✅ | ✅ | — | — | ✅ |
| F2 QA externo | ✅ | ⚡ | ✅ | ⚡ | ✅ | ✅ | — | — | ✅ |
| F3 Datos | ✅ | ✅ | ✅ | ⚡ | ✅ | ⚡ | — | ⚡ | ⚡ |

## Frecuencia por KB (prioridad)

| Prioridad | KB | Puntuación | Cobertura |
|---|---|---|---|
| P1 | kb_global | 20.0 | 20/20 (100%) |
| P2 | kb_cliente_N | 18.5 | 20/20 (100%) |
| P3 | kb_cx | 16.0 | 20/20 (100%) |
| P4 | kb_diseño | 15.0 | 17/20 (85%) |
| P5 | kb_diagnostico | 14.5 | 17/20 (85%) |
| P6 | kb_plataformas | 12.5 | 15/20 (75%) |
| P7 | kb_optimizacion | 12.0 | 15/20 (75%) |
| P8 | kb_metricas ⭐ NUEVO | ~11.0 | 8/20 (40%) |
| P9 | kb_gobierno ⭐ NUEVO | ~8.0 | 6/20 (30%) |
| P12 | kb_transferencia ⭐ NUEVO | ~4.0 | 5/20 (25%) |

## 3 KBs nuevos detectados (no estaban en la estructura)

| KB | Por qué | Escenarios |
|---|---|---|
| kb_metricas | KPIs conversacionales, benchmarks, metodología de baselining | A1, A2, A3, B1, B2, E2, F2, F3 |
| kb_gobierno | Cumplimiento normativo, GDPR, AI Act, gobernanza de datos | B4, A3, E2, F3, C4 |
| kb_transferencia | Metodología de handoff y formación al equipo cliente | E3, C1, C2, D1, D2 |

## Secuencia mínima viable (MVP de KBs)

```
Paso 1  kb_diseño + kb_diagnostico  →  desbloquean 12 escenarios
Paso 2  kb_plataformas + kb_optimizacion  →  5 escenarios adicionales
Paso 3  kb_metricas  →  hace evaluables todos los A-series
Paso 5  kb_gobierno + kb_transferencia  →  cierran regulatorios y entrega
```

## Estado actual del sistema

Con solo kb_global + kb_cx + kb_petal:
- INOPERABLE (faltan KBs críticos): 10 escenarios
- GAP CRÍTICO (calidad degradada): 10 escenarios
- Completamente cubiertos: 0 escenarios

El sistema actual solo es operativo para mantenimiento puntual de Petal.
