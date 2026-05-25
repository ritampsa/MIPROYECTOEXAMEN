# ADR-0002: Mecanismo de comunicación entre servicios (IPC)

**Status**: Accepted

**Fecha**: 2026-05-24

## Contexto

Tras decidir la descomposición en microservicios por capacidad de negocio (ADR-0001), es necesario definir el **mecanismo de comunicación entre servicios (IPC)**. La decisión impacta directamente en la latencia percibida, la tolerancia a fallos, la consistencia de datos y la complejidad operativa.

Restricciones que esta decisión debe respetar:
- Latencia <200 ms p95 en acciones del consumidor [Brief §A.4 Latencia UX] → el IPC del flujo crítico (Order Taking → Payment) debe ser rápido.
- Tolerancia a fallos externos: el sistema debe tomar pedidos aunque Stripe esté caído [Brief §A.4 Tolerancia a fallos] → requiere asincronía/colas para pagos.
- Consistencia de datos: fuerte consistencia dentro del aggregate de un pedido; eventual consistency aceptada para reporting [Brief §A.4 Consistencia].
- Trazabilidad end-to-end con correlation ID [Brief §A.4 Trazabilidad].
- Migración incremental Strangler Fig [Brief §A.4 Migración].

## Opciones consideradas

### Opción 1: REST síncrono (JSON/HTTPS) para todo

**Descripción**: todos los servicios se comunican mediante APIs REST síncronas. Cada llamada espera la respuesta del servicio destino.

**Pros**:
- Simple de implementar y depurar (petición/respuesta familiar).
- Bueno para consultas de datos en tiempo real (ej. obtener menú del restaurante).
- Compatible con el enfoque Strangler Fig (el monolito existente ya expone algunas APIs REST).
- Fácil de instrumentar con correlation ID en headers HTTP.

**Contras**:
- Acoplamiento temporal: si un servicio no responde, toda la cadena falla → viola NFR-04 (tolerancia a fallos externos).
- Latencia aumenta con la profundidad de la cadena de llamadas → riesgo para NFR-02.
- Dificultad para coordinar transacciones distribuidas (ej. pagar y crear pedido atómicamente).
- No es adecuado para notificaciones o eventos (push).

**Impacto en NFRs**: NFR-02 ± (bueno para consultas simples, malo para cadenas largas), NFR-04 ✗ (sin tolerancia a fallos), NFR-06 ✗ (difícil consistencia eventual).

### Opción 2: Eventos asíncronos (Kafka / RabbitMQ) para todo

**Descripción**: todos los servicios se comunican exclusivamente mediante mensajería asíncrona. Cada servicio publica eventos y consume eventos de otros servicios.

**Pros**:
- Alto desacoplamiento: los servicios no necesitan conocerse entre sí.
- Tolerancia a fallos: si un servicio está caído, los eventos se encolan y procesan cuando recupera → cubre NFR-04.
- Escalabilidad natural: los consumidores pueden escalar independientemente según la carga de eventos.
- Trazabilidad nativa con correlation ID en los headers del mensaje.

**Contras**:
- Latencia adicional por el broker (serialización, persistencia, round-trip) → riesgo para NFR-02 (<200 ms p95).
- Complejidad operativa: hay que manejar brokers, particiones, rebalancing, dead-letter queues.
- Dificultad para implementar consultas síncronas (ej. "dame el menú del restaurante") → requiere CQRS o API separada.
- Mayor complejidad de testing y debugging (eventos no deterministas).

**Impacto en NFRs**: NFR-02 ✗ (latencia extra del broker), NFR-04 ✓ (cola de retry), NFR-06 ✓ (eventual consistency nativa), NFR-07 ✓.

### Opción 3: Híbrido (REST para consultas síncronas + Eventos para comandos y notificaciones)

**Descripción**: se utiliza REST (JSON/HTTPS) para el flujo síncrono crítico del consumidor (seleccionar menú, confirming pedido) y eventos asíncronos (Kafka) para comunicar cambios de estado entre servicios (OrderCreated, PaymentProcessed, DeliveryAssigned), notificaciones y coordinación de procesos largos.

**Pros**:
- Balance entre latencia y desacoplamiento: el flujo crítico usa REST rápido (<200 ms p95), mientras que la propagación de estados usa eventos → cubre NFR-02 y NFR-04.
- Tolerancia a fallos parcial: el pago puede encolarse si Stripe falla mientras el pedido se confirma.
- Consistencia: fuerte consistencia en el aggregate del pedido (REST), eventual para reportes (eventos) → cubre NFR-06.
- Trazabilidad: correlation ID viaja en headers HTTP y en headers de Kafka → cubre NFR-07.
- Patrón recomendado por Richardson [Cap. 3] para marketplaces de delivery.
- Compatible con Strangler Fig: el monolito existente puede consumir eventos sin modificaciones mayores.

**Contras**:
- Dos mecanismos de comunicación = más complejidad operativa y de testing.
- Riesgo de inconsistencia si un evento se pierde (requiere exactly-once delivery o idempotencia).
- Mayor esfuerzo de diseño: hay que decidir qué va por REST y qué por eventos.

**Impacto en NFRs**: NFR-02 ✓ (REST para flujo crítico), NFR-04 ✓ (eventos para tolerancia), NFR-05 ✓, NFR-06 ✓, NFR-07 ✓.

### Opción 4: gRPC (HTTP/2, Protocol Buffers)

**Descripción**: usar gRPC como mecanismo de comunicación síncrono, con contratos definidos en .proto, streaming bidireccional y serialización binaria eficiente. Posible combinación con eventos para asincronía.

**Pros**:
- Alto rendimiento y baja latencia (Protocol Buffers + HTTP/2) → excelente para NFR-02.
- Streaming bidireccional nativo (útil para tracking en tiempo real).
- Contratos fuertes (código generado) que evitan errores de serialización.

**Contras**:
- Curva de aprendizaje alta para el equipo (acostumbrado a REST/JSON).
- ecosistema menos maduro para browser/mobile (requiere proxies como grpc-web).
- Los contratos .proto agregan fricción al cambio rápido durante la migración Strangler Fig.
- No es nativo en Java/Spring Boot tanto como REST (requiere dependencias adicionales).
- Sigue teniendo el problema de acoplamiento temporal (como REST) para la tolerancia a fallos.

**Impacto en NFRs**: NFR-02 ✓✓ (muy rápido), NFR-04 ✗ (mismo problema de acoplamiento temporal que REST), NFR-09 ? (no es stack preferido Java/Spring).

## Decisión

**Se elige la Opción 3: Híbrido (REST para consultas síncronas + Eventos asíncronos con Kafka para comandos y notificaciones).**

Justificación:
1. **Latencia**: el flujo crítico del consumidor (ver menú, crear pedido) se mantiene en REST síncrono para cumplir NFR-02 (<200 ms p95).
2. **Tolerancia a fallos**: Stripe caído? El cobro se encola como evento en Kafka (retry) mientras el pedido se confirma → cubre NFR-04.
3. **Consistencia**: el aggregate del pedido se maneja con fuerte consistencia vía REST; la propagación a otros servicios (cocina, delivery, billing) es eventual vía eventos → cubre NFR-06.
4. **Trazabilidad**: el correlation ID se inyecta en el request REST y se propaga a los eventos de Kafka → cubre NFR-07.
5. **Alineamiento con el libro**: Richardson [Cap. 3] recomienda este patrón híbrido para evitar los extremos de REST-puro (acoplamiento) y eventos-puro (latencia).
6. **Strangler Fig**: el monolito actual expone APIs REST y puede empezar a consumir/publicar eventos de Kafka sin reescribirse completamente.

## Consecuencias

### Positivas
- El flujo crítico del consumidor mantiene latencia <200 ms p95.
- Tolerancia a fallos delegada a eventos asíncronos (colas de retry).
- Consistencia adecuada al contexto: fuerte para el pedido, eventual para el resto.
- Trazabilidad end-to-end con correlation ID.
- El equipo puede empezar con REST (conocido) e incorporar Kafka gradualmente.

### Negativas
- Dos paradigmas de comunicación que el equipo debe dominar (REST + eventos).
- Se requiere infraestructura Kafka (broker, topics, schemas) desde fases tempranas.
- Diseño más cuidadoso: hay que decidir qué operaciones son síncronas y cuáles asíncronas.
- Riesgo de duplicación de eventos: requiere idempotencia en los consumidores.
- Mayor complejidad de testing (tests de integración con Kafka + REST).

## Follow-ups
- Definir el esquema de eventos (AVRO/JSON Schema) para el bus de Kafka.
- Definir la estrategia de idempotencia en consumidores de eventos.
- POC: validar latencia p95 del flujo REST + Kafka con un piloto de Order Service antes de migrar producción.
- ADR-0003: Estrategia de base de datos (DB per service vs shared DB) considerando los patrones de comunicación definidos.

---

*Referencias: [Brief §A.4] [Richardson Cap. 3, Cap. 4] [ADR-0001]*
