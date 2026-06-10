---
name: sys_b_writer
version: 0.1
línea: Sistema B
scope: ag
estado: 🔴 No existe
descripción: >
  Recibe los registros clasificados (de sys_b_classifier), detecta conflictos
  y duplicados contra el KB existente, genera borradores de entradas KB y
  empaqueta todo para la revisión humana (gate Jero). Solo escribe al KB
  tras aprobación explícita.
kbs:
  - kb_sys_core      # estructura y formato de entradas KB
  - kb_ag_global     # KB universal — lectura para detección de conflictos
  - kb_proj_<proyecto>  # KB de proyecto — lectura + escritura tras gate
input:
  - array de registros clasificados (JSON del sys_b_classifier)
  - contenido actual de los KBs relevantes (para detección de conflictos)
output:
  - paquete de revisión para Jero:
      entradas_nuevas: [{kb_destino, contenido_borrador, confianza}]
      conflictos: [{entrada_nueva, entrada_existente, descripcion_conflicto}]
      duplicados: [{entrada_nueva, entrada_existente, merge_propuesto}]
  - tras aprobación: entradas escritas en los archivos KB correspondientes
modelo_recomendado: claude-sonnet-4-6
razon_modelo: >
  Escritura estructurada y detección de conflictos/duplicados. El juicio
  semántico complejo ya ocurrió en sys_b_classifier. Aquí el trabajo es
  comparar, formatear y redactar — Sonnet es suficiente y más económico.
sistema_eval: escritura_kb
paso: 3  # tercera y última etapa de Sistema B
gate: >
  OBLIGATORIO antes de escribir al KB. sys_b_writer nunca escribe sin
  aprobación explícita de Jero. El paquete de revisión es la salida
  principal; la escritura es la salida secundaria condicionada al gate.
---

# sys_b_writer

> ⚠️ SKILL EN CONSTRUCCIÓN — este archivo es el placeholder. El contenido real se escribe al construir la skill.

## Cuándo invocar

Tercera etapa de Sistema B, después de sys_b_classifier.
Se invoca una vez por batch (no por registro individual).

## Flujo

```
registros clasificados (N)
       ↓
sys_b_writer
  ├── lee KBs existentes
  ├── detecta conflictos → flag inmediato
  ├── detecta duplicados → propone merge
  └── genera borradores de entradas nuevas
       ↓
paquete de revisión → Jero aprueba / descarta
       ↓
escritura al KB (solo si aprobado)
```

## Tipos de salida por registro

| Caso | Acción de sys_b_writer |
|---|---|
| Entrada nueva, sin conflicto | Genera borrador, lo incluye en paquete de revisión |
| Contradice entrada existente | Flag inmediato, incluye ambas entradas para que Jero decida |
| Duplicado de entrada existente | Propone merge, muestra diff entre versión actual y nueva |
| Confianza baja (≤0.5) | Incluye en paquete pero marcado como "requiere más ciclos" |

## Formato de entrada KB

Cada entrada escrita al KB sigue la estructura:

```markdown
## <título del patrón>

**Tipo:** patrón / anti-patrón / caso-límite
**Nivel de confianza:** alta / media / baja
**Origen:** Sistema B — ciclos <N> a <M> de <fecha>

<descripción del patrón en lenguaje natural>

**Aplicar cuando:** <contexto de uso>
**Evitar cuando:** <restricciones conocidas>
```

## Coste

## DoD
