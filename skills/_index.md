# _index — Registro de skills del sistema

> ⮕ EMPIEZA AQUÍ para saber qué skills existen y cuál necesitas.
> Simétrico a ~/CD/kb/_index.md — mismo gobierno, misma lógica.
> Para añadir una skill, editar solo este archivo.
> Última actualización: 2026-06-10

---

## Raíz fija

```
SKILLS_ROOT = ~/CD/skills/
```

Cada skill vive en su propia subcarpeta: `~/CD/skills/<nombre>/SKILL.md`

**Nota:** las skills del repo ACT viven en `~/cx-automation-template/.claude/skills/` mientras pertenezcan a ese repo.
Cuando el sistema QAP madure y tenga repo propio, migrarán a `~/CD/skills/`.

---

## Estados

> - 🔴 **No existe** — SKILL.md no creado aún
> - 🟡 **En curso** — SKILL.md existe, pendiente de validar
> - ✅ **Validado** — probada sobre el sistema real

---

## Skills registradas

### Fase 1 — Motor (E3)

| Paso | Skill | Función | Línea | Tipo | Modelo | Estado |
|---|---|---|---|---|---|---|
| 1 | **qap_sys_plan_generator** | Decide qué testear cada noche — historial + FAILs + cobertura → coverage_plan.yaml | QAP | Skill | Claude API | 🔴 |
| 2 | **gen_ag_adversarial** | Genera conversaciones difíciles para romper cualquier agente | GEN | Skill | Qwen 32B local ($0) | 🔴 |
| 3 | ADK runner | Ejecuta conversaciones contra el agente — devuelve PASS/FAIL | — | **Infra** | — | 🔴 |
| 5 | **qap_ag_cluster_analyzer** | Agrupa FAILs por similitud semántica → patrones con ROI | QAP | Skill | Embeddings local ($0) | 🔴 |
| 6 | **qap_plat_cx_playbook_expert** | Experto comité capa comportamiento — identifica instrucción causante con cita exacta | QAP | Skill | Gemini Flash free | 🔴 |
| 7 | **qap_ag_hypothesis_generator** | Sintetiza los informes del comité en un diferencial ranqueado de 2-4 hipótesis de CAUSA (abducción) — extensible, no repite intentos fallidos. Es diagnóstico; el fix concreto lo genera `gen_plat_cx_hypothesis_fixer`. | QAP | Skill | Claude API | 🔴 |
| 8 | **gen_plat_cx_hypothesis_fixer** | Genera cambio puntual (sección + contenido nuevo) — NO el playbook completo | GEN | Skill | Gemini Flash free | 🔴 |
| 9 | hypothesis-validator | Testa el fix: Fase A ADK + Fase B staging CX (entornos efímeros por hipótesis) | — | **Infra** | — | 🔴 |
| 10 | **qap_ag_juez** | Evalúa respuesta real vs rúbrica — veredicto por criterio (sí/no/parcial) con evidencia | QAP | Skill | Claude API | 🔴 |
| 10 | scorer | Agrega veredictos del juez → PASS / PARTIAL / FAIL + % numérico | — | **Infra** | — | 🔴 |

### Fase 1 — QA heredado de ACT (en repo cx-automation-template)

| Skill | Ruta actual | Función | Estado |
|---|---|---|---|
| **qa-tc-analyzer** | `~/cx-automation-template/.claude/skills/qa-tc-analyzer/` | Analiza FAILs con 9 capas de causa raíz | 🟡 |
| **qa-fix** | `~/cx-automation-template/.claude/skills/qa-fix/` | Aplica fix → PR → deploy → valida | 🟡 |
| **qa-revert** | `~/cx-automation-template/.claude/skills/qa-revert/` | Revierte el último fix para demo/test | 🟡 |

### GEN — Motor de generación de playbooks

| Paso | Skill | Función | Línea | Tipo | Modelo | Estado |
|---|---|---|---|---|---|---|
| 1 | **gen_context_loader** | Lee el brief → identifica y carga KBs relevantes → contexto estructurado para el generador | GEN | Skill | Sonnet 4.6 | 🔴 |
| 2 | **gen_ag_generator** | Genera 5 variantes completas del playbook con enfoques distintos | GEN | Skill | Qwen 32B local ($0) | 🔴 |
| 3 | **gen_ag_reviewer** | Evalúa las 5 variantes contra criterios KB → descarta 2 → top 3 rankeadas | GEN | Skill | Sonnet 4.6 | 🔴 |
| 4 | **gen_ag_adversarial** | Stress test: intenta romper cada candidata con conversaciones difíciles | GEN | Skill | Qwen 32B local ($0) | 🔴 |
| 5 | **gen_plat_cx_hypothesis_fixer** | Fix puntual sobre playbook existente (usado en Sistema A, paso 8) | GEN | Skill | Gemini Flash free | 🔴 |

> Trigger: bajo demanda (no cron). Las top 3 del reviewer pasan a QAP (Sistema A) para validación final antes del gate humano.

### Sistema B — Bucle de conocimiento (E14)

| Paso | Skill | Función | Línea | Tipo | Modelo | Estado |
|---|---|---|---|---|---|---|
| 1 | **sys_b_extractor** | Lee logs de Sistema A → registros de experimento estructurados (✅/❌/⚠️) | Sistema B | Skill | Sonnet 4.6 | 🔴 |
| 2 | **sys_b_classifier** | Aplica 3 tests → asigna nivel KB (proyecto/servicio/universal/plataforma) + confianza | Sistema B | Skill | Opus 4.8 | 🔴 |
| 3 | **sys_b_writer** | Detecta conflictos/duplicados → genera borrador → gate humano → escribe KB | Sistema B | Skill | Sonnet 4.6 | 🔴 |

> Cadencia: cada 7 ciclos de Sistema A (síntesis estándar) o inmediato (contradicción detectada).

### Reserva (`skills/_reserve/`) — retiradas de diagnóstico

> Skills retiradas al simplificar la causa raíz del Sistema A (**modelo comité → pipeline**). Las que **existían** se conservan en `~/CD/skills/_reserve/`. Las **expertas por elemento que nunca se construyeron** se quitaron de la lista (eran aspiracionales del modelo comité; el trace de CX ya cubre esas capas o el pipeline no las necesita): **test-expert, integration-expert, infra-expert, routing-expert, llm-expert**.

| Skill | Por qué se retiró | Ubicación |
|---|---|---|
| **qap_proj_petal_inventory_expert** | Redundante — la info de inventario ya la da el trace de la tool call en CX | `skills/_reserve/qap_proj_petal_inventory_expert/` |
| **qap_ag_git_expert** | Lookup mecánico (`git log`); no necesita ser skill LLM | `skills/_reserve/qap_ag_git_expert/` |

---

## Datos operativos del sistema (no son KBs ni skills)

> Archivos de estado que el ciclo autónomo lee y escribe. No contienen conocimiento — contienen estado.

| Archivo | Ruta | Quién escribe | Quién lee |
|---|---|---|---|
| `log_hypotheses.jsonl` | `~/CD/proyectos/<proyecto>/_durable/` | log+KB (paso 11) | plan-generator · hypothesis-generator |
| `coverage_matrix.yaml` | `~/CD/proyectos/<proyecto>/_durable/` | log+KB (paso 11) ⚠️ pendiente incluir en su descripción | plan-generator |

---

## Mapa skill → KB (provisional — se cierra al definir cada SKILL.md)

Ver `~/CD/kb/_index.md` sección "Mapa de carga por skill".
