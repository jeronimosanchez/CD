# Sistema A — Tabla de skills y backlog
> Fecha: 2026-06-06
> 🟢 Se mantiene | 🔴 Se elimina/cambia | ⬜ No existe aún

| Descriptor | LLM | Qué hace | Backlog actual | Nuevo en backlog |
|---|---|---|---|---|
| **plan-generator** | Claude | Diseña cada noche qué testear, cuántas conversaciones por hipótesis y componente usando la fórmula N_conversaciones | ⬜ No existe | ➕ Crear QAP-US-18 |
| **adversarial** | Qwen/Ollama local | Genera conversaciones difíciles diseñadas para romper el sistema siguiendo el coverage plan | 🔴 GEN-US-01 (línea incorrecta, descripción incompleta) | ➕ Crear QAP-US-19 con Qwen local explícito |
| **Setup Ollama/Qwen** | — (infraestructura) | Instala y configura Qwen en local (M4) | 🔴 GEN-US-02 (línea incorrecta) | 🔄 Mover a QAP |
| **Perfiles de usuario** | Qwen local | Define los tipos de usuario que el adversarial simula | 🔴 GEN-US-03 (línea incorrecta) | 🔄 Absorber en QAP adversarial |
| **Outputs esperados** | Qwen local | Define qué debería responder el agente para cada conversación | 🔴 GEN-US-04 (línea incorrecta) | 🔄 Absorber en QAP adversarial |
| **ADK runner** | No LLM (runtime) | Ejecuta las conversaciones contra Petal en local ($0) | 🟢 ACT-US-06 | — |
| **cluster-analyzer** | Embeddings local | Agrupa los FAILs por similitud semántica, detecta patrones | 🟢 QAP-US-01 | — |
| **hypothesis-generator** | Claude | Propone hipótesis por cluster con predicción de mejora, consulta log para no repetir | 🔴 QAP-US-05 (descripción incompleta, falta fórmula y log JSONL) | 🔄 Actualizar descripción |
| **playbook-expert** | Claude | Valida hipótesis desde perspectiva del playbook (binario) | 🟢 QAP-US-02 | — |
| **inventory-expert** | Claude | Valida hipótesis desde perspectiva del inventario (binario) | 🟢 QAP-US-03 | — |
| **git-expert** | Claude | Valida si el fix ya se intentó antes (binario) | 🟢 QAP-US-04 | — |
| **hypothesis-fixer** | Claude | Genera el artefacto concreto: el diff real del playbook | 🔴 QAP-US-06 (línea incorrecta, debe ser GEN) | 🔄 Mover a GEN |
| **Juez LLM** | Claude | Evalúa si el fix mejora el golden set en staging sin regresiones | 🟢 QAP-US-14 | — |
| **coverage-tracker** | No LLM | Matriz histórica de cobertura por componente: cuántas conversaciones, qué tipos, cuándo | ⬜ No existe | ➕ Crear QAP-US-20 |
| **Golden set** | Gemini (staging CX) | 50 conversaciones validadas que son la referencia de qué es correcto | 🔴 RES-US-03 + E13-US-01 (duplicados) | 🔄 Fusionar en E13-US-01 |
| **Log JSONL** | No LLM | Registro append-only de todas las hipótesis probadas y sus resultados | 🔴 SYS-EN-03 + E13-US-03 (duplicados) | 🔄 Fusionar en E13-US-03 |
| **CI/CD deploy** | No LLM (GitHub Actions) | Despliega a producción los fixes validados en staging | 🟢 ACT pipeline | — |
| **Dashboard madurez** | No LLM | Muestra curva de bugs/noche, score golden set, cobertura por componente | 🟢 E13-EN-01 | — |
| **Orquestador overnight** | Claude | Coordina los 11 pasos del ciclo completo | 🔴 QAP-US-15 (demasiado genérico, ahora distribuido en pasos específicos) | ❓ Revisar si hace falta o lo absorbe plan-generator |
| **qa-tc-analyzer → orquestador** | Claude | Evolución del QA actual de Petal hacia orquestador (distinto del overnight) | 🟢 QAP-US-11 | — |
