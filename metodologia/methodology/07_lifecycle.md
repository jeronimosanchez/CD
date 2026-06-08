# 07 — Lifecycle methodology

## Propósito

Definir el ciclo de vida completo de un agente CD, desde el arranque hasta la operación continua.

## Las 5 fases

```
1. DESIGN     → briefing + query analysis + layer assignment
2. BUILD      → ACT + GEN ejecutan el diseño
3. VALIDATE   → QAP analiza, detecta bugs, propone fixes
4. ITERATE    → outcomes alimentan KB, ajustan diseño
5. STRATEGIC  → RES propone redesigns arquitectónicos periódicos
```

---

## Fase 1 — DESIGN

**Tiempo**: 2-3 días (versión ligera) hasta 2-3 semanas (versión rigurosa enterprise)

**Inputs**: briefing del cliente
**Outputs**:
- `briefing_form.md` rellenado
- `query_inventory.yaml` rellenado
- `layer_assignment_matrix.md` rellenado
- Esbozo arquitectónico

**Templates relevantes**: `01_briefing_template.md`, `02_query_analysis.md`, `03_layer_assignment.md`, `04_architecture_derivation.md`

---

## Fase 2 — BUILD

**Tiempo**: 2-6 semanas según complejidad

**Líneas activas**:
- **ACT**: deploy automático de playbooks a la plataforma
- **GEN**: generación de playbooks, examples, intents

**Outputs**:
- Agente desplegado en entorno dev/test
- Suite QA inicial con TCs derivados del briefing
- CI/CD operativo

---

## Fase 3 — VALIDATE

**Tiempo**: continuo desde el momento del primer deploy

**Línea activa**:
- **QAP**: skill `qa-tc-analyzer` + 4 capas del framework

**Outputs**:
- MDs de análisis por TC fallido
- Recomendaciones de fix con score
- Dashboard de calidad publicado
- KB que se acumula con outcomes

**Templates relevantes**: `05_qap_framework.md`, `06_validation_approach.md`

---

## Fase 4 — ITERATE

**Tiempo**: continuo

**Mecanismos**:
- Fix aplicado → outcome medido → KB actualizado
- Patterns recurrentes detectados → re-asignación entre capas
- Mecanismos verificados → reutilización en bugs similares
- KB con anti-drift mantiene conocimiento fresco

**Outputs**:
- KB que crece y se refina
- Accuracy del skill medible y creciente
- Re-ajustes arquitectónicos pequeños justificados en datos

---

## Fase 5 — STRATEGIC

**Tiempo**: mensual o trimestral

**Línea activa**:
- **RES**: investigación continua en segundo plano

**Mecanismos**:
- Script `strategic_context_dump.py` ensambla contexto completo
- Análisis con Claude Sonnet 4.7 (1M context)
- Brief estratégico con 3-5 arquitecturas alternativas

**Outputs**:
- Briefs estratégicos mensuales
- Propuestas de refactor mayor cuando aplique
- Decisiones de pivot arquitectónico con evidencia

---

## Tiempos típicos por fase

| Fase | Versión rigurosa | Versión ligera |
|---|---|---|
| DESIGN | 2-3 semanas | 2-3 días |
| BUILD | 4-6 semanas | 2-3 semanas |
| VALIDATE | Continuo | Continuo |
| ITERATE | Continuo | Continuo |
| STRATEGIC | Mensual | Trimestral |

**Sumar las fases iniciales (DESIGN + BUILD)**:
- Riguroso: ~8-9 semanas
- Ligero: ~3 semanas

Para MVP cliente piloto: ligero. Para proyecto enterprise grande: riguroso.

---

## El cliclo virtuoso del KB

```
DESIGN inicial (imperfecto)
   ↓
BUILD ejecuta diseño
   ↓
VALIDATE encuentra fallos del diseño
   ↓
ITERATE alimenta KB con outcomes
   ↓
STRATEGIC propone redesign basado en evidencia
   ↓
DESIGN refinado (siguiente proyecto se beneficia)
```

**El KB acumula conocimiento cross-proyecto, no solo intra-proyecto**. Cada cliente nuevo arranca con mecanismos verificados de clientes anteriores (anonimizados).

---

## Anti-patrones del ciclo

| Anti-patrón | Síntoma | Corrección |
|---|---|---|
| Saltar DESIGN | Refactors constantes en BUILD | Volver a hacer Query Analysis |
| Saltar VALIDATE | Bugs en producción | Activar QAP inmediatamente |
| No iterar | KB inerte, mismos errores | Asegurar que cada fix actualiza el KB |
| Strategic sin evidencia | Redesigns por opinión | Acumular ≥3 meses de KB antes de strategic |

---

## Vendible al cliente

Frase clave:

> "No vendo un proyecto, vendo un ciclo. Cada cliente entra al ciclo, su agente mejora con uso, el conocimiento acumulado en el KB beneficia al siguiente cliente. **Es un activo compuesto, no un entregable puntual**."
