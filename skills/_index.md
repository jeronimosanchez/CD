# _index — Registro de skills del sistema

> ⮕ EMPIEZA AQUÍ para saber qué skills existen y cuál necesitas.
> Simétrico a ~/CD/kb/_index.md — mismo gobierno, misma lógica.
> Para añadir una skill, editar solo este archivo.
> Última actualización: 2026-06-07

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

| Skill | Función | Línea | Estado |
|---|---|---|---|
| **qap_sys_plan_generator** | Paso 1 del ciclo autónomo — decide qué testear cada noche: lee historial, FAILs y cobertura, produce coverage_plan.yaml | QAP | 🔴 |
| **gen_ag_adversarial** | Genera conversaciones difíciles para romper cualquier agente — Qwen local, $0 | GEN | 🔴 |
| **qap_ag_cluster_analyzer** | Agrupa FAILs por similitud semántica usando embeddings locales → patrones con ROI, $0 | QAP | 🔴 |
| **qap_plat_cx_playbook_expert** | Experto comité capa comportamiento — lee YAML de CX, identifica instrucción causante con cita exacta, crea kb_proj_petal_playbook | QAP | 🔴 |
| **inventory-expert** | Analiza desde la perspectiva del catálogo/Sheet | QAP | 🔴 |
| **git-expert** | Analiza desde el historial git — ¿ya se intentó? ¿se revirtió? | QAP | 🔴 |
| **qap_ag_hypothesis_generator** | Sintetiza informes de N expertos del comité en 3-5 hipótesis accionables con predicción de mejora % — comité extensible, no repite hipótesis ya intentadas, carga kb_plat para generar soluciones factibles en la plataforma del cliente | QAP | 🔴 |
| **gen_plat_cx_hypothesis_fixer** | Genera diff YAML de CX listo para desplegar a partir de la hipótesis — Gemini Flash free (formato nativo CX) | GEN | 🔴 |
| **hypothesis-validator** | Prueba el fix en ADK (antes/después) + staging golden set | QAP | 🔴 |
| **juez-llm** | Evalúa output real del agente contra rúbricas — veredicto con evidencia | QAP | 🔴 |

### Fase 1 — QA heredado de ACT (en repo cx-automation-template)

| Skill | Ruta actual | Función | Estado |
|---|---|---|---|
| **qa-tc-analyzer** | `~/cx-automation-template/.claude/skills/qa-tc-analyzer/` | Analiza FAILs con 9 capas de causa raíz | 🟡 |
| **qa-fix** | `~/cx-automation-template/.claude/skills/qa-fix/` | Aplica fix → PR → deploy → valida | 🟡 |
| **qa-revert** | `~/cx-automation-template/.claude/skills/qa-revert/` | Revierte el último fix para demo/test | 🟡 |

### Fase 2 — Profundidad (E7)

| Skill | Función | Estado |
|---|---|---|
| **test-expert** | Analiza desde la perspectiva de los TCs — gaps, calibración | 🔴 |
| **integration-expert** | Analiza tools/webhooks/API — fallos de integración | 🔴 |
| **infra-expert** | Analiza environments/versions/CI-CD — problemas de deploy | 🔴 |

### Fase 3 — Avanzada (E11)

| Skill | Función | Estado |
|---|---|---|
| **routing-expert** | Analiza flows/intents/entities — enrutamiento NLU | 🔴 |
| **llm-expert** | Analiza comportamiento de Gemini — alucinaciones, varianza | 🔴 |

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
