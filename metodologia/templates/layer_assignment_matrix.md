# Layer Assignment Matrix — [Nombre proyecto]

**Fecha**:
**Versión**:
**Basado en**: `query_inventory.yaml` v[X]

---

## Distribución resultante

| Capa | % esperado | # queries |
|---|---|---|
| NLU clásico | | |
| NLU + lookup Sheet | | |
| LLM Task pequeña | | |
| LLM creative | | |

**Total**: 100%

---

## Asignación detallada por categoría

### Intents NLU clásicos (alta frecuencia, determinística)

| Intent | Training phrases iniciales | Respuesta tipo | Fuente datos |
|---|---|---|---|
| | | | |
| | | | |

### Intents NLU con lookup

| Intent | Training phrases | Backend endpoint | Lookup key |
|---|---|---|---|
| | | | |

### Entity Types necesarios

| Entity | Valores principales | Sinónimos |
|---|---|---|
| | | |

### Flows + Pages (state management)

| Flow | Pages internas | Slots capturados |
|---|---|---|
| | | |

### Playbooks LLM (Tasks pequeñas)

| Playbook | Goal | Inputs | Outputs | Tools usados |
|---|---|---|---|---|
| | | | | |

---

## Trade-offs aceptados

| Trade-off | Decisión | Razón |
|---|---|---|
| Coste vs flexibilidad | | |
| Determinismo vs naturalidad | | |
| Latencia vs profundidad | | |

---

## Anti-patrones evitados activamente

- [ ] No hay "mega-playbook" (>10K tokens)
- [ ] Orquestador es delgado (solo routing, no lógica de negocio)
- [ ] Rutas críticas (pagos, escalación) NO dependen 100% del LLM
- [ ] Slot-filling está en Pages, no en LLM
- [ ] Cada playbook tiene un único propósito claro

---

## Validación pre-construcción

- [ ] ¿La distribución NLU/LLM es razonable para el caso de uso?
- [ ] ¿Hay alguna query mal asignada según el criterio?
- [ ] ¿Las rutas críticas están protegidas?
- [ ] ¿El coste estimado mensual es viable?
- [ ] ¿La arquitectura es testeable componente a componente?

---

## Estimación de coste y latencia

| Tipo turno | Coste estimado | Latencia esperada | % de turnos |
|---|---|---|---|
| NLU | | | |
| NLU + lookup | | | |
| LLM Task | | | |
| LLM creative | | | |

**Coste promedio por conversación** (asumiendo X turnos):
**Latencia promedio por turno**:

---

## Próximos pasos

- [ ] Architecture derivation (cómo se traduce esto en artefactos)
- [ ] Setup CI/CD para el agente
- [ ] Primer deploy en entorno dev
- [ ] Suite QA inicial con TCs derivados del briefing
