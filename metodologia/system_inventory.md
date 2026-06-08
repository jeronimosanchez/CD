# System Inventory — Componentes del sistema CD por fase

**Propósito**: inventario completo de qué hace cada componente y en qué fase del ciclo opera. Punto único de referencia para entender el sistema.

**Convenciones**:
- ✅ Implementado y operativo
- 🟡 Implementación parcial / MVP
- ❌ Roadmap, no implementado
- 🔵 Activo externo (no del repo)

---

## Fase 1 — DESIGN (pre-arquitectura)

Decisiones tomadas antes de construir nada. Hoy minimalista, en evolución.

| Componente | Tipo | Función | Ubicación | Estado |
|---|---|---|---|---|
| `00_overview.md` | Doc | Overview de las 4 líneas + 5 fases del sistema | `definitions/cd_templates_library/` | ✅ |
| `01_briefing_template.md` | Doc methodology | Metodología de briefing al arrancar proyecto | `cd_templates_library/methodology/` | ✅ |
| `02_query_analysis.md` | Doc methodology | Cómo inventariar queries por frecuencia + complejidad | `cd_templates_library/methodology/` | ✅ |
| `03_layer_assignment.md` | Doc methodology | Criterio de asignación NLU vs LLM vs datos | `cd_templates_library/methodology/` | ✅ |
| `04_architecture_derivation.md` | Doc methodology | Cómo derivar arquitectura desde el mapping | `cd_templates_library/methodology/` | ✅ |
| `briefing_form.md` | Template | Plantilla del briefing para llenar | `cd_templates_library/templates/` | ✅ |
| `query_inventory.yaml` | Template | Plantilla del inventario de queries | `cd_templates_library/templates/` | ✅ |
| `layer_assignment_matrix.md` | Template | Plantilla de la matriz de asignación | `cd_templates_library/templates/` | ✅ |
| `petal_filled_briefing.md` | Example | Briefing de Petal como referencia | `cd_templates_library/examples/` | ✅ |
| `cd-briefing-assistant` | Skill | Guía al cliente a rellenar el briefing | `.claude/skills/` | ❌ |
| `cd-query-inventory-builder` | Skill | Asiste a inventariar queries automáticamente | `.claude/skills/` | ❌ |
| `cd-layer-assigner` | Skill | Sugiere asignación NLU/LLM | `.claude/skills/` | ❌ |
| `cd-architecture-generator` | Skill | Genera arquitectura concreta desde el mapping | `.claude/skills/` | ❌ |

---

## Fase 2 — BUILD (ACT + GEN)

Construcción y deploy del agente en la plataforma destino.

### ACT — Despliegue automático

| Componente | Tipo | Función | Ubicación | Estado |
|---|---|---|---|---|
| `push_playbooks.py` | Python | Despliega playbooks a CX (Full Update por bug regional) | `src/` | ✅ |
| `push_examples.py` | Python | Despliega examples a CX | `src/` | ✅ |
| `push_tools.py` | Python | Despliega tools a CX | `src/` | ✅ |
| `push_agent_config.py` | Python | Despliega config global del agente | `src/` | ✅ |
| `push_flows.py` | Python | Despliega flows a CX | `src/` | ✅ |
| `push_pages.py` | Python | Despliega pages a CX | `src/` | ✅ |
| `push_intents.py` | Python | Despliega intents a CX | `src/` | ✅ |
| `push_entity_types.py` | Python | Despliega entity types a CX | `src/` | ✅ |
| `push_webhooks.py` | Python | Despliega webhooks a CX | `src/` | ✅ |
| `push_generators.py` | Python | Despliega generators (LLM prompts) a CX | `src/` | ✅ |
| `push_environments.py` | Python | Despliega environments a CX | `src/` | ✅ |
| `push_versions.py` | Python | Crea snapshots de version (LRO polling) | `src/` | ✅ |
| `pull_*.py` (12 archivos) | Python | Inverso de push: descarga estado actual de CX | `src/` | ✅ |
| `validate_api.py` / `validate_api_v2.py` | Python | Valida credenciales + conectividad CX | `src/` | ✅ |
| `diff.py` | Python | Calcula diff entre local y CX (idempotencia) | `src/` | ✅ |
| `deploy.yml` | Workflow | CI/CD a CX (WIF auth, concurrency: 1, paths-filter) | `.github/workflows/` | ✅ |
| `definitions/playbooks/` | Definition | 6 playbooks de Petal (Orquestador, Compra, Checkout, etc.) | `definitions/` | ✅ |
| `definitions/examples/` | Definition | Few-shot examples por playbook | `definitions/` | ✅ |
| `definitions/tools/` | Definition | Tool definitions (PetalDataTool) | `definitions/` | ✅ |
| `definitions/flows/`, `pages/`, `intents/`, `entity_types/`, `webhooks/`, `generators/`, `environments/`, `versions/` | Definitions | 12 tipos de recursos CX completos | `definitions/` | ✅ |
| `agent.yaml` | Config | Configuración global del agente Petal | `definitions/` | ✅ |
| `requirements.txt` | Config | Dependencias Python del proyecto | raíz | ✅ |

### GEN — Generación

| Componente | Tipo | Función | Ubicación | Estado |
|---|---|---|---|---|
| `cd-test-suite-bootstrapper` | Skill | Genera TCs iniciales desde briefing + backlog default | `.claude/skills/` | ❌ |
| `backlog_cd_default.md` | Template | Backlog base de TCs transversales reutilizables | `cd_templates_library/templates/` | ✅ |

---

## Fase 3 — QAP (validación + análisis)

Verificación de calidad continua y análisis de bugs.

### Suite QA

| Componente | Tipo | Función | Ubicación | Estado |
|---|---|---|---|---|
| `test_QA_Playbooks_v23.py` | Python | Runner principal · 51 TCs contra CX real · genera dashboard HTML | `qa/` | ✅ |
| `list_fails.py` | Python | Lista TCs en FAIL del último run (input para análisis batch) | `qa/` | ✅ |
| `regenerate_html.py` | Python | Regenera dashboard HTML desde logs JSON + MDs de análisis | `qa/` | ✅ |
| `regenerate_all_html.sh` | Script | Regenera HTML de todos los runs históricos | `qa/` | ✅ |
| `rebuild_history.py` | Python | Reconstruye `history.json` del dashboard | `qa/` | ✅ |
| `rerun_single_tc.sh` | Script | Re-ejecuta un único TC contra CX | `qa/` | ✅ |
| `publish_html.sh` | Script | Publica HTML a gh-pages branch | `qa/` | ✅ |
| `qa.yml` | Workflow | CI QA · runs post-deploy + manual + PR · 29 TCs base | `.github/workflows/` | ✅ |

### Análisis (skills)

| Componente | Tipo | Función | Ubicación | Estado |
|---|---|---|---|---|
| `qa-tc-analyzer` v1 | Skill | Análisis 9 capas + propuesta de N soluciones | `.claude/skills/` | ✅ |
| `qa-orchestrator` | Skill | Coordina flujo end-to-end (analyzer → generator → tester → curator) | `.claude/skills/` | ❌ |
| `qa-candidate-generator` | Skill | Genera 3-30 candidatos diversos de fix por bug | `.claude/skills/` | ❌ |
| `qa-pre-validator` | Skill | Filtra candidatos vía Gemini API directa (cheap pre-CX) | `.claude/skills/` | ❌ |
| `qa-empirical-tester` | Skill | Deploya y testea candidatos contra CX, recoge outcomes | `.claude/skills/` | ❌ |
| `qa-synthesizer` | Skill | Combina mejores mecanismos en solución superior | `.claude/skills/` | ❌ |
| `qa-cluster-detector` | Skill | Clasifica bug + busca adyacentes + genera edge cases sintéticos | `.claude/skills/` | ❌ |
| `qa-platform-adapter-{X}` | Skill | Adapta el sistema a plataforma específica (CX/Lex/Voiceflow) | `.claude/skills/` | ❌ |

### Métodos documentados

| Componente | Tipo | Función | Ubicación | Estado |
|---|---|---|---|---|
| `05_qap_framework.md` | Doc methodology | 9 capas + KB + 5 estrategias + 11 retos del framework | `cd_templates_library/methodology/` | ✅ |
| `06_validation_approach.md` | Doc methodology | Closed-loop empirical validation explicada | `cd_templates_library/methodology/` | ✅ |
| `qa_analysis_process.md` | Doc | Proceso operativo de análisis | `docs/` | ✅ |
| `setup-cicd.md` | Doc | Setup del pipeline CI/CD | `docs/` | ✅ |
| `setup-qa.md` | Doc | Setup del QA dashboard | `docs/` | ✅ |

### Artefactos generados

| Componente | Tipo | Función | Ubicación | Estado |
|---|---|---|---|---|
| Dashboard QA público | Output | HTML interactivo con 51 TCs · histórico de runs · análisis embedded | gh-pages branch | ✅ |
| MDs de análisis por TC | Output | Análisis 9 capas detallado de cada bug | `qa/tc_analysis/{TS}/` | ✅ |
| Patterns cross-TC | Output | Detección automática de patrones de bug | `qa/tc_analysis/{TS}/_patterns_*.md` | ✅ |

---

## Fase 4 — ITERATE (KB + retroalimentación)

Conocimiento acumulado y mejora continua. Esta fase es ROADMAP completo.

| Componente | Tipo | Función | Ubicación | Estado |
|---|---|---|---|---|
| `system_knowledge.md` | KB file | Mapa del agente analizado (cargado al inicio del análisis) | `qa/analysis_kb/` | ❌ |
| `mechanisms_library.yaml` | KB file | Mecanismos de fix verificados con efectividad medida | `qa/analysis_kb/` | ❌ |
| `patterns_catalog.yaml` | KB file | Patrones de bug recurrentes detectados cross-TC | `qa/analysis_kb/` | ❌ |
| `gemini_behaviors.md` | KB file | Comportamientos verificados empíricamente del LLM | `qa/analysis_kb/` | ❌ |
| `decisions.log` | KB file | Audit trail completo (cada análisis con su outcome) | `qa/analysis_kb/` | ❌ |
| `confidence_track.yaml` | KB file | Métricas agregadas de precisión por capa del análisis | `qa/analysis_kb/` | ❌ |
| `qa-knowledge-curator` | Skill | Mantiene el KB · validation gates · curation periódica | `.claude/skills/` | ❌ |
| `qa-drift-detector` | Skill | Detecta concept drift cuando el sistema cambia · re-validación | `.claude/skills/` | ❌ |

---

## Fase 5 — STRATEGIC (research + redesigns)

Análisis transversal y propuestas arquitectónicas. Cadencia mensual.

| Componente | Tipo | Función | Ubicación | Estado |
|---|---|---|---|---|
| `strategic_context_dump.py` | Python | Ensambla contexto completo (~500K tokens) para análisis estratégico | `qa/strategic_analyzer/` | ❌ |
| `qa-strategic-advisor` | Skill | Análisis transversal mensual semi-automatizado | `.claude/skills/` | 🟡 (lite, manual con Claude chat) |
| `cd-real-log-miner` | Skill | Extrae señales de error de logs conversacionales reales del cliente | `.claude/skills/` | ❌ |
| Strategic briefs mensuales | Output | Documento estratégico con 3-5 arquitecturas alternativas | `qa/strategic_briefs/` | ❌ |
| `07_lifecycle.md` | Doc methodology | Define el ciclo completo con cadencia y outputs | `cd_templates_library/methodology/` | ✅ |

---

## Componentes transversales (sirven a varias fases)

### Configuración del repo

| Componente | Tipo | Función | Ubicación | Estado |
|---|---|---|---|---|
| `CLAUDE.md` | Config | Instrucciones del proyecto para Claude Code | raíz | ✅ |
| `README.md` | Doc | Documentación pública del proyecto | raíz | ✅ |

### Memoria del proyecto

| Componente | Tipo | Función | Ubicación | Estado |
|---|---|---|---|---|
| `MEMORY.md` | Memory | Índice general de la memoria | `memory/` | ✅ |
| `SESIONES_ACTIVAS.md` | Memory | Sesiones paralelas trabajando · qué tocan | `memory/` | ✅ |
| `automatizacion/` | Memory dir | Aprendizajes sobre el template/infraestructura | `memory/` | ✅ |
| `petal/` | Memory dir | Decisiones específicas de Petal como caso de uso | `memory/` | ✅ |
| `shared/` | Memory dir | Aprendizajes transversales (políticas, feedback) | `memory/` | ✅ |
| `current/` | Memory dir | Estado activo del proyecto · próximos pasos | `memory/` | ✅ |
| `archived/` | Memory dir | Memorias obsoletas conservadas por trazabilidad | `memory/` | ✅ |
| `epicas_skill_qa_analyzer_v2.md` | Memory | Roadmap completo de las 7 épicas del skill | `memory/automatizacion/` | ✅ |

### Backend externo

| Componente | Tipo | Función | Ubicación | Estado |
|---|---|---|---|---|
| `petal-sheet-api` | Backend | Cloud Run service · 6 endpoints (business, agent_copy, inventario, perfil, pedidos) | externo (otro repo) | 🔵 |
| Google Sheets | Backend | Fuente de datos de negocio (business, inventario, perfil, pedidos, etc.) | externo (Google) | 🔵 |
| Dialogflow CX | Plataforma | Donde corre Petal · europe-west1 · agent 745375ba-... | externo (Google) | 🔵 |

---

## Resumen agregado

### Por fase

| Fase | Componentes | Implementados | Roadmap |
|---|---|---|---|
| DESIGN | 13 | 9 | 4 |
| BUILD | 24 | 24 | 0 |
| QAP | 19 | 11 | 8 |
| ITERATE | 8 | 0 | 8 |
| STRATEGIC | 5 | 1 (lite) | 4 |
| Transversal | 11 | 11 | 0 |
| **TOTAL** | **80** | **56** | **24** |

### Por tipo

| Tipo | Cantidad |
|---|---|
| Skills | 17 (1 implementada + 16 roadmap) |
| Python scripts | 28 (todos implementados) |
| Workflows CI/CD | 2 (todos implementados) |
| Doc methodology | 7 (todos implementados) |
| Templates | 4 (todos implementados) |
| KB files | 6 (todos roadmap) |
| Memory dirs | 5 (todos implementados) |
| Definitions CX | 12 tipos de recurso + agent.yaml |
| Outputs/artefactos | 4 (3 implementados + 1 roadmap) |
| Componentes externos | 3 (referenciados, no en repo) |

### Estado global del sistema

```
✅ Implementado:  56 componentes (70%)
🟡 Parcial:        2 componentes (3%)
❌ Roadmap:       22 componentes (27%)
🔵 Externos:       3 componentes (referenciados)
```

**Conclusión**: el sistema tiene un esqueleto operativo robusto (BUILD + QA suite + dashboard). Los componentes en roadmap son fundamentalmente skills nuevas y el KB persistente — las piezas que multiplicarían el valor.

---

## Cómo usar este inventario

| Si necesitas... | Dónde mirar |
|---|---|
| Entender el sistema completo | Empezar por overview (`00_overview.md`) |
| Modificar el deploy de algo | `src/push_*.py` correspondiente |
| Añadir un nuevo TC | `qa/test_QA_Playbooks_v23.py` + suite |
| Analizar un bug | `qa-tc-analyzer` skill + 9 capas |
| Implementar una skill nueva | Mirar épicas en `memory/automatizacion/epicas_skill_qa_analyzer_v2.md` |
| Onboarding de alguien nuevo | Este inventario + `CLAUDE.md` + overview |

---

## Versionado

| Versión | Fecha | Cambios |
|---|---|---|
| 0.1 | 2026-05-28 | Versión inicial con inventario completo por fase |
