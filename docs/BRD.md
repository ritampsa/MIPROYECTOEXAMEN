# Business Requirements Document (BRD) – FTGO (Food To Go)

> **Propósito del BRD**: formalizar las necesidades y restricciones de negocio que justifican la migración de FTGO de una arquitectura monolítica a microservicios, independientemente de la solución técnica. Responde a "¿qué necesita el negocio y por qué?".
>
> **Alcance**: este BRD captura la visión del sponsor y las capacidades de negocio estables identificadas en *Microservices Patterns* (Richardson, 2019), como primer documento de la cadena `BRD → MRD → PRD → FSD → DTI`.

---

## 0. Metadatos

| Campo | Valor |
|-------|-------|
| Producto | FTGO (Food To Go) |
| Grupo | FTGO Architecture Team |
| Versión | v0.1 |
| Fecha | 24/05/2026 |
| Sponsor de negocio | Dirección de Producto FTGO |
| Stakeholders | Consumidor, Restaurante, Courier, Empleado FTGO (back office), Equipo de arquitectura, Sistemas externos (Stripe, Google Maps, SendGrid, Twilio) |
| Autores | Equipo de arquitectura FTGO |
| Revisores | Docente + 1 grupo par |
| Estado | Borrador |
| Insumo del Módulo Anterior (M2 UI/UX) | Brief del caso FTGO (Anexo A, PracticaM4.docx) |
| Prompts utilizados | PR-BRD-001 |

## 1. Resumen ejecutivo

**Problema**: la plataforma FTGO opera como un monolito Java que presenta builds lentos (>20 min), escalado conflictivo entre módulos, falta de aislamiento de fallos y lock-in tecnológico, lo que limita el crecimiento del negocio en un mercado de delivery que exige despliegues frecuentes y alta disponibilidad.

**Propuesta**: migrar la plataforma FTGO a una arquitectura de microservicios mediante el patrón Strangler Fig, descomponiendo el monolito en servicios autónomos alineados a las 7 capacidades de negocio estables, permitiendo escalado independiente, despliegues desacoplados y evolución tecnológica sin interrumpir la operación.

**Valor esperado**:
- Reducción del tiempo de build y despliegue de 20 min a <5 min.
- Disponibilidad del flujo crítico de toma de pedidos al 99.9 % mensual.
- Capacidad de escalar componentes de forma independiente durante picos 5x en horas de almuerzo y cena.

**Métricas clave de éxito**:
- Tiempo de respuesta percibido <200 ms p95 en acciones del consumidor.
- 0 incidentes críticos por acoplamiento entre módulos durante la migración.
- 100 % de trazabilidad end-to-end con correlation ID en todas las transacciones.

**Llamada a la acción**: aprobar la hoja de ruta de migración a microservicios con horizonte 18–24 meses y asignar los equipos de desarrollo necesarios para iniciar el desacoplamiento incremental.

## 2. Contexto del negocio

- **Organización**: FTGO Inc.
- **Unidad impactada**: Plataforma de delivery (Producto, Ingeniería, Operaciones).
- **Proceso(s) de negocio afectado(s)**: Toma de pedidos, preparación en restaurante, asignación de entregas, facturación y pagos, notificaciones al consumidor.
- **Estrategia de la organización** que justifica el proyecto: sostener el crecimiento acelerado del marketplace de delivery eliminando el bloqueo tecnológico del monolito, permitiendo despliegues frecuentes, escalado elástico y adopción de nuevas capacidades de negocio sin fricción.

## 3. Problema y oportunidad de negocio

### 3.1 Problema

FTGO ha operado durante varios años como una aplicación monolítica Java empaquetada como WAR. Este monolito ha acumulado los síntomas clásicos del "infierno monolítico": los builds tardan más de 20 minutos, el despliegue requiere coordinar a todo el equipo, un fallo en un módulo (ej. notificaciones) puede derribar todo el sistema, y el escalado horizontal es ineficiente porque se replica el monolito completo aunque solo un módulo (ej. toma de pedidos en hora pico) requiera más capacidad. El equipo de desarrollo está cada vez más bloqueado por el tamaño del código base, las dependencias ocultas entre módulos y la imposibilidad de adoptar tecnologías nuevas sin afectar al resto del sistema. Si no se actúa, FTGO perderá capacidad de respuesta frente a competidores nativos de microservicios, aumentará el time-to-market de nuevas funcionalidades y elevará el riesgo de incidentes graves en producción.

### 3.2 Oportunidad

- **Valor económico estimado**: reducción de costos operativos al escalar solo los componentes necesarios (estimado 20–30 % de ahorro en infraestructura), disminución de horas-hombre perdidas en builds y despliegues lentos, y evitación de pérdidas de ingresos por caídas del monolito completo.
- **Valor estratégico / reputacional**: capacidad de lanzar nuevas funcionalidades en días en lugar de semanas, mejorando la posición competitiva en el mercado de delivery.
- **Ventana de oportunidad**: el monolito aún es mantenible pero la deuda técnica crece; cada mes sin migrar incrementa el costo de refactorización.

### 3.3 Evidencia de Continuous Discovery

- **Documento de Discovery**: brief del caso FTGO (Anexo A del PracticaM4.docx).
- **Entrevistas realizadas**: derivadas del libro *Microservices Patterns* (Richardson, 2019), Cap. 1–2, donde se documentan los síntomas y soluciones.
- **Hipótesis principales validadas**: las 7 capacidades de negocio (Consumer Management, Restaurant Management, Order Taking, Order Fulfillment/Kitchen, Delivery, Billing & Accounting, Notifications) son estables y candidatas viables a microservicios según el Cap. 2 del libro.
- **Artefactos M2 (UI/UX)**: user stories semilla US-01 (toma de pedido), US-02 (aceptación de tickets), US-03 (asignación de entrega).
- **Próxima cadencia de Discovery**: semanal durante la iteración de migración.

## 4. Usuarios objetivo / Personas clave

### 4.1 Persona principal – Consumidor

| Atributo | Valor |
|----------|-------|
| Nombre / rol | Consumidor – Usuario final (móvil/web) que ordena comida a domicilio |
| Contexto | Interactúa con la app FTGO desde su smartphone para buscar restaurantes, ver menús, realizar pedidos y hacer seguimiento de la entrega en tiempo real |
| *Jobs‑to‑be‑done* | 1. Encontrar un restaurante cercano con el tipo de comida deseada<br>2. Realizar un pedido en menos de 2 minutos<br>3. Pagar de forma segura y rápida<br>4. Conocer el estado del pedido en tiempo real<br>5. Recibir la comida en el domicilio dentro del tiempo estimado |
| Dolores principales | Lentitud en la app durante horas pico, falta de transparencia del estado del pedido, errores en el tracking, caídas del sistema que impiden completar pedidos |
| Ganancia esperada | Experiencia de usuario rápida (<200 ms p95), tracking confiable, disponibilidad 99.9 % del flujo de pedidos |

### 4.2 Persona secundaria – Restaurante

| Atributo | Valor |
|----------|-------|
| Nombre / rol | Restaurante – Socio que prepara la comida |
| Contexto | Opera su cocina y recibe tickets de pedido desde la plataforma FTGO; necesita gestionar la carga de trabajo sin saturarse |
| *Jobs‑to‑be‑done* | 1. Recibir y aceptar/rechazar tickets de pedido<br>2. Actualizar tiempo estimado de preparación<br>3. Gestionar disponibilidad del menú en tiempo real<br>4. Coordinar con couriers la entrega<br>5. Conciliar pagos y comisiones |
| Dolores principales | Notificaciones tardías de nuevos pedidos, dashboard lento durante horas pico, falta de control sobre la carga de cocina |
| Ganancia esperada | Dashboard ágil, notificaciones en tiempo real, control de capacidad de cocina para evitar rejections no deseadas |

## 5. Propuesta de valor

| Eje | Contenido |
|-----|-----------|
| **Para quién** | Consumidores que desean pedir comida a domicilio y restaurantes asociados que buscan gestionar pedidos de forma eficiente |
| **Que necesita** | Realizar pedidos de comida y gestionar tickets de cocina con rapidez, confiabilidad y tracking en tiempo real |
| **Nuestra propuesta es** | Una plataforma de marketplace de delivery con arquitectura de microservicios que ofrece experiencia de usuario fluida, alta disponibilidad y escalado elástico |
| **Que le aporta** | • Tiempo de respuesta <200 ms p95 en la app del consumidor<br>• Disponibilidad 99.9 % del flujo crítico de pedidos<br>• Tracking en tiempo real con actualizaciones de estado<br>• Dashboard de tickets con notificaciones instantáneas para restaurantes<br>• Tolerancia a fallos externos (pagos, mapas) sin interrumpir la operación |
| **A diferencia de** | El monolito actual que sufre caídas completas, builds lentos, escalado ineficiente y falta de aislamiento de fallos |
| **Nuestro diferencial es** | Migración incremental controlada (Strangler Fig) que permite evolucionar sin detener el negocio, con trazabilidad end-to-end y consistencia de datos dentro del aggregate de cada pedido |

## 6. Panorama competitivo (resumen)

| Competidor / alternativa | Tipo | Fortaleza percibida | Debilidad percibida |
|--------------------------|------|---------------------|---------------------|
| Monolito FTGO actual | *do‑nothing* | Conocido por el equipo, sin riesgo de migración | Builds >20 min, escalado conflictivo, caídas completas, lock-in tecnológico |
| Competidores nativos cloud (Uber Eats, Rappi) | directo | UX moderna, microservicios nativos, escalado elástico | Alto costo operativo, no personalizado para el mercado local de FTGO |
| Plataformas white‑label de delivery (OpenTable, Toast) | indirecto | Datos integrados, soluciones llave en mano | No cubren el modelo de negocio específico de FTGO, lock-in de proveedor |

> Nota: este resumen se complementa con la sección de competencia del MRD.

## 7. Business Model Canvas

| Bloque | Mínimo 3 elementos concretos |
|--------|-------------------------------|
| 1. Segmentos de clientes | Consumidores finales / Restaurantes asociados / Couriers independientes |
| 2. Propuesta de valor | Entrega rápida y confiable / Gestión eficiente de pedidos para restaurantes / Oportunidad de ingresos para couriers |
| 3. Canales | App móvil consumidor / Dashboard web restaurante / App courier / Notificaciones push, email y SMS |
| 4. Relación con clientes | Soporte al consumidor 24/7 / Dashboard de reportes para restaurantes / Programa de fidelidad y promociones |
| 5. Fuentes de ingresos | Comisión por pedido (+15 %) / Tarifa de delivery / Suscripción premium para restaurantes |
| 6. Recursos clave | Plataforma tecnológica (microservicios) / Red de restaurantes / Flota de couriers / Datos de pedidos y preferencias |
| 7. Actividades clave | Migración a microservicios (Strangler Fig) / Mantenimiento de plataforma / Adquisición de restaurantes / Optimización de rutas de delivery |
| 8. Socios clave | Stripe (pasarela de pago) / Google Maps (geocoding y rutas) / SendGrid (email) / Twilio (SMS y push) |
| 9. Estructura de costos | Infraestructura cloud (escalado horizontal) / Salarios equipo de ingeniería / Comisiones a pasarelas de pago / Marketing y adquisición de usuarios |

## 8. Métricas clave de éxito (North Star + apoyo)

| ID | KPI | North Star? | Línea base | Meta | Horizonte | Fuente del dato |
|----|-----|-------------|------------|------|-----------|-----------------|
| KPI-01 | Tiempo medio de respuesta p95 en app consumidor | Sí | >500 ms (monolito actual) | ≤200 ms p95 | Q4 2026 | APM (Distributed Tracing) |
| KPI-02 | Disponibilidad flujo de toma de pedidos | No | <99.5 % (caídas completas) | ≥99.9 % mensual | Q4 2026 | Monitoreo de uptime |
| KPI-03 | Tiempo medio de build y despliegue | No | >20 min | ≤5 min | Q2 2026 | Pipeline CI/CD |

## 9. Objetivos de negocio (SMART)

| ID | Objetivo | Métrica | Línea base | Meta | Horizonte |
|----|----------|---------|------------|------|-----------|
| BO-01 | Reducir tiempo de build y despliegue del monolito a microservicios | minutos promedio | >20 min | ≤5 min | Q2 2026 |
| BO-02 | Alcanzar disponibilidad del flujo crítico de pedidos | % mensual | <99.5 % | ≥99.9 % | Q4 2026 |
| BO-03 | Mantener latencia percibida por el consumidor por debajo del umbral crítico | ms p95 | >500 ms | ≤200 ms | Q4 2026 |
| BO-04 | Lograr trazabilidad end-to-end en todas las transacciones | % de transacciones con correlation ID | 0 % | 100 % | Q2 2026 |

## 10. Stakeholders y roles (modelo RACI)

| Stakeholder | Interés | R / A / C / I |
|-------------|---------|----------------|
| Patrocinador (Dirección de Producto) | estratégico | A |
| Consumidor | experiencia | C |
| Restaurante | operativo | C |
| Courier | operativo | C |
| Empleado FTGO (back office) | visibilidad, reportes | C |
| Equipo de arquitectura | calidad arquitectónica | R |
| Sistemas externos (Stripe, Google Maps, SendGrid, Twilio) | integración estable | I |

## 11. Requerimientos de negocio

| ID | Requerimiento de negocio | Prioridad (MoSCoW) | Justificación | Métrica de aceptación |
|----|---------------------------|--------------------|---------------|-----------------------|
| BR-001 | El consumidor debe poder realizar un pedido desde el menú de un restaurante seleccionado sin presencia física | Must | Core del negocio de delivery | 95 % de pedidos iniciados y completados en línea |
| BR-002 | El restaurante debe poder aceptar o rechazar tickets de pedido entrantes en tiempo real | Must | Control de carga de cocina | 100 % de tickets notificados en <5 s |
| BR-003 | El sistema debe asignar entregas a couriers cercanos de forma automática | Must | Eficiencia operativa de delivery | 80 % de entregas asignadas en <30 s |
| BR-004 | El consumidor debe poder pagar el pedido con tarjeta (Stripe) y ver el cobro reflejado | Must | Flujo de ingresos | Tasa de éxito de pago ≥98 % |
| BR-005 | El sistema debe tolerar la caída de la pasarela de pago sin perder pedidos | Should | Continuidad del negocio | 100 % de pedidos encolados para retry si Stripe falla |
| BR-006 | El consumidor debe poder ver el tracking en tiempo real de su pedido | Should | Diferenciación UX | 90 % de pedidos con al menos 3 actualizaciones de tracking |
| BR-007 | El back office debe poder generar reportes de pedidos, ingresos y comisiones | Could | Visibilidad operativa | Reportes generados en <10 s con datos de hasta 24 h atrás |
| BR-008 | El restaurante debe poder actualizar su menú y disponibilidad en tiempo real | Must | Gestión de oferta | Cambios de menú reflejados en <30 s |
| BR-009 | El sistema debe notificar al consumidor cambios de estado del pedido (confirmación, preparación, envío, entrega) | Must | Transparencia UX | 100 % de cambios de estado notificados en <10 s |
| BR-010 | El sistema debe soportar escalado horizontal independiente por capacidad de negocio | Must | Picos de tráfico 5x en horas punta | Cada servicio escala sin afectar a los demás |

## 12. Reglas de negocio y políticas

| ID | Regla | Tipo | Origen |
|----|-------|------|--------|
| RB-01 | El consumidor debe tener un método de pago válido antes de confirmar un pedido | política | Flujo de cobro |
| RB-02 | Un restaurante puede rechazar un pedido si supera su capacidad actual de cocina | política | Gestión de carga del restaurante |
| RB-03 | Los datos de pago deben procesarse exclusivamente a través de Stripe (PCI-DSS delegado) | normativa | Cumplimiento PCI-DSS |
| RB-04 | Los datos personales del consumidor deben cumplir con GDPR y regulaciones locales | normativa | Cumplimiento GDPR |

## 13. Supuestos, restricciones y dependencias

- **Supuestos**:
  - El consumidor tiene acceso a internet móvil y un smartphone compatible.
  - Stripe, Google Maps, SendGrid y Twilio mantendrán sus SLAs actuales.
  - El equipo de desarrollo conoce Java/Spring Boot y puede adoptar nuevas tecnologías gradualmente.
- **Restricciones**:
  - Tecnología core: Java/Spring Boot preferido para reutilizar conocimiento del equipo.
  - Migración incremental: el monolito no se reemplaza de golpe (Strangler Fig durante 18–24 meses).
  - Cumplimiento: PCI-DSS delegado a Stripe; GDPR para datos de consumidores.
  - Presupuesto: infraestructura cloud con escalado horizontal (CAPEX/OPEX a definir).
- **Dependencias**:
  - Stripe: pasarela de pago (cobros, reembolsos, conciliación).
  - Google Maps: geocoding, optimización de rutas, tracking en tiempo real.
  - SendGrid: notificaciones email transaccionales.
  - Twilio: notificaciones SMS y push.
  - Monolito legacy: debe coexistir durante la migración.

## 14. Alcance de negocio

### 14.1 En alcance
- Toma de pedidos (Order Taking) con validación, cálculo de total y confirmación.
- Gestión de tickets de cocina (Order Fulfillment/Kitchen) con aceptación/rechazo por el restaurante.
- Asignación de delivery a couriers con rutas optimizadas y tracking.
- Facturación y pagos (Billing & Accounting) con Stripe.
- Notificaciones transaccionales (email, SMS, push).
- Registro y gestión de consumidores y restaurantes.

### 14.2 Fuera de alcance
- Migración completa del monolito a microservicios (se realiza en fases, Strangler Fig 18–24 meses). Este BRD cubre la arquitectura objetivo, no la ejecución de la migración completa.
- Desarrollo de funcionalidades nuevas no contempladas en las 7 capacidades de negocio del Cap. 2 del libro.
- Integración con sistemas de pago distintos a Stripe.
- Soporte offline para la app del consumidor.

## 15. Beneficios esperados y *business case* resumido

| Tipo | Año 1 | Año 2 | Año 3 |
|------|-------|-------|-------|
| Ahorro operativo | USD 50 000 (infraestructura eficiente) | USD 120 000 | USD 200 000 |
| Ingresos adicionales | USD 30 000 (menos caídas = más pedidos) | USD 80 000 | USD 150 000 |
| Inversión (CAPEX) | USD 200 000 (equipos, infraestructura) | USD 100 000 | USD 50 000 |
| Costo operación (OPEX) | USD 60 000 (cloud, herramientas) | USD 70 000 | USD 80 000 |
| **VAN** | | | USD 87 000 (estimado, tasa 10 %) |
| **TIR** | | | 28 % (estimado) |

## 16. Riesgos de negocio

| Riesgo | Probabilidad | Impacto | Mitigación | Responsable |
|--------|--------------|---------|------------|-------------|
| Baja adopción de la nueva arquitectura por parte del equipo de desarrollo | media | alto | Capacitación continua, pair programming, documentación clara | Tech Lead |
| Degradación de rendimiento durante la migración (coexistencia monolito + microservicios) | alta | alto | Estrategia Strangler Fig con feature toggles, monitoreo continuo, rollback plan | Arquitecto |
| Dependencia externa crítica (Stripe, Google Maps) sin SLA garantizado | baja | alto | Cola de retry, degradación graceful con caché local | Equipo de infraestructura |
| Desvío del cronograma de migración (18–24 meses) | media | medio | Hitos trimestrales, MVP por capacidad de negocio, priorización MoSCoW | PM |

## 17. Criterios de éxito del proyecto de negocio

- Cumplimiento ≥80 % de los objetivos SMART (BO-01 a BO-04).
- *Business case* positivo al año 1 con VAN positivo a 3 años.
- Satisfacción del sponsor ≥4/5.
- Cero incidentes críticos por acoplamiento entre módulos durante la migración.
- Métricas de rendimiento (KPI-01, KPI-02, KPI-03) dentro de los umbrales definidos.

## 18. Trazabilidad a documentos hijos

| BRD ID | MRD relacionado | PRD relacionado | Caso de uso FSD |
|--------|-----------------|-----------------|-----------------|
| BR-001 | MRD-N-01 | PRD-REQ-01 | FSD-UC-01 |
| BR-002 | MRD-N-01 | PRD-REQ-02 | FSD-UC-02 |
| BR-003 | MRD-N-02 | PRD-REQ-03 | FSD-UC-03 |
| BR-004 | MRD-N-03 | PRD-REQ-04 | FSD-UC-04 |
| BR-005 | MRD-N-03 | PRD-REQ-05 | FSD-UC-05 |
| BR-006 | MRD-N-02 | PRD-REQ-06 | FSD-UC-06 |
| BR-010 | MRD-N-04 | PRD-REQ-07 | — (NFR transversal) |

## 19. Aprobaciones

| Rol | Nombre | Firma | Fecha |
|-----|--------|-------|-------|
| Sponsor | Dirección de Producto FTGO | | |
| PM | Gerente de Proyectos FTGO | | |
| Arquitecto | Equipo de Arquitectura FTGO | | |

## 20. Registro de cambios

| Versión | Fecha | Autor | Cambio |
|---------|-------|-------|--------|
| v0.1 | 24/05/2026 | Equipo de Arquitectura FTGO | Versión inicial |
