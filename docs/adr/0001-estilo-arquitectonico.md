# ADR-0001: Estilo arquitectónico para FTGO

**Status**: Accepted

**Fecha**: 2026-05-24

## Contexto

FTGO opera actualmente como un monolito Java que presenta builds >20 min, escalado conflictivo entre módulos, falta de aislamiento de fallos y lock-in tecnológico [Brief §A.1]. La dirección decidió migrar a una arquitectura de microservicios para sostener el crecimiento.

El equipo de arquitectura debe decidir el **estilo arquitectónico objetivo** para la migración, considerando las siguientes restricciones:
- Tráfico pico 5x en horas de almuerzo y cena [Brief §A.4 Carga] → exige escalado horizontal independiente.
- Latencia <200 ms p95 para acciones del consumidor [Brief §A.4 Latencia UX].
- Tolerancia a fallos externos (Stripe, Google Maps) [Brief §A.4 Tolerancia a fallos].
- Migración incremental Strangler Fig (18–24 meses) [Brief §A.4 Migración].
- Java/Spring Boot preferido en el core [Brief §A.4 Tecnología].

## Opciones consideradas

### Opción 1: Microservicios por capacidad de negocio (7 servicios)

**Descripción**: descomponer el monolito en 7 microservicios, uno por cada capacidad de negocio del Cap. 2 del libro: Consumer, Restaurant, Order, Kitchen, Delivery, Billing, Notification.

**Pros**:
- Escalabilidad horizontal independiente por capacidad → cubre NFR-01 (5x pico) y NFR-05.
- Aislamiento de fallos → un fallo en Notifications no bloquea Order Taking (NFR-04).
- Alineamiento directo con Bounded Context de DDD [Richardson Cap. 2].
- Equipos pequeños y autónomos por servicio.

**Contras**:
- 7 microservicios desde el día 1 → alta complejidad operativa.
- Riesgo de Distributed Monolith si las capacidades están acopladas.
- Costo de infraestructura inicial alto.
- Overhead de comunicación entre servicios puede degradar latencia (NFR-02).

**Impacto en NFRs**: NFR-01 ✓, NFR-02 ? (latencia puede degradar por red), NFR-03 ✓, NFR-04 ✓, NFR-05 ✓.

### Opción 2: Monolito modular con módulos desacoplables

**Descripción**: mantener un solo proceso pero con módulos internos con interfaces bien definidas (boundaries), que puedan extraerse a microservicios progresivamente.

**Pros**:
- Menor complejidad operativa inicial (un solo deploy).
- Latencia interna sin overhead de red → cubre NFR-02 (<200 ms p95).
- Costo de infraestructura inicial bajo.
- Estrategia natural para Strangler Fig: se extraen módulos uno a uno.

**Contras**:
- No hay escalado independiente real hasta que se extrae el módulo → parcial para NFR-01.
- Falta de aislamiento de fallos mientras no se extraiga → no cubre NFR-04 totalmente.
- El equipo debe disciplinarse para no acoplar módulos (deuda técnica).
- La modularización inicial es un paso intermedio que retrasa el objetivo final.

**Impacto en NFRs**: NFR-01 ± (escalado solo cuando se extraiga), NFR-02 ✓, NFR-03 ±, NFR-04 ±, NFR-05 ±.

### Opción 3: Serverless (Funciones como servicio)

**Descripción**: implementar cada capacidad como funciones serverless (AWS Lambda / Azure Functions) con API Gateway y BDs serverless.

**Pros**:
- Escalado elástico automático → cubre NFR-01 (5x pico) sin provisioning manual.
- Pago por uso → costo proporcional al tráfico real.
- Sin gestión de servidores.

**Contras**:
- Límite de tiempo de ejecución (15 min Lambda) → conflictivo para procesos largos (ej. payout batch).
- Cold start puede degradar latencia en horas valle → riesgo para NFR-02 (<200 ms p95).
- Vendor lock-in significativo.
- Complejidad de debugging distribuido y testing local.
- Dificultad para migración incremental Strangler Fig (el monolito existente es Java/Spring Boot, no serverless).

**Impacto en NFRs**: NFR-01 ✓, NFR-02 ? (cold start), NFR-03 ✓, NFR-04 ✓, NFR-05 ✓, NFR-08 ? (difícil migración incremental).

### Opción 4: Microservicios por subdominio DDD (descomposición más fina)

**Descripción**: descomponer en más de 7 servicios aplicando DDD estratégico, identificando subdominios dentro de cada capacidad. Ej: separar Order Taking de Order Validation, o separar Payouts de Billing.

**Pros**:
- Alta granularidad → escalado muy fino y aislamiento máximo.
- Equipos muy pequeños y autónomos.
- Mapeo preciso a subdominios DDD [Richardson Cap. 2].

**Contras**:
- Complejidad operativa excesiva (posiblemente >15 servicios).
- Latencia agregada por llamadas entre servicios → riesgo severo para NFR-02.
- Dificultad de mantener consistencia de datos entre servicios.
- Overhead de orquestación y observabilidad muy alto.
- Contradice la restricción del laboratorio de "PRD ligero, no exhaustivo".

**Impacto en NFRs**: NFR-01 ✓✓, NFR-02 ✗ (excesivo overhead), NFR-03 ?, NFR-04 ✓, NFR-05 ✓.

## Decisión

**Se elige la Opción 1: Microservicios por capacidad de negocio (7 servicios), pero con un enfoque híbrido de migración.**

Justificación:
1. Es la opción que mejor balancea escalado independiente (NFR-01/05), aislamiento de fallos (NFR-04) y viabilidad de migración incremental (NFR-08) [Richardson Cap. 2].
2. La migración comienza extrayendo un servicio a la vez (Order Taking primero por ser el core del negocio), mientras el resto del monolito sigue operando (Strangler Fig).
3. Se mantiene Java/Spring Boot en el core para reutilizar conocimiento del equipo (NFR-09).
4. Para mitigar el riesgo de latencia (NFR-02), los servicios críticos en el flujo del pedido se comunican de forma síncrona eficiente (ver ADR-0002).
5. Se evita la granularidad excesiva de la Opción 4 (que penalizaría NFR-02) y el paso intermedio sin valor real de la Opción 2.

## Consecuencias

### Positivas
- Escalado horizontal independiente por capacidad de negocio.
- Aislamiento de fallos: un crash en Notifications no afecta Order Taking.
- Equipos pequeños pueden ser dueños de cada servicio (autonomía).
- La migración gradual (Strangler Fig) permite retroalimentación temprana.

### Negativas
- Complejidad operativa: 7 servicios requieren orquestación, monitoreo y observabilidad robustos.
- Overhead de comunicación en red → requiere diseño cuidadoso de IPC (ver ADR-0002).
- Costos iniciales de infraestructura y tooling más altos que el monolito.
- Se requiere inversión en CI/CD, containerización (Docker/K8s) y distributed tracing.

## Follow-ups
- ADR-0002: Definir mecanismo IPC predominante (síncrono REST vs asíncrono eventos).
- ADR-0003: Estrategia de base de datos (DB per service vs shared DB).
- POC: Validar latencia p95 con un piloto de Order Service + Kafka antes de migrar producción.

---

*Referencias: [Brief §A.1] [Brief §A.4] [Richardson Cap. 1, Cap. 2]*
