# Functional Specification Document (FSD) – FTGO (Food To Go)

> **Propósito**: especificar los casos de uso (UCs) del sistema FTGO en formato Given/When/Then (BDD), derivados de las 3 user stories semilla del brief y del PRD. Sirve como entrada para los ADRs y diagramas C4.
>
> **Fuentes**: docs/PRD.md • Brief del caso FTGO (Anexo A) • Richardson Cap. 2–3

---

## 1. Introducción

Este FSD documenta los casos de uso esenciales de la plataforma FTGO para la migración a microservicios. Cubre el flujo completo del pedido desde la selección de restaurante hasta la entrega, incluyendo pagos, notificaciones y tracking. Los UCs derivados se justifican desde el brief o el libro.

## 2. Tabla de casos de uso

| ID | Título | Actor primario | Capacidad PRD | Origen |
|----|--------|----------------|---------------|--------|
| UC-01 | Tomar pedido del consumidor | Consumidor | Order Taking | US-01 [Brief §A.5] |
| UC-02 | Aceptar/rechazar ticket de pedido | Restaurante | Order Fulfillment / Kitchen | US-02 [Brief §A.5] |
| UC-03 | Asignar entrega a courier | Courier | Delivery | US-03 [Brief §A.5] |
| UC-04 | Procesar pago del pedido | Consumidor | Billing & Accounting | Derivado del flujo de pago [Richardson Cap. 3] |
| UC-05 | Tracking en tiempo real del pedido | Consumidor | Delivery | Derivado de NFR-02 (latencia UX) y NFR-07 (trazabilidad) [Brief §A.4] |
| UC-06 | Cancelar pedido por consumidor | Consumidor | Order Taking | UCs adicionales válidos [Brief §A.5] |

## 3. Detalle de casos de uso

---

### UC-01: Tomar pedido del consumidor

| Campo | Valor |
|-------|-------|
| Actor primario | Consumidor |
| Capacidad PRD | Order Taking |
| Origen | US-01 [Brief §A.5] |
| **Precondiciones** | El consumidor tiene sesión activa en la app. El restaurante está disponible y tiene ítems en el menú. |
| **Flujo principal** | 1. El consumidor selecciona un restaurante de la lista.<br>2. El sistema muestra el menú con ítems, precios y disponibilidad.<br>3. El consumidor agrega/quita ítems al carrito.<br>4. El consumidor ingresa dirección de entrega y selecciona método de pago.<br>5. El sistema valida disponibilidad del restaurante y stock.<br>6. El sistema calcula el total, aplica cargos de delivery y confirma el pedido.<br>7. El sistema asigna un número de pedido único y notifica al consumidor. |
| **Flujos alternativos** | 3a. Ítem no disponible: se muestra como agotado, no se permite agregar.<br>5a. Restaurante no disponible en el horario: se rechaza el pedido y se notifica. |
| **Postcondiciones** | Pedido creado en estado `PENDING_PAYMENT`. Número de pedido único generado. Notificación enviada al consumidor. |

**Given/When/Then**:
- **Given** el Consumidor tiene un carrito con al menos 1 ítem, dirección de entrega y método de pago válidos
- **When** el Consumidor confirma el pedido
- **Then** el sistema crea el pedido en estado `PENDING_PAYMENT`, asigna un número único, y envía confirmación al consumidor

---

### UC-02: Aceptar/rechazar ticket de pedido

| Campo | Valor |
|-------|-------|
| Actor primario | Restaurante |
| Capacidad PRD | Order Fulfillment / Kitchen |
| Origen | US-02 [Brief §A.5] |
| **Precondiciones** | El restaurante tiene sesión activa en el dashboard. Existe al menos un ticket pendiente. |
| **Flujo principal** | 1. El sistema notifica al restaurante de un nuevo ticket entrante.<br>2. El restaurante visualiza el ticket con ítems, cantidad y tiempo estimado.<br>3. El restaurante acepta el ticket e ingresa tiempo estimado de preparación.<br>4. El sistema actualiza el estado del pedido a `ACCEPTED` y notifica al consumidor. |
| **Flujos alternativos** | 3a. El restaurante rechaza el ticket con motivo (ej. falta de insumos).<br>4a. Si rechaza, el pedido pasa a `CANCELLED` y se notifica al consumidor. |
| **Postcondiciones** | Ticket aceptado o rechazado. Consumidor notificado del cambio de estado. |

**Given/When/Then**:
- **Given** el Restaurante tiene un ticket entrante pendiente en el dashboard
- **When** el Restaurante acepta el ticket e ingresa tiempo estimado de preparación
- **Then** el sistema actualiza el pedido a estado `ACCEPTED` y notifica al Consumidor

---

### UC-03: Asignar entrega a courier

| Campo | Valor |
|-------|-------|
| Actor primario | Courier |
| Capacidad PRD | Delivery |
| Origen | US-03 [Brief §A.5] |
| **Precondiciones** | El pedido está en estado `READY_FOR_PICKUP`. El courier está disponible y marcado como disponible en la app. |
| **Flujo principal** | 1. El sistema identifica pedidos listos para retiro cerca de la ubicación del courier.<br>2. El sistema ofrece la asignación al courier con detalles (restaurante, destino, distancia).<br>3. El courier acepta la asignación.<br>4. El sistema muestra la ruta optimizada al restaurante y al consumidor.<br>5. El sistema actualiza el estado del pedido a `IN_DELIVERY`. |
| **Flujos alternativos** | 3a. El courier rechaza dentro del timeout (30 s): el sistema reasigna automáticamente a otro courier disponible.<br>3b. Timeout sin respuesta: el sistema cancela la oferta y reasigna. |
| **Postcondiciones** | Pedido asignado a un courier en estado `IN_DELIVERY`. Ruta optimizada generada. |

**Given/When/Then**:
- **Given** un pedido está listo para retiro y un Courier está disponible cerca
- **When** el Courier acepta la asignación dentro del timeout de 30 s
- **Then** el sistema asigna el pedido, muestra la ruta optimizada y actualiza el estado a `IN_DELIVERY`

---

### UC-04: Procesar pago del pedido

| Campo | Valor |
|-------|-------|
| Actor primario | Consumidor (a través de Stripe) |
| Capacidad PRD | Billing & Accounting |
| Origen | Derivado del flujo de pago [Richardson Cap. 3] |
| **Precondiciones** | Pedido en estado `PENDING_PAYMENT`. Consumidor tiene método de pago registrado (tarjeta). |
| **Flujo principal** | 1. El sistema envía la solicitud de cobro a Stripe con el monto total.<br>2. Stripe procesa el pago y devuelve confirmación.<br>3. El sistema actualiza el pedido a `PAID` y registra la transacción.<br>4. El sistema notifica al consumidor del pago exitoso.<br>5. El sistema envía el ticket al restaurante. |
| **Flujos alternativos** | 2a. Stripe rechaza el pago (fondos insuficientes, tarjeta inválida): el sistema notifica al consumidor para que intente con otro método.<br>2b. Stripe no responde (timeout): el sistema encola el cobro en una cola de retry y mantiene el pedido en `PENDING_PAYMENT`. |
| **Postcondiciones** | Pedido pagado o encolado para retry. Transacción registrada en Billing. |

**Given/When/Then**:
- **Given** un pedido en estado `PENDING_PAYMENT` con un método de pago válido asociado a Stripe
- **When** Stripe confirma el cobro exitoso
- **Then** el sistema actualiza el pedido a `PAID`, registra la transacción y notifica al Consumidor

---

### UC-05: Tracking en tiempo real del pedido

| Campo | Valor |
|-------|-------|
| Actor primario | Consumidor |
| Capacidad PRD | Delivery |
| Origen | Derivado de NFR-02 (latencia UX) y NFR-07 (trazabilidad) [Brief §A.4] |
| **Precondiciones** | Pedido en estado `IN_DELIVERY`. Courier en ruta. |
| **Flujo principal** | 1. El sistema recibe actualizaciones de geolocalización del courier (Google Maps).<br>2. El sistema muestra al consumidor la posición del courier en un mapa en tiempo real.<br>3. El sistema muestra el tiempo estimado de llegada actualizado dinámicamente.<br>4. El sistema notifica al consumidor cuando el courier está "cerca" (a <5 min). |
| **Flujos alternativos** | 1a. Pérdida de señal GPS: el sistema muestra la última posición conocida y un mensaje de "actualizando...".<br>2a. Google Maps no disponible: el sistema degrada a tracking por hitos (texto) sin mapa. |
| **Postcondiciones** | Consumidor visualiza tracking en tiempo real. Notificación de llegada inminente enviada. |

**Given/When/Then**:
- **Given** un pedido en estado `IN_DELIVERY` con un Courier en ruta
- **When** el sistema recibe una actualización de geolocalización del Courier
- **Then** el sistema actualiza la posición en el mapa del Consumidor y recalcula el tiempo estimado de llegada

---

### UC-06: Cancelar pedido por consumidor

| Campo | Valor |
|-------|-------|
| Actor primario | Consumidor |
| Capacidad PRD | Order Taking |
| Origen | UCs adicionales válidos [Brief §A.5] |
| **Precondiciones** | Pedido existe y está en estado `PENDING_PAYMENT` o `PAID` (antes de que el restaurante comience la preparación). |
| **Flujo principal** | 1. El consumidor selecciona "Cancelar pedido" desde la app.<br>2. El sistema verifica que el pedido sea cancelable (estado permitido).<br>3. El sistema cambia el estado a `CANCELLED_BY_CONSUMER`.<br>4. El sistema notifica al restaurante de la cancelación.<br>5. Si el pedido fue pagado, el sistema inicia el reembolso vía Stripe. |
| **Flujos alternativos** | 2a. Pedido ya en preparación (estado `ACCEPTED` por el restaurante): el sistema informa que no es cancelable y sugiere contactar a soporte.<br>5a. Stripe falla al reembolsar: se encola para retry y se notifica al back office. |
| **Postcondiciones** | Pedido cancelado. Restaurante notificado. Reembolso iniciado si aplica. |

**Given/When/Then**:
- **Given** un pedido en estado `PENDING_PAYMENT` o `PAID` que aún no ha sido aceptado por el Restaurante
- **When** el Consumidor solicita la cancelación
- **Then** el sistema cambia el estado a `CANCELLED_BY_CONSUMER`, notifica al Restaurante e inicia el reembolso si corresponde

---

*Trazabilidad: este FSD se alinea con el PRD (`docs/PRD.md`) y alimenta a los ADRs (`docs/adr/0001-*.md`, `docs/adr/0002-*.md`) y diagramas C4 (`docs/diagrams/`).*
