# Sistema A — Mapping: lo que tenemos vs lo que estamos definiendo
> Estado: trabajo en curso · Fecha: 2026-06-06
> Objetivo: decidir qué eliminar, qué renombrar, qué añadir antes de tocar el backlog

---

## Tabla de mapping

| Paso ciclo A | Qué hace | LLM | EN EL BACKLOG (ID) | DEFINIENDO AHORA | Acción |
|---|---|---|---|---|---|
| **Paso 1** | Diseña el plan de cobertura noche a noche | Claude | ❌ No existe | QAP plan-generator + coverage matrix | CREAR |
| **Paso 2** | Genera conversaciones difíciles para romper el sistema | Qwen/Ollama local | GEN-US-01 (mal etiquetado como GEN) | QAP adversarial (Qwen local) | MOVER a QAP + actualizar descripción |
| **Paso 2b** | Perfiles de usuario como semilla | Qwen local | GEN-US-03 | — | ✅ OK, absorber en Paso 2 |
| **Paso 2c** | Generación de outputs esperados | Qwen local | GEN-US-04 | — | ✅ OK, absorber en Paso 2 |
| **Paso 2d** | Setup Ollama/Qwen local | — | GEN-US-02 | — | ✅ OK, cambiar línea a QAP |
| **Paso 3** | Ejecuta conversaciones en ADK (runtime) | ADK (no LLM) | ACT-US-06 batch runner | — | ✅ OK |
| **Paso 4** | Agrupa patrones de fallo por similitud | Embeddings local | QAP-US-01 cluster-analyzer | — | ✅ OK |
| **Paso 5** | Propone hipótesis + predicción de mejora | Claude | QAP-US-05 hypothesis-generator | Redefinir con más detalle (lee log JSONL, predice %) | ACTUALIZAR descripción |
| **Paso 6** | Comité valida teóricamente (binario) | Claude ×3 | QAP-US-02 playbook-expert | — | ✅ OK |
| **Paso 6b** | — | Claude ×3 | QAP-US-03 inventory-expert | — | ✅ OK |
| **Paso 6c** | — | Claude ×3 | QAP-US-04 git-expert | — | ✅ OK |
| **Paso 7** | Re-ejecuta en ADK con fix aplicado (confirma mejora) | ADK (no LLM) | ACT-US-06 batch runner | — | ✅ OK (mismo runner) |
| **Paso 8** | Genera el artefacto concreto (diff del playbook) | Claude | QAP-US-06 hypothesis-fixer (mal en QAP) | GEN hypothesis-fixer | MOVER a GEN |
| **Paso 9** | Valida en staging con golden set | Gemini + Claude juez | QAP-US-14 Juez LLM + RES-US-03 golden set | — | ✅ OK |
| **Paso 9b** | Coverage matrix tracker (histórico de cobertura) | — | ❌ No existe | QAP coverage-tracker | CREAR |
| **Paso 10** | Despliega a producción vía CI/CD | GitHub Actions | ACT pipeline (ya existe) | — | ✅ OK |
| **Paso 11** | Aprende: log JSONL + KB + dashboard | Claude | SYS-EN-03 / E13-US-03 (duplicado) | — | FUSIONAR en uno |

---

## Orquestador y duplicados

| ID | Título | Situación | Acción |
|---|---|---|---|
| QAP-US-15 | Orquestador del bucle overnight | Demasiado genérico — ahora está distribuido en 11 pasos bien definidos | REVISAR: ¿renombrar a plan-generator o eliminar? |
| QAP-US-11 | qa-tc-analyzer → orquestador | Es la evolución del QA actual de Petal al orquestador. Diferente al overnight. | ✅ OK, mantener |
| SYS-EN-03 | Enabler — Log JSONL de hipótesis | Duplicado de E13-US-03 | ELIMINAR este, conservar E13-US-03 |
| RES-US-03 | Golden set inicial | Posible duplicado de E13-US-01 | REVISAR si son el mismo o tienen diferente scope |

---

## Lo que falta crear

| Nuevo item | Paso | Línea | Por qué |
|---|---|---|---|
| QAP plan-generator | Paso 1 | QAP | No existe. Diseña el plan de cobertura cada noche con la fórmula de N_conversaciones |
| QAP coverage-tracker | Paso 9b | QAP | No existe. Matriz histórica de cobertura por componente |

---

## Resumen de acciones

```
CREAR    →  QAP plan-generator · QAP coverage-tracker
MOVER    →  GEN-US-01 adversarial (GEN → QAP)
             QAP-US-06 hypothesis-fixer (QAP → GEN)
             GEN-US-02 Setup Qwen (GEN → QAP)
ACTUALIZAR →  QAP-US-05 hypothesis-generator (descripción más detallada)
FUSIONAR  →  SYS-EN-03 + E13-US-03 → conservar E13-US-03
REVISAR   →  QAP-US-15 orquestador (¿renombrar o eliminar?)
             RES-US-03 vs E13-US-01 (¿duplicados?)
NO TOCAR  →  QAP-US-01·02·03·04·07·14 · ACT-US-06 · ACT pipeline
```
