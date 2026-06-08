# Briefing — Petal (florería digital)

**Cliente**: Floristería Petal (caso piloto / simulación)
**Fecha briefing**: 2026-Q1 (retroactivo, derivado del proyecto real)
**Stakeholders presentes**: Jero (founder + builder)
**Versión**: 1.0

---

## Negocio

- **Empresa / cliente**: Floristería Petal — simulación profesional con calidad de producción real
- **Sector**: Retail / comercio electrónico de flores
- **Escala**: 1 floristería online ficticia, simulación de proyecto real
- **Objetivo del agente**: asistente conversacional de ventas que ayuda a clientes a comprar flores, gestionar pedidos y consultar información
- **KPIs medibles**:
  - Tasa de conversión conversación → compra
  - Tiempo medio de cierre de venta
  - % conversaciones con escalación a humano
  - Tasa de satisfacción (proxy via outcomes)
- **Restricciones de compliance**: GDPR (datos cliente: nombre, email, dirección)
- **Plazo estimado de go-live**: ya en producción
- **Presupuesto disponible**: bootstrap (créditos GCP existentes)

---

## Usuarios

- **Perfil tipo del usuario final**:
  - Edad: 25-65
  - Contexto: comprando flores para regalo, evento, decoración propia
  - Expectativas: rapidez, recomendación útil, no presión comercial
  - Nivel técnico: bajo a medio
- **Volumen esperado**: bajo (simulación), pero diseño para escalar
- **Canales**: chat web (Dialogflow CX)
- **Idiomas**:
  - Principal: español
  - Secundarios: ninguno por ahora

---

## Técnico

- **Plataforma destino**: Dialogflow CX (europe-west1)
- **Backend existente**:
  - APIs: petal-sheet-api en Cloud Run
  - Bases de datos: Google Sheets (inventario, business, agent_copy, perfil, pedidos)
  - Integraciones: WIF (GitHub Actions → GCP)
- **Stack del cliente**: GCP exclusivo
- **Equipo disponible**: 1 persona (Jero) + Claude Code como asistente
- **Restricciones técnicas conocidas**:
  - Bug regional en europe-west1: Playbooks requieren Full Update
  - Concept drift en KB cuando se refactorizan playbooks

---

## Producto

- **MVP scope**: 6 playbooks (Orquestador, Compra, Checkout, Registro, Handoff, Gestión Deuda)
- **Roadmap visible**:
  - Refactor de Compra a sub-Tasks (deuda técnica conocida)
  - Petal 2 con arquitectura híbrida NLU+LLM 80/20 (opcional)
  - QAP épicas 01-07
- **Casos de uso priorizados**:
  1. Compra de ramo de flores con refinamiento
  2. Multi-producto (varios items en carrito)
  3. Gestión de cliente nuevo (registro) vs recurrente
  4. Manejo de objeciones y abandono
  5. Información del negocio (horarios, zonas, devoluciones)
- **Edge cases conocidos**:
  - TC-URGENCIA: entregas con restricción temporal
  - TC-MULTI-PRODUCTO: ECO RESUMEN con total
  - TC-CAMBIO-OP: abandono mid-flow
  - TC-FRUSTRACION: manejo emocional
- **Casos explícitamente fuera de scope**:
  - Pagos reales
  - Logística real (envíos físicos)
  - Voz / TTS

---

## Riesgos identificados

- **Técnicos**:
  - Compra está saturado (~12.5K tokens) → bugs LLM por context
  - Variabilidad LLM en queries del inventario
  - Concept drift cuando refactorizamos
- **De negocio**:
  - Es portfolio, no cliente real → puede no escalar a producción real
- **De cliente**: N/A (no hay cliente externo)

---

## Decisiones de arquitectura preliminares

- **NLU vs LLM**: actualmente 90% LLM (Playbooks dominantes)
- **Hosting decision**: GCP (Dialogflow CX + Cloud Run + Sheets)
- **Modelo LLM elegido**: Gemini 2.5 Flash (default de Dialogflow CX)

---

## Próximos pasos

- [x] Query Analysis retrospectivo
- [ ] Layer Assignment (si arrancamos Petal 2)
- [ ] Arquitectura derivada (Petal 2 si aplica)
- [ ] QAP cubre validación continua

---

## Notas / comentarios libres

Petal sirve como **caso piloto del sistema completo**. Sus aprendizajes alimentan la CD Templates Library para futuros clientes (Farma-ECH, Globant clientes, otros).

El proyecto está sobre-documentado intencionalmente porque es la **base del conocimiento** del sistema. Cada decisión y cada bug analizado enriquece el KB que beneficiará a proyectos futuros.

---

## Decisiones aprendidas (retrospectivamente)

| Decisión inicial | Aprendizaje | Cómo se aplicará en futuros proyectos |
|---|---|---|
| 90% LLM Playbooks | Caro y con varianza | Distribuir 70/30 al menos |
| Compra monolítico | Bugs por context window | Empezar con sub-Tasks desde día 1 |
| Sin KB persistente | Cada análisis de cero | EP-SKILL-01 obligatorio en mes 1 |
| Sin closed-loop | Recomendaciones no validadas | EP-SKILL-02.1 obligatorio en mes 2 |
