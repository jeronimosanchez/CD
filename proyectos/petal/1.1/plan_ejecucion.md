# Plan de ejecución — Petal 1.1

**Fecha:** 2026-06-04
**Estado:** listo para lanzar multiflow

---

## Orden de ejecución (11 pasos, fijo)

```
1.  Staging
2.  Task 0 — Inventario Sheet
3.  Task 1 — ConsultaInventario_Task
4.  Task 2 — GestionFiltros_Task
5.  Task 3 — VerificacionEntrega_Task
6.  Task 4 — DeteccionContexto_Task
7.  NLU — reemplaza Orquestador
8.  Compra — refactor (solo casos difusos)
9.  TCs nuevos (~15)
10. Rich cards
11. Suggestion chips
```

## Paralelismo posible

```
Paso 1 (Staging) — secuencial, bloqueante
    ↓
Paso 2 (Inventario) — secuencial, prerrequisito de Tasks
    ↓
Pasos 3+4+5+6 — PARALELO (Tasks independientes)
    ↓
Paso 7 (NLU) — secuencial, necesita Tasks
    ↓
Paso 8 (Compra refactor) — secuencial
    ↓
Paso 9 (TCs) — puede ir en paralelo con 8
    ↓
Pasos 10+11 — PARALELO (Rich cards + Chips)
```

## Arquitectura clave

- **Orquestador eliminado** → sustituido por NLU
- **Compra** → solo casos difusos (~6k tokens, antes 11.4k)
- **6 modalidades** → contexto (particular/empresa/funeral) × modo (concreto/difuso)
- **NLU router** → fast path si concreto, Compra si difuso

## Fuera de 1.1

- Zona entrega centro/periferia por CP
- Seguridad backend (petal-sheet-api público)
- Auto-detección ciudad
- NLU completo para intents de alto volumen (con datos reales)

## Prerequisitos antes de lanzar el multiflow

- [ ] Revisar Sheet inventario (columnas actuales)
- [ ] Confirmar que Staging no tiene dependencias externas pendientes
- [ ] Brief 1.1 revisado y aprobado
