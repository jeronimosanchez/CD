# Ciclo de Entrenamiento A — v1 (borrador para optimizar)
> Estado: borrador en revisión · Fecha: 2026-06-06

## Pasos y LLMs

**PASO 1 — QAP diseña el plan de cobertura**
- LLM: Claude
- Cómo: Lee ciclos anteriores + hipótesis activas + lista de componentes → genera coverage_plan.yaml

**PASO 2 — QAP adversarial genera conversaciones difíciles**
- LLM: Qwen (Ollama local, M4, $0)
- Cómo: Lee coverage_plan.yaml + kb_petal → genera 2.000 conversaciones diseñadas para romper el sistema

**PASO 3 — ACT ejecuta en ADK**
- LLM: ninguno (ADK runtime)
- Cómo: Ejecuta 2.000 conversaciones contra Petal local → devuelve PASS/FAIL + respuesta real
- Coste: $0

**PASO 4 — QAP cluster-analyzer detecta patrones**
- LLM: modelo de embeddings local (sentence-transformers, $0)
- Cómo: Agrupa FAILs por similitud semántica → N patrones + severidad

**PASO 5 — QAP hypothesis-generator propone fixes**
- LLM: Claude
- Cómo: Lee cluster + kb_playbook + kb_inventory + kb_git + log JSONL → 1 hipótesis por cluster con predicción de mejora

**PASO 6 — QAP comité valida teóricamente**
- LLM: Claude × 3 instancias en paralelo (playbook-expert · inventory-expert · git-expert)
- Cómo: Cada experto carga su KB + hipótesis → FUNCIONA/NO (binario) → top 3-5 pasan

**PASO 7 — ACT prueba fixes en ADK**
- LLM: ninguno (ADK runtime)
- Cómo: Aplica fix temporalmente → re-ejecuta conversaciones afectadas → compara PASS% antes/después
- Coste: $0

**PASO 8 — GEN genera el artefacto concreto**
- LLM: Claude (hypothesis-fixer)
- Cómo: Lee hipótesis validada + kb_global + kb_playbook → genera diff real del playbook

**PASO 9 — QAP valida en staging con golden set**
- LLM: Gemini (agente en staging CX) + Claude (juez evaluador)
- Cómo: Despliega diff en staging → ejecuta 50 conversaciones golden set → evalúa mejora sin regresiones
- Coste: ~$0.25 por fix × 5 fixes = ~$1.25

**PASO 10 — ACT despliega a producción**
- LLM: ninguno (GitHub Actions CI/CD)
- Cómo: PR automático → pipeline → deploy CX producción
- Coste: $0

**PASO 11 — Sistema aprende**
- LLM: Claude
- Cómo: Añade entrada al log JSONL → actualiza KB especialista → actualiza dashboard

## Resumen de LLMs

| LLM | Rol |
|---|---|
| Claude | Planifica · hipotetiza · valida · aprende |
| Qwen/Ollama (local) | Adversarial — rompe el sistema ($0) |
| Embeddings (local) | Clustering ($0) |
| Gemini | El agente en staging (ya existe en Petal) |
| ADK | Runtime de ejecución (no LLM) |
| GitHub Actions | Deploy (no LLM) |

## Coste estimado por ciclo
- ADK: $0
- Embeddings: $0
- Qwen adversarial: ~$0.10 (electricidad)
- Staging golden set: ~$1.25
- Total: ~$1.35-1.80 por noche
