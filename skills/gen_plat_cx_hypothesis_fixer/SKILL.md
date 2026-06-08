---
name: gen_plat_cx_hypothesis_fixer
version: 0.2
línea: GEN
scope: plat_cx
estado: 🔴 No existe
descripción: >
  Recibe la hipótesis validada por el comité (qué falta, dónde, por qué falla)
  y genera el cambio puntual necesario: qué sección modificar y cómo.
  No genera el playbook completo — eso es responsabilidad de act_plat_cx_fix (infra).
  No diagnostica ni razona sobre causas — implementa la dirección ya decidida.
kbs:
  - kb_ag_global
  - kb_plat_cx             # formato YAML de CX: estructura de playbooks, pasos, parámetros
  - kb_proj_<proyecto>     # estado actual del agente: solo la sección relevante, no el playbook completo
input:
  - hipótesis validada (qué cambiar, en qué sección, dirección de fix)
  - sección actual del playbook afectada (NO el playbook completo)
output:
  - cambio puntual estructurado:
      - ruta: instruction.steps[N]
      - contenido_nuevo: <texto de la instrucción modificada>
      - razon: <por qué este cambio implementa la hipótesis>
modelo_recomendado: Gemini Flash (Google AI Studio free tier)
razon_modelo: >
  Generación de código estructurado en el formato nativo de Gemini (YAML de CX).
  La hipótesis ya especifica QUÉ hacer — el fixer solo necesita escribirlo correctamente.
  Gemini conoce el formato de playbooks CX mejor que cualquier otro modelo.
  Coste $0 (free tier Google AI Studio).
  Riesgo de error sintáctico → se detecta en ADK re-test antes de tocar producción.
  Si el YAML generado falla consistentemente → escalar a Claude API.
restricciones:
  - SOLO modificar la sección especificada en la hipótesis
  - NO regenerar ni inferir el resto del playbook
  - El Full Update (enviar playbook completo a CX) es responsabilidad de act_plat_cx_fix
separacion_responsabilidades: >
  hypothesis-fixer  → QUÉ cambiar (cambio puntual, mínimo, con evidencia)
  act_plat_cx_fix   → CÓMO desplegarlo (GET playbook completo → aplica cambio → PATCH Full Update)
  Esta separación permite que hypothesis-fixer sea agnóstico al bug de europe-west1.
  Si en el futuro se cambia a us-central1, solo cambia act_plat_cx_fix.
---

# gen_plat_cx_hypothesis_fixer

> ⚠️ SKILL EN CONSTRUCCIÓN — este archivo es el placeholder. El contenido real se escribe al construir la skill.

## Cuándo invocar

Paso 8 del ciclo autónomo — después de hypothesis-generator, antes de ADK re-test.

## Flujo

## Coste

## DoD
