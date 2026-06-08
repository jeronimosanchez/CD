# CD Templates Library — Overview

**Propósito**: librería de metodología, templates y ejemplos para construir agentes conversacionales de calidad production-grade en cualquier plataforma (CX, Lex, Voiceflow, custom).

**Audiencia**: equipos de Conversational Design, AI Pods, consultoras que entregan agentes a clientes enterprise.

---

## Las 4 líneas del sistema de Automatización CD

```
ACT (Deploy)        — Despliegue automático de artefactos a la plataforma
GEN (Generación)    — Generación de playbooks, examples, intents
QAP (Calidad)       — Validación de calidad automática + análisis empírico
RES (Research)      — Investigación continua en segundo plano
```

**+ una capa upstream que se suele olvidar:**

```
DESIGN (Diseño)     — Decisiones arquitectónicas antes de construir
                       (Query Analysis, Layer Assignment, briefing)
```

Sin DESIGN bien hecho, las otras 4 líneas construyen sobre fundación arbitraria.

---

## Las 5 fases del ciclo de vida de un agente CD

```
1. DESIGN     → briefing + query analysis + layer assignment
2. BUILD      → ACT + GEN ejecutan el diseño
3. VALIDATE   → QAP analiza, detecta bugs, propone fixes
4. ITERATE    → outcomes alimentan KB, ajustan diseño
5. STRATEGIC  → RES propone redesigns arquitectónicos periódicos
```

Cada fase tiene templates y metodología en esta librería.

---

## Estructura de la librería

```
cd_templates_library/
├── 00_overview.md                    ← este archivo
├── methodology/                      ← cómo hacer cada cosa
│   ├── 01_briefing_template.md
│   ├── 02_query_analysis.md
│   ├── 03_layer_assignment.md
│   ├── 04_architecture_derivation.md
│   ├── 05_qap_framework.md
│   ├── 06_validation_approach.md
│   └── 07_lifecycle.md
├── templates/                        ← plantillas para llenar
│   ├── briefing_form.md
│   ├── query_inventory.yaml
│   ├── layer_assignment_matrix.md
│   └── backlog_cd_default.md
└── examples/                         ← casos reales como referencia
    └── petal_filled_briefing.md
```

---

## Cómo usar esta librería

| Si estás en... | Lee... | Llena... |
|---|---|---|
| Arranque de proyecto nuevo | `01_briefing_template.md` | `briefing_form.md` |
| Después del briefing | `02_query_analysis.md` | `query_inventory.yaml` |
| Decidiendo arquitectura | `03_layer_assignment.md` | `layer_assignment_matrix.md` |
| Construyendo | `04_architecture_derivation.md` | — |
| Validando calidad | `05_qap_framework.md` + `06_validation_approach.md` | — |
| Operando en producción | `07_lifecycle.md` | — |

---

## Principios transversales

1. **Bottom-up validado, top-down articulado** — la metodología emergió de trabajo real, no de teoría
2. **Versión ligera > versión perfecta** — pragmatismo gana a rigor académico
3. **Híbrido NLU + LLM** — la arquitectura del futuro no es 100% nada
4. **Closed-loop empirical** — los outcomes validan, no los expertos
5. **Multi-plataforma agnóstica** — la metodología sirve para CX, Lex, Voiceflow, custom

---

## Estado de la librería

| Componente | Estado |
|---|---|
| Overview (este archivo) | ✅ |
| Methodology files | ✅ inicial — refinables con uso |
| Templates | ✅ inicial — base para llenar |
| Examples (Petal) | ✅ inicial — caso piloto |
| Versionado | v0.1 — primera iteración |

**Esta librería es viva**. Se refina con cada proyecto que la usa.
