# Brief — Petal 1.1

**Fecha:** 2026-06-04
**Versión:** 1.1
**Método:** entrevista de discovery con Carmen Ruiz (simulación) → deltas extraídos respecto a 1.0
**CD responsable:** Jerónimo Sánchez

---

## 1. Contexto del negocio (delta de 1.0)

| Dimensión | 1.0 | 1.1 |
|---|---|---|
| Ciudades | 1 (Madrid) | 5: Madrid, Sevilla, Valencia, Barcelona, Bilbao |
| Tiendas | 1 | 15 (3 por ciudad: 1 centro + 2 periferia) |
| Cobertura | única | por zona — centro da servicio al centro, periferia a sus áreas |
| Problemas nuevos | — | (a) detección de tipo de producto falla · (b) gestión de ciudades/zonas |

---

## 2. Política de entrega (nueva, 2 modelos)

### Las 2 políticas

| Política | Aplica a | Plazo | Corte | Precio envío |
|---|---|---|---|---|
| **CENTRO** | Madrid-centro · Barcelona-centro · TODO Sevilla/Valencia/Bilbao | Mismo día | Antes de 14:00 | 6€ |
| **PERIFERIA** | Madrid-periferia · Barcelona-periferia únicamente | Día anterior | Mañana → antes 14:00 · Tarde → antes 19:00 | 7€ |

### Restricciones
- Entrega siempre hasta las 19:00 (límite de entrega hoy).
- Periferia: quien llame a las 19:00 para mañana por la mañana → NO es posible.
- Sevilla, Valencia y Bilbao tienen capacidad física de cubrir periferia, pero **la política comercial que aplican es CENTRO** (un solo modelo para las 3 ciudades).

### Clasificador de zona
- **Por código postal.** Lista de CPs "centro" de Madrid y de Barcelona en el Sheet.
- **Cuándo se captura:** dentro de la dirección de checkout, nunca como pregunta de apertura (*lazy slot-filling*).
- **Si el usuario pregunta plazo antes del checkout:** el agente pregunta solo la ciudad → da la política aplicable. Si es Madrid/Barcelona, resuelve zona cuando llegue la dirección.
- **Sin auto-detección de ciudad en 1.1** (geo/IP se difiere a 2.0).

### Criterios de aceptación
- El agente nunca pide código postal de apertura.
- Si el usuario pregunta plazo → el agente pregunta solo la ciudad (no el CP).
- Ciudad fuera de las 5 → informa que no hay servicio y lista las 5 ciudades con cobertura.
- CP en la lista de centro → aplica política CENTRO.
- CP no en la lista → aplica política PERIFERIA (solo relevante en Madrid y Barcelona).
- Combinación imposible (ej. 19:00 + mañana en periferia) → el agente lo bloquea e informa.

---

## 3. Catálogo — 2 ejes (nuevo modelo)

### Estructura
- **Eje 1 — Tipo:** ramos, centros, coronas (funeral), complementos.
- **Eje 2 — Ocasión:** funeral, cumpleaños, San Valentín, día de la madre, boda, evento corporativo.
- El catálogo en el Sheet debe estar indexado por ambos ejes para que la recomendación por ocasión funcione.

### Problema identificado
El catálogo actual no tiene las características suficientes para que el agente detecte bien el tipo de producto cuando el usuario es difuso. Auditoría del Sheet pendiente antes de construir 1.1.

---

## 4. Tipos de cliente — matriz contexto × modo

### 3 contextos (quién pide y para qué)
| Contexto | Lo que determina |
|---|---|
| **Particular** | Catálogo estándar · Tono estándar o solemne según ocasión |
| **Empresa** | Tarifa pactada (vía email) · Tono corporativo |
| **Funeral** | Catálogo fúnebre (coronas, centros) · Tono solemne siempre |

Nota: Empresa y Funeral no son excluyentes. Regla de composición: **quién paga** define precio (Particular/Empresa), **la ocasión funeral** define tono y productos.

### 2 modos (cómo decide)
| Modo | Señal | Flujo del agente |
|---|---|---|
| **Concreto** | Sabe qué quiere ("quiero rosas rojas") | Detección directa → confirmación → checkout |
| **Difuso** | Necesita asesoramiento ("algo para mi madre") | Orientación guiada → ancla primaria flor → ancla de respaldo ocasión |

### La matriz (6 perfiles de atención)
|  | Concreto | Difuso |
|---|---|---|
| **Particular** | detección rápida | asesoramiento guiado |
| **Empresa** | rápido + tarifa | asesoramiento + tarifa |
| **Funeral** | corona/centro directo | orientación solemne |

### Flujo de orientación (cliente difuso)
El agente **no impone un orden**. Escucha la primera señal del cliente y trabaja desde ahí:
1. Si da **tipo de flor** (lo más común) → trabaja desde la flor.
2. Si no sabe la flor → pregunta por la **ocasión** → recomienda.
3. Nunca suelta el catálogo completo ni aplica un cuestionario rígido.

### Criterios de aceptación
- Cliente que dice "quiero rosas rojas" → no recibe el cuestionario de asesoramiento.
- Cliente que dice "algo para mi madre" → recibe orientación (flor u ocasión), no un volcado de catálogo.
- Transición de modo (decidido → indeciso a mitad de conversación) → el agente se adapta.

---

## 5. Reconocimiento de empresa y tarifa

### Mecanismo (POC)
- **Por defecto:** todos los clientes son Particular.
- **Disparador:** el cliente declara ser empresa ("soy de ACME").
- **Verificación:** el agente solicita el email → lo cruza contra tabla `cuentas_empresa` en el Sheet.
- **Si consta:** aplica tarifa pactada de esa empresa.
- **Si no consta:** precio de catálogo estándar. Autodeclarar ser empresa no da tarifa.

### Captura del email
Dentro del checkout (donde ya se pide para confirmación/factura). Si el cliente se declara empresa antes, el agente puede pedir el email en ese momento para verificar.

### Historial de cliente (corrección respecto a 1.0)
El historial **sí existe** en el Sheet (`perfil` + `pedidos`). Se accede vía lookup por email en el checkout.
Un email → el agente sabe 3 cosas: (1) si es cliente previo, (2) qué pidió antes, (3) si es empresa con tarifa.

### Diferido a 2.0
- Identificación automática/silenciosa (login, sesión).
- Historial personalizado proactivo ("la última vez pediste X").
- Verificación passwordless (magic link / OTP) para post-venta.

### Criterios de aceptación
- Sin declaración de empresa → tratado como Particular.
- Declara empresa + email en Sheet → tarifa empresa.
- Declara empresa + email NO consta → precio estándar.
- El email debe estar en el checkout de todas formas (factura/confirmación).

---

## 6. Post-venta / estado del pedido

**Fuera de 1.1.** Diferido a 2.0.

Nota de diseño: cuando se implemente, el flujo es NLU (intent claro de alto volumen) + lookup por email + verificación de propiedad del pedido. No requiere login completo — verificación ligera (OTP/magic link) es suficiente.

---

## 7. KPIs de negocio + proxy en el POC

### KPIs de negocio (capa durable — se activan con cliente real)
1. **% consultas resueltas sin intervención humana** (autonomía del agente).
2. **Pedidos capturados fuera de horario** ← la métrica más importante. Recupera demanda que hoy se pierde.
3. **Menos llamadas a la tienda.**
4. **Más pedidos cerrados por el agente.**

> Reframe clave: Petal no es un ahorro de costes — es **captura de ingresos**. Un pedido a las 9am que hoy se evapora, mañana lo cierra el agente.

### Proxy en el POC (sin cliente real)
| KPI de negocio | Proxy |
|---|---|
| % consultas resueltas | % QA pass de TCs de resolución autónoma |
| Pedidos fuera de horario | TCs de urgencia/temporal verdes + escenario demo E2E |
| Más pedidos cerrados | TCs de checkout completo verdes |
| Menos llamadas | Sin proxy limpio — cualitativo en el POC |

---

## 8. Diferido a 2.0 (no entra en 1.1)

- Auto-detección de ciudad (geo/IP).
- Identificación automática de cliente (login/sesión).
- Historial personalizado proactivo.
- Post-venta / tracking de pedido.
- Verificación passwordless (magic link/OTP).
- Arquitectura NLU completa para intents de alto volumen (clasificación automática).

---

## 9. Impacto arquitectónico en 1.1

### Por qué NLU+LLM (justificación de negocio, no técnica)
Los 3 casos de uso que piden NLU emergen directamente de este brief:
1. **Cliente decidido** — intent claro, alto volumen, cero ambigüedad.
2. **Tracking de pedido** (2.0) — intent trivial, lookup, sin conversación.
3. **Detección de ciudad** — clasificación determinista por CP.

El LLM se reserva para:
- Asesoramiento al cliente difuso.
- Gestión de frustración y empatía.
- Slot-filling complejo (combinación flor + ocasión + presupuesto).
- Casos edge y transiciones de modo.

### Arquitectura final decidida (2026-06-04)

**El Orquestador desaparece.** Lo sustituye el NLU layer.

```
HOY (1.0)                        1.1
Usuario → Orquestador (LLM)      Usuario → NLU (Flows + Intents)
          clasifica G1-G5                   detecta contexto + modo
          4.2k tokens                       0 tokens
          → Compra (todo)                   → fast path si CONCRETO
                                            → Compra si DIFUSO
```

**Las 6 modalidades y su router:**

| Contexto | Modo | Handler |
|---|---|---|
| Particular + Concreto | "quiero rosas rojas" | NLU → ConsultaInventario_Task → Checkout |
| Particular + Difuso | "algo para mi madre" | NLU → Compra |
| Empresa + Concreto | "200 centros para el viernes" | NLU → fast path + tarifa empresa |
| Empresa + Difuso | "algo para un evento" | NLU → Compra |
| Funeral + Concreto | "quiero una corona" | NLU → fast path + tono solemne |
| Funeral + Difuso | "algo para un tanatorio" | NLU → Compra + tono solemne |

Compra pasa de 11.4k → ~6k tokens porque solo gestiona casos DIFUSOS.

### Plan de ejecución (orden fijo)

```
1. Staging
2. Task 0 — Inventario Sheet (tipo, ocasión, descripción, url_foto)
3. Task 1 — ConsultaInventario_Task
4. Task 2 — GestionFiltros_Task
5. Task 3 — VerificacionEntrega_Task
6. Task 4 — DeteccionContexto_Task
7. NLU — reemplaza Orquestador + router de las 6 modalidades
8. Compra — refactor (ahora solo casos difusos)
9. TCs nuevos (~15) — validar las 6 modalidades
10. Rich cards
11. Suggestion chips
```

**Fuera de 1.1:** zona entrega CP, seguridad backend, auto-detección ciudad.

---

## 10. Épicas técnicas del backlog

### Épicas heredadas de 1.0 (documentadas en `petal_1_1.md`)

| # | Épica | Tiempo | Orden |
|---|---|---|---|
| **0** | **Staging** — Environment CX + Sheet ficticia + flag sandbox + compare_envs.py | ~3h | PRIMERO — bloqueante |
| **1** | **Consistencia** — auditoría 9 criterios × 6 playbooks (baseline: 0✅/27⚠️/3❌) | ~2 días | paralela con Épica 2 |
| **2A** | **Arquitectura Tasks** — extraer ConsultaInventario + GestionFiltros de Compra (785 líneas → ~400) | ~3 días | paralela con Épica 1 |
| **2B** | **Arquitectura NLU+LLM** — Flows + Intents para clasificación; LLM para conversación compleja | ~1 semana | alternativa a 2A |
| **3** | **Comparación + Promoción** — gate de 7 pasos a producción con KPIs de negocio | ~0.5 días | última |

### Tasks de arquitectura (decisión 2026-06-04)

Arquitectura confirmada: **Épica 2B — NLU+LLM híbrido**. Las Tasks redistribuyen responsabilidades antes de introducir el layer NLU.

| Task | Qué extrae | Quita de | Tokens ahorrados |
|---|---|---|---|
| **Task 0** — Auditoría inventario Sheet | Añade campos: tipo, ocasion, descripcion, url_foto, tags | Sheet (prerrequisito de datos) | — |
| **Task 1** — ConsultaInventario_Task | Todas las llamadas a PetalDataTool para buscar productos | Compra | ~2.5k |
| **Task 2** — GestionFiltros_Task | Bucle de refinamiento cuando el usuario rechaza opciones | Compra | ~2k |
| **Task 3** — VerificacionEntrega_Task | Lógica de restricción temporal (casos A-E). Compartida entre Orquestador y Compra | Orquestador | ~80 líneas |
| **Task 4** — DeteccionContexto_Task | Detecta contexto (particular/empresa/funeral) y modo (concreto/difuso) | Orquestador PASO 0 | ~1k |

**Orden de ejecución:** Task 0 → Task 1+2 (paralelo) → Task 3+4 (paralelo) → NLU Layer.

**Resultado esperado:** Compra 11.4k → ~6k tokens (-47%) · Orquestador 4.2k → ~3k (-30%).

### Deuda técnica conocida

- **7 bloques EJEMPLO inline** (~1.5k tokens, 13%) pendientes de mover a Examples de CX.
- **push_playbooks.py** — reconciliar con §3.8 (Full Update obligatorio en europe-west1).
- **Zona entrega centro/periferia** — diferida. En 1.1 todos los clientes usan política CENTRO.

### Pendientes de sesiones anteriores

- **TCs URGENCIA** (9 ejecuciones finales) — feature/urgencia-temporal pendiente de validación final.
- **Inyección temporal en producción** — Flow de entrada + Start Page webhook. Endpoint `/temporal-context` ya desplegado y validado. Diseño cerrado.

### Épicas de UX/interfaz (nuevas en 1.1)

#### Épica UX-1 — Rich product cards
**Decisión (2026-06-04):** se implementa DESPUÉS de las Tasks y el layer NLU. Depende de la arquitectura estable.
**Estado:** diseño hi-fi **completo** (`~/Downloads/design_handoff_product_cards/`, 31-may-2026).
Componentes: `ProductCardA`, `ProductDetailPanel`, `CartAddedBubble`, `PetalHeader`. Prototipo standalone navegable incluido.

**Lo que falta:**
- Portar al widget real de CX (hoy son prototipos React/Babel, no producción).
- Campo `url_foto` en el Sheet por producto (por ahora placeholder — el README del handoff lo contempla explícitamente: solo sustituir placeholder por `<img src={imageUrl}>`).
- El plazo de entrega en la card ("Llega mañana" / "Llega en 2 días") debe derivarse de la política real (§2) — conecta con el clasificador CP + ciudad.

**Modelo de datos mínimo (Sheet → card):**
```
id · name · size · detail · price · shipping · url_foto (nullable)
```

**Criterios de aceptación:**
- Petal muestra los 3 productos recomendados como cards, no como texto plano.
- Sin `url_foto` → placeholder visual (no texto, no error).
- Tocar una card → abre panel de detalle.
- Botón `+` → feedback "añadido al carrito ✓" en el chat + badge contador.
- El plazo en la card es coherente con la política de entrega real (§2).

#### Épica UX-2 — Suggestion chips (botones de selección)
**Estado:** no implementado. Feature nativa de CX (custom payload).

Cuando Petal ofrece opciones discretas, muestra **botones pulsables** encima del compositor en vez de esperar texto del usuario.

**Casos de uso principales:**
- Tramo de entrega periferia: `[Mañana]` `[Tarde]`
- Confirmación de pedido: `[Confirmar]` `[Modificar]`
- Ocasión (cliente difuso): `[Cumpleaños]` `[Aniversario]` `[Sin ocasión]`
- Ciudad: `[Madrid]` `[Barcelona]` `[Otra]`

**Criterios de aceptación:**
- Opciones discretas (≤4, mutuamente excluyentes) → chips.
- El usuario puede pulsar el chip O escribir — ambas vías funcionan.
- Slot-filling abierto (espacio de respuesta amplio) → sin chips.

### Seguridad (pendiente, crítico antes de cliente real)

- **`petal-sheet-api`** — Cloud Run público sin autenticación. Cerrar antes de exponer a cliente real. No bloquea el POC pero es deuda crítica de 1.1.
