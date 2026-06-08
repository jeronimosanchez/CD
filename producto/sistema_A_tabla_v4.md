# Sistema A — Tabla completa de skills
> Fecha: 2026-06-06
> 🟢 Se mantiene | 🔴 Eliminar/cambiar | ⬜ No existe | ❓ Revisar

| Paso | Descriptor | LLM | Qué hace en el ciclo | Backlog actual (ID) | Qué dice el backlog actual | Nuevo en backlog | Qué debería hacer |
|---|---|---|---|---|---|---|---|
| **1** | plan-generator | Claude | Diseña qué testear cada noche, calcula N_conversaciones por hipótesis/componente con fórmula estadística | ⬜ No existe | — | ➕ QAP-US-18 | Leer ciclos anteriores + hipótesis activas + cobertura histórica → producir coverage_plan.yaml con N_conversaciones y N_runs por hipótesis y componente |
| **2** | adversarial | Qwen local ($0) | Genera conversaciones difíciles diseñadas para romper el sistema siguiendo el coverage plan | 🔴 GEN-US-01 | Genera conversaciones realistas + outputs esperados + mutaciones en local | ➕ QAP-US-19 | Genera conversaciones DIFÍCILES (edge cases, inputs trampa) siguiendo coverage_plan.yaml · diferente LLM para diferente sesgo |
| **2** | Setup Ollama/Qwen | — (infra) | Instala y configura Qwen en M4 | 🔴 GEN-US-02 | (sin descripción detallada) | 🔄 Mover a QAP | Misma descripción, línea correcta |
| **2** | Perfiles usuario | Qwen local | Tipos de usuario que simula el adversarial | 🔴 GEN-US-03 | (sin descripción detallada) | 🔄 Absorber en QAP-US-19 | Se integra dentro del adversarial como input |
| **2** | Outputs esperados | Qwen local | Qué debería responder el agente por conversación | 🔴 GEN-US-04 | (sin descripción detallada) | 🔄 Absorber en QAP-US-19 | Se integra dentro del adversarial como output esperado |
| **3** | ADK runner | No LLM (runtime) | Ejecuta las conversaciones contra Petal en local ($0), devuelve PASS/FAIL por turno | 🟢 ACT-US-06 | Lanzar N conversaciones en paralelo en ADK para el volumen overnight · 10k+ conversaciones manejables | — | Sin cambios |
| **4** | cluster-analyzer | Embeddings local ($0) | Agrupa FAILs por similitud semántica, detecta patrones con frecuencia y ROI | 🟢 QAP-US-01 | Agrupa los JSONs en patrones, prioriza por ROI · 8-12 clusters con frecuencia + impacto | — | Sin cambios |
| **5** | hypothesis-generator | Claude | Propone hipótesis por cluster con predicción de mejora, consulta log JSONL para no repetir hipótesis fallidas | 🔴 QAP-US-05 | Combina informes de expertos y genera hipótesis grounded con predicción cuantificable | 🔄 Actualizar | Añadir: consulta log JSONL antes de proponer · no repite hipótesis ya fallidas · incluye % predicción de mejora |
| **6** | playbook-expert | Claude | Valida hipótesis desde perspectiva del playbook (binario FUNCIONA/NO) | 🟢 QAP-US-02 | (experto de playbook) | — | Sin cambios |
| **6** | inventory-expert | Claude | Valida hipótesis desde perspectiva del inventario (binario FUNCIONA/NO) | 🟢 QAP-US-03 | (experto de inventario) | — | Sin cambios |
| **6** | git-expert | Claude | Valida si el fix ya se intentó antes (binario FUNCIONA/NO) | 🟢 QAP-US-04 | (experto de git) | — | Sin cambios |
| **7** | ADK runner | No LLM (runtime) | Re-ejecuta conversaciones afectadas con fix aplicado, compara PASS% antes/después | 🟢 ACT-US-06 | (mismo runner que paso 3) | — | Sin cambios |
| **8** | hypothesis-fixer | Claude | Genera el artefacto concreto: el diff real del playbook listo para desplegar | 🔴 QAP-US-06 | Genera y aplica el fix para un cluster completo · despliega en staging vía PR | 🔄 Mover a GEN | Línea correcta: GEN. Es quien genera el artefacto desplegable |
| **9** | Juez LLM | Claude | Evalúa si el fix mejora el golden set en staging sin regresiones (PASS/FAIL + motivo) | 🟢 QAP-US-14 | Compara output real vs esperado usando rúbricas · casos ambiguos van a cola humana | — | Sin cambios |
| **9** | Golden set | Gemini (staging CX) | 50 conversaciones validadas que son la referencia de qué es correcto | 🔴 RES-US-03 + E13-US-01 | (duplicados) | 🔄 Fusionar en E13-US-01 | Conservar E13-US-01, eliminar RES-US-03 |
| **9** | coverage-tracker | No LLM | Matriz histórica: cuántas conversaciones por componente, qué tipos, cuándo, % cobertura | ⬜ No existe | — | ➕ QAP-US-20 | Actualiza la matriz después de cada ciclo · identifica componentes con baja cobertura para el próximo plan |
| **10** | CI/CD deploy | No LLM (GitHub Actions) | Despliega a producción los fixes con doble validación ADK + staging | 🟢 ACT pipeline | (pipeline existente) | — | Sin cambios |
| **11** | Log JSONL | No LLM | Registro append-only de hipótesis, resultados y aprendizajes de cada ciclo | 🔴 SYS-EN-03 + E13-US-03 | (duplicados) | 🔄 Fusionar en E13-US-03 | Conservar E13-US-03, eliminar SYS-EN-03 |
| **11** | Dashboard madurez | No LLM | Curva bugs/noche, score golden set, cobertura por componente | 🟢 E13-EN-01 | KPI dashboard del sistema | — | Sin cambios |
| **—** | Orquestador overnight | Claude | Encadena las 11 skills en el bucle overnight con resiliencia ante fallos | 🔴 QAP-US-15 | Encadena generar→ADK→juez→cluster→hipótesis→fix→validar | ❓ Revisar | Ahora que existe plan-generator (paso 1), ¿sigue haciendo falta como skill separada o lo absorbe plan-generator? |
| **—** | qa-tc-analyzer | Claude | Evolución del QA actual de Petal hacia orquestador (distinto del overnight) | 🟢 QAP-US-11 | Evoluciona a orquestador completo | — | Sin cambios |
