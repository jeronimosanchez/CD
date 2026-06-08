# Backlog — Sistema de Automatización CD (v1)

> **Visión:** diseñar Y optimizar agentes conversacionales en distintos ámbitos,
> arquitecturas y realidades de cliente — agnóstico y multiplataforma —
> orquestando cada plataforma a través de su API (no se toca la consola).
>
> **Borrador para volcar a Notion** (backlog maestro único, consolidando ACT_Backlog + QAP_Backlog).
> Fecha: 2026-06-05

---

## Molde de cada item
```
ID · Tipo · Línea · Fase · Prioridad(MoSCoW) · Complejidad · Scope in/out
Criterios(given/when/then) · DoD(por tipo, + "Principio") · Trazabilidad("sirve a"+deps)
Estado · Validación(Requerida/No · Recordado IA · Confirmado tú)
```

## Restricciones del sistema
```
IP privada (método fuera de lo entregable) · $0 local · solo · sin plazo (por valor)
Agnóstico/multiplataforma · validación final en sistema real · datos = TCs realistas (proxy)
```

## MVP
```
E1 + E2 + E4 + E3(parcial: adversarial + cluster + hypothesis-generator) sobre Petal
→ "detectar + diagnosticar con evidencia a ESCALA, overnight, $0"
```

---

# FASE 1 — Cimientos (Must)

## E1 — KB núcleo (compartidos) · Sistema · M · casi hecho
| ID | Story | Estado |
|---|---|---|
| SYS-US-01 | kb_global (P1-13 · A1-4 · G1-2 · V1-5) | ✅ hecho |
| SYS-US-02 | kb_cx | ✅ hecho |
| SYS-US-03 | kb_skills | ✅ hecho |
| SYS-US-04 | kb_petal | ✅ hecho |
| SYS-US-05 | Gobierno KB (_index · _politica · _mapa) | ✅ hecho |

## E2 — ADK setup (runtime CX) · ACT · M
- Scope in: ADK local corriendo, reproduce conversaciones CX, $0 · out: adapters otras plataformas
| ID | Story |
|---|---|
| ACT-US-01 | Instalar/configurar ADK local |
| ACT-US-02 | Validar fidelidad ADK vs CX real (~95%) — *Validación: Requerida* |
| ACT-US-03 | Conectar tool calls (petal-sheet-api) en ADK |

## E4 — RES: criterios operacionalizados · RES · L · **prerrequisito del motor**
| ID | Story | DoD |
|---|---|---|
| RES-US-01 | Operacionalizar los 13 principios de KB_GLOBAL en checks/TCs | tipo "Principio" |
| RES-US-02 | Definir rúbricas del juez (qué mide cada check) | — |
| RES-US-03 | Golden set inicial (~50 conversaciones validadas) | *Validación: Requerida* |

## E3 — Skills Fase 1 (motor) · QAP/GEN · L · depende de E2+E4 · ⚠️8 stories (vigilar split)
| ID | Story (skill + su KB especialista) |
|---|---|
| GEN-US-01 | adversarial (genera conversaciones realistas + outputs esperados) |
| QAP-US-01 | cluster-analyzer (agrupa patrones) |
| QAP-US-02 | playbook-expert + KB_PLAYBOOK |
| QAP-US-03 | inventory-expert + KB_INVENTORY |
| QAP-US-04 | git-expert + KB_GIT |
| QAP-US-05 | hypothesis-generator |
| QAP-US-06 | hypothesis-fixer |
| QAP-US-07 | hypothesis-validator — *Validación: Requerida* |

## E5 — Backlog maestro + tooling meta · Sistema · S
| ID | Story | Tipo |
|---|---|---|
| SYS-US-06 | Montar base maestra en Notion (consolidar ACT+QAP backlog) | Story |
| SYS-EN-01 | done-checker (verifica DoD solo) | Enabler |
| SYS-EN-02 | KPI dashboard (curva bugs/noche · % resuelto · $/noche) | Enabler |

---

# FASE 2 — Profundidad (Should)

## E7 — Skills Fase 2 · QAP · M
| ID | Story |
|---|---|
| QAP-US-08 | test-expert + KB_TEST |
| QAP-US-09 | integration-expert + KB_INTEGRATION |
| QAP-US-10 | infra-expert + KB_INFRA |
| QAP-US-11 | qa-tc-analyzer → evoluciona a orquestador |

## E6 — Orden Notion · Sistema · S
| ID | Story |
|---|---|
| SYS-US-07 | Consolidar teamspaces legacy en los R_ canónicos |
| SYS-US-08 | Archivar duplicados (Met-S60: Automatización CD [Archivado], Transversales, Farma/Petal dup) |

---

# FASE 3 — Expansión (Could)

| Épica | ID | Story | Línea |
|---|---|---|---|
| E8 — RAG | SYS-US-09 | RAG de hipótesis/principios (capa a demanda, con volumen) | Sistema |
| E9 — Adapters | ACT-US-04 | Adapter Lex · ACT-US-05 Adapter Rasa | ACT |
| E10 — Multi-cliente | SYS-US-10 | 2º cliente real (tras piloto Petal) | Sistema |
| E11 — Skills Fase 3 | QAP-US-12 | routing-expert + KB_ROUTING · QAP-US-13 llm-expert + KB_LLM | QAP |

---

## Pendientes anotados (no son épicas todavía)
- Articular el mecanismo de conocimiento: cómo el sistema usa Petal y cómo vuelven los aprendizajes al KB.
- Recolocar ADK en KB_SKILLS como "runtime del adapter CX" (hoy aparece como universal).

## Resumen
```
11 épicas · ~30 stories/enablers · 3 fases
Fase 1 (Must): E1·E2·E4·E3·E5  → el MVP vive aquí
Fase 2 (Should): E7·E6
Fase 3 (Could): E8·E9·E10·E11
```
