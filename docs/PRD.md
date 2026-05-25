# Product Requirements Document (PRD) – FTGO (Food To Go)

> **Propósito**: formalizar los requisitos de producto de FTGO para guiar la migración de monolito Java a microarquitectura de servicios. Sirve como entrada para el FSD y los ADRs arquitectónicos.
>
> **Fuente canónica**: Richardson, *Microservices Patterns*, Manning 2019; Brief del caso FTGO (Anexo A).

---

## 1. Contexto y objetivos

FTGO es un marketplace de delivery de comida que opera como monolito Java desde hace varios años. El monolito presenta builds >20 min, escalado conflictivo entre módulos, falta de aislamiento de fallos, lock-in tecnológico y un equipo bloqueado por el tamaño del código base [Brief §A.1].

La dirección decidió migrar a microservicios mediante el patrón Strangler Fig [Brief §A.4 Migración incremental]. Este PRD documenta la arquitectura objetivo para que los equipos de desarrollo puedan comenzar la migración incremental durante 18–24 meses.

**Objetivo**: descomponer el monolito en servicios autónomos alineados a las capacidades de negocio, manteniendo la operación continua durante la migración.

## 2. Stakeholders

| Rol | Descripción | Necesidad principal |
|-----|-------------|---------------------|
| Consumidor | Usuario final (móvil/web) que ordena comida | UX rápida, tracking en tiempo real, disponibilidad 99.9 % |
| Restaurante | Socio que prepara la comida | Dashboard de tickets, control de carga de cocina |
| Courier | Repartidor independiente | Asignaciones cercanas, rutas optimizadas, pago confiable |
| Empleado FTGO (back office) | Customer support, finanzas, operaciones | Visibilidad, reportes, resolución de incidentes |
| Equipo de arquitectura | Responsable del rediseño a microservicios | Calidad arquitectónica, trazabilidad, mantenibilidad |
| Sistemas externos | Stripe, Google Maps, SendGrid, Twilio | Integración estable, SLAs predecibles |

*Fuente: [Brief §A.2]*

## 3. Capacidades de negocio

Las 7 capacidades de negocio estables identificadas en el Cap. 2 del libro como candidatas a descomposición:

| # | Capacidad | Responsabilidad |
|---|-----------|-----------------|
| 1 | **Consumer Management** | Registro, perfiles, direcciones, preferencias de los consumidores |
| 2 | **Restaurant Management** | Restaurantes registrados, menús, horarios, disponibilidad |
| 3 | **Order Taking** | Toma de pedidos: validación, cálculo de total, confirmación |
| 4 | **Order Fulfillment / Kitchen** | Tickets entregados al restaurante, estado de preparación |
| 5 | **Delivery** | Asignación de couriers, rutas, tracking en tiempo real |
| 6 | **Billing & Accounting** | Cobros, comisiones, payouts a restaurantes y couriers |
| 7 | **Notifications** | Emails, SMS, push: confirmaciones, alertas, recibos |

*Fuente: [Brief §A.3] [Richardson Cap. 2]*

> **Nota**: estas capacidades NO son automáticamente microservicios uno-a-uno. La decisión de granularidad se documenta en los ADRs.

## 4. Requisitos no funcionales

| ID | Categoría | Métrica | Origen | Justificación |
|----|-----------|---------|--------|---------------|
| NFR-01 | Carga | Tráfico pico de 5x en horarios 12:00-14:00 y 19:00-22:00 | [Brief §A.4 Carga] | Escalado horizontal independiente requerido |
| NFR-02 | Latencia UX | Tiempo de respuesta <200 ms p95 para acciones del consumidor | [Brief §A.4 Latencia UX] | Experiencia móvil competitiva |
| NFR-03 | Disponibilidad | 99.9 % mensual flujo de toma de pedidos; tracking puede degradar a 99.5 % | [Brief §A.4 Disponibilidad] | SLA crítico para el core del negocio |
| NFR-04 | Tolerancia a fallos externos | Pedidos aceptados aunque Stripe esté caído (cola de retry); degradación graceful de mapas | [Brief §A.4 Tolerancia a fallos] | Continuidad del negocio ante fallos de terceros |
| NFR-05 | Escalabilidad horizontal | Cada componente escala independientemente (X-axis y Y-axis del Scale Cube) | [Brief §A.4 Escalabilidad] | Aislamiento de carga por capacidad |
| NFR-06 | Consistencia de datos | Eventual consistency para reporting; fuerte consistencia dentro del aggregate de un pedido | [Brief §A.4 Consistencia] | Integridad del pedido vs rendimiento de reportes |
| NFR-07 | Trazabilidad | Cada acción del consumidor rastreada end-to-end con correlation ID | [Brief §A.4 Trazabilidad] | Auditoría y depuración distribuida |
| NFR-08 | Migración incremental | Strangler Fig durante 18-24 meses; monolito no se reemplaza de golpe | [Brief §A.4 Migración] | Riesgo controlado durante la transición |
| NFR-09 | Tecnología | Java/Spring Boot en el core; libertad tecnológica en servicios satélite | [Brief §A.4 Tecnología] | Reutilizar conocimiento del equipo existente |
| NFR-10 | Cumplimiento | PCI-DSS delegado a Stripe; GDPR para datos de consumidores | [Brief §A.4 Cumplimiento] | Seguridad de pagos y privacidad de datos |

## 5. Alcance

### En alcance
- Documentación de la arquitectura objetivo de microservicios para FTGO.
- Descomposición de las 7 capacidades de negocio en servicios candidatos.
- Definición de NFRs, stakeholders y restricciones para la migración.
- Trazabilidad al brief del caso FTGO y al libro de Richardson.

### Fuera de alcance
- Implementación del código de los microservicios (el PRD documenta, no construye).
- Migración completa del monolito (se realiza por fases durante 18–24 meses, fuera del alcance de este documento).
- Nuevas funcionalidades de negocio no contempladas en las 7 capacidades del Cap. 2.
- Infraestructura cloud específica (proveedor, costos detallados).

---

*Trazabilidad: este PRD se alinea con el BRD (`docs/brd.md`) y alimenta al FSD (`docs/FSD.md`) y a los ADRs (`docs/adr/0001-*.md`, `docs/adr/0002-*.md`).*
