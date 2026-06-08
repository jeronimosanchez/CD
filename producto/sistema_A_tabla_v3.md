# Sistema A — Tabla de skills y backlog v3
> Fecha: 2026-06-06
> 🟢 Se mantiene | 🔴 Se elimina/cambia | ⬜ No existe aún

| Paso | Descriptor | LLM | Qué hace | Backlog actual | Nuevo en backlog |
|---|---|---|---|---|---|
| **1** | **plan-generator** | Claude | Diseña qué testear cada noche, calcula N_conversaciones por hipótesis y componente | ⬜ No existe | ➕ QAP-US-18 |
| **2** | **adversarial** | Qwen local | Genera conversaciones difíciles para romper el sistema siguiendo el coverage plan | 🔴 GEN-US-01 (línea y descripción incorrectas) | ➕ QAP-US-19 |
| **2** | **Setup Ollama/Qwen** | — (infra) | Instala y configura Qwen en M4 | 🔴 GEN-US-02 (línea incorrecta) | 🔄 Mover a QAP |
| **2** | **Perfiles de usuario** | Qwen local | Tipos de usuario que simula el adversarial | 🔴 GEN-US-03 (línea incorrecta) | 🔄 Absorber en adversarial |
| **2** | **Outputs esperados** | Qwen local | Qué debería responder el agente por conversación | 🔴 GEN-US-04 (línea incorrecta) | 🔄 Absorber en adversarial |
| **3** | **ADK runner** | No LLM (runtime) | Ejecuta las conversaciones contra Petal en local ($0) | 🟢 ACT-US-06 | — |
| **4** | **cluster-analyzer** | Embeddings local | Agrupa FAILs por similitud semántica, detecta patrones | 🟢 QAP-US-01 | — |
| **5** | **hypothesis-generator** | Claude | Propone hipótesis por cluster con predicción de mejora, consulta log JSONL para no repetir | 🔴 QAP-US-05 (descripción incompleta) | 🔄 Actualizar descripción |
| **6** | **playbook-expert** | Claude | Valida hipótesis desde perspectiva del playbook (binario) | 🟢 QAP-US-02 | — |
| **6** | **inventory-expert** | Claude | Valida hipótesis desde perspectiva del inventario (binario) | 🟢 QAP-US-03 | — |
| **6** | **git-expert** | Claude | Valida si el fix ya se intentó antes (binario) | 🟢 QAP-US-04 | — |
| **7** | **ADK runner** | No LLM (runtime) | Re-ejecuta conversaciones con fix aplicado, compara PASS% antes/después | 🟢 ACT-US-06 (mismo runner) | — |
| **8** | **hypothesis-fixer** | Claude | Genera el artefacto concreto: el diff real del playbook | 🔴 QAP-US-06 (debe ser GEN, no QAP) | 🔄 Mover a GEN |
| **9** | **Juez LLM** | Claude | Evalúa si el fix mejora el golden set en staging sin regresiones | 🟢 QAP-US-14 | — |
| **9** | **Golden set** | Gemini (staging CX) | 50 conversaciones validadas que son la referencia de qué es correcto | 🔴 RES-US-03 + E13-US-01 (duplicados) | 🔄 Fusionar en E13-US-01 |
| **9** | **coverage-tracker** | No LLM | Matriz histórica de cobertura por componente: tipos, runs, fecha | ⬜ No existe | ➕ QAP-US-20 |
| **10** | **CI/CD deploy** | No LLM (GitHub Actions) | Despliega a producción los fixes validados en staging | 🟢 ACT pipeline | — |
| **11** | **Log JSONL** | No LLM | Registro append-only de hipótesis y resultados de cada ciclo | 🔴 SYS-EN-03 + E13-US-03 (duplicados) | 🔄 Fusionar en E13-US-03 |
| **11** | **Dashboard madurez** | No LLM | Curva bugs/noche, score golden set, cobertura por componente | 🟢 E13-EN-01 | — |
| **—** | **Orquestador overnight** | Claude | Coordina los 11 pasos del ciclo completo | 🔴 QAP-US-15 (genérico, ahora distribuido en pasos específicos) | ❓ Revisar si hace falta |
| **—** | **qa-tc-analyzer → orquestador** | Claude | Evolución del QA de Petal (distinto del overnight) | 🟢 QAP-US-11 | — |
