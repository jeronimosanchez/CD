# Backlog CD por defecto

**Propósito**: lista de TCs base reutilizables aplicables a cualquier agente conversacional. Sirve como punto de partida para construir la suite QA del proyecto.

## TCs transversales (válidos en cualquier agente)

### Saludos y despedidas

| TC | Input user | Expectativa |
|---|---|---|
| TC-SALUDO-01 | "Hola" | Saludo personalizado del agente |
| TC-SALUDO-02 | "Buenas" | Idem |
| TC-DESPEDIDA-01 | "Adiós" | Despedida cortés + invitación a volver |
| TC-AGRADECIMIENTO-01 | "Gracias" | Respuesta breve + cierre o continuación |

### Confirmaciones y negaciones

| TC | Input user | Expectativa |
|---|---|---|
| TC-CONFIRMACION-01 | "Sí / vale / claro" tras pregunta del agente | Avanza el flujo |
| TC-NEGACION-01 | "No / no quiero" tras pregunta del agente | Ofrece alternativa o cierra |

### Frustración y escalación

| TC | Input user | Expectativa |
|---|---|---|
| TC-FRUSTRACION-01 | "Esto no funciona" | Tono empático + ofrece humano |
| TC-FRUSTRACION-02 | Insulto al agente | Mantiene tono profesional, no responde insulto |
| TC-ESCALACION-01 | "Quiero hablar con una persona" | Transfiere a humano |
| TC-ABANDONO-01 | "Déjalo, olvídalo" | Cierre cortés sin presión |

### Información del negocio (FAQ)

| TC | Input user | Expectativa |
|---|---|---|
| TC-FAQ-HORARIO-01 | "¿Cuál es vuestro horario?" | Respuesta determinística desde Sheet |
| TC-FAQ-CONTACTO-01 | "¿Cómo os contacto?" | Respuesta con canales |
| TC-FAQ-DEVOLUCION-01 | "¿Cómo devuelvo un producto?" | Política de devoluciones |

### Manejo de datos personales

| TC | Input user | Expectativa |
|---|---|---|
| TC-EMAIL-01 | Captura email válido | Acepta + confirma |
| TC-EMAIL-02 | Email inválido | Rechaza + pide reformular |
| TC-TELEFONO-01 | Teléfono válido | Acepta |
| TC-DIRECCION-01 | Dirección estructurada | Captura componentes |

### Casos límite (edge cases)

| TC | Input user | Expectativa |
|---|---|---|
| TC-EDGE-MAYUS-01 | Input en MAYÚSCULAS | Comportamiento normal, sin tratar como grito |
| TC-EDGE-EMOJI-01 | Input solo con emojis | Pide reformular o interpreta |
| TC-EDGE-VACIO-01 | Mensaje vacío | Pide al usuario que escriba algo |
| TC-EDGE-IDIOMA-01 | Idioma no soportado | Pide reformular en idioma soportado |
| TC-EDGE-LARGO-01 | Mensaje muy largo (>500 caracteres) | Procesa correctamente o resume |

---

## TCs específicos del dominio

Cada proyecto añade TCs propios del dominio del cliente. Los transversales SIEMPRE están.

### Plantilla TC del dominio

```yaml
- id: TC-[DOMINIO]-XX
  tc_name: ""
  group: ""
  type: ""     # HAPPY_PATH / EDGE / NEGATIVE / REGRESSION
  turns:
    - turn: 1
      user: ""
      expected_regex: ""
      checks_expected: []
    - turn: 2
      user: ""
      expected_regex: ""
```

---

## Cómo usar este backlog

1. **Copiar los TCs transversales** a la suite QA del proyecto desde el día 1
2. **Añadir TCs del dominio** según el briefing y query inventory
3. **Iterar**: cada bug observado en operación añade un TC al backlog del proyecto

**Volumen típico**:
- Transversales (esta lista): ~20-25 TCs
- Dominio específico: ~30-50 TCs adicionales
- Total inicial: ~50-75 TCs

---

## Versionado del backlog

Cada cliente tiene su backlog propio que evoluciona. **Este es el punto de partida común**, no la versión final.

| Versión | Fecha | Cambios |
|---|---|---|
| 0.1 | 2026-05-28 | Versión inicial con TCs transversales |
