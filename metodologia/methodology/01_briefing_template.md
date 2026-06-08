# 01 — Briefing methodology

## Propósito

Capturar la información mínima necesaria para empezar un proyecto CD sin asumir ni inventar.

## Cuándo usarlo

Antes de cualquier mapeo de queries o decisión arquitectónica. Es el paso 0.

## Información obligatoria a capturar

### Negocio

- **Empresa / cliente**: nombre, sector, escala
- **Objetivo del agente**: ¿qué problema resuelve?
- **KPIs medibles**: tasa de éxito, tiempo de resolución, conversión, ahorro de coste
- **Restricciones de compliance**: GDPR, sectoriales, datos sensibles
- **Plazo estimado**: cuándo tiene que estar en producción

### Usuarios

- **Perfil tipo del usuario final**: edad, contexto, expectativas
- **Volumen esperado**: conversaciones/día, picos
- **Canales**: web, móvil, voz, WhatsApp, etc.
- **Idiomas**: monolingüe, multilingüe, dialectos

### Técnico

- **Plataforma destino**: CX, Lex, Voiceflow, custom
- **Backend existente**: APIs, bases de datos, integraciones
- **Stack del cliente**: GCP, AWS, Azure, mixto
- **Equipo disponible**: roles, seniority, dedicación

### Producto

- **MVP scope**: qué entra en la primera versión
- **Roadmap visible**: qué viene después
- **Casos de uso priorizados**: lista ordenada
- **Edge cases conocidos**: cosas que ya saben que pueden romperse

## Output del briefing

Documento `briefing_form.md` rellenado (ver `templates/`).

Sirve como contrato entre stakeholders. Si algo no está en el briefing, no se asume — se pregunta.

## Honestidad

El briefing perfecto no existe. **Un briefing del 70% es suficiente para arrancar**. Los huecos se llenan con preguntas durante la fase de Query Analysis.

## Anti-patrón común

Saltarse el briefing porque "el cliente ya lo explicó en la reunión". Esto multiplica reprocesos x3. **Documenta o paga después**.
