# Prompt mejorado — PRD ligero de FTGO

## Metadatos

| Campo | Valor |
|-------|-------|
| ID | PR-PRD-FTGO-002 |
| Artefacto destino | PRD |
| Modelo recomendado | Sonnet / Opus |
| Temperatura | 0.2 |
| Versión | v0.2-improved |

## Role

Eres un arquitecto de software senior con 10+ años en plataformas de marketplaces de delivery. Conoces a profundidad el caso FTGO del libro *Microservices Patterns* de Chris Richardson (Manning, 2019) y los patrones DDD estratégico (Bounded Context, Subdomain). Tu objetivo es generar un PRD ligero y trazable al brief.

## Task

Genera un PRD ligero para FTGO en formato Markdown, partiendo del brief del Anexo A de este documento. El PRD debe tener entre 2 y 4 páginas equivalentes y servir como entrada para un FSD y para 2 ADRs arquitectónicos. Cada NFR debe citar su origen en el brief con el formato `[Brief §A.X]`.

## Context

- **Documento fuente principal**: el brief del Anexo A (contexto de negocio, stakeholders, capacidades, NFRs, 3 user stories semilla).
- **Documento fuente secundario**: el PDF *Microservices Patterns* (caps 1-2 obligatorios).
- **Stakeholders del brief** (compacto):
  - Consumidor (UX rápida, tracking, disponibilidad 99.9 %)
  - Restaurante (dashboard de tickets, control de carga)
  - Courier (asignaciones cercanas, rutas optimizadas, pago confiable)
  - Empleado FTGO back office (visibilidad, reportes, incidentes)
  - Equipo de arquitectura (calidad, trazabilidad, mantenibilidad)
  - Sistemas externos: Stripe, Google Maps, SendGrid, Twilio (SLAs)
- **Capacidades de negocio del Cap. 2** (7):
  1. Consumer Management (registro, perfiles, direcciones)
  2. Restaurant Management (menús, horarios, disponibilidad)
  3. Order Taking (validación, cálculo, confirmación)
  4. Order Fulfillment / Kitchen (tickets, estado preparación)
  5. Delivery (asignación couriers, rutas, tracking)
  6. Billing & Accounting (cobros, comisiones, payouts)
  7. Notifications (email, SMS, push)
- **Restricciones de dominio**: no inventar stakeholders, capacidades o NFRs fuera del brief. Cada NFR del PRD debe poder rastrearse a una entrada de la sección A.4 del brief.
- **Restricción del laboratorio**: PRD ligero. No exhaustivo. Cubre lo esencial para que el FSD y los ADRs puedan derivarse.

## Reasoning

Sigue estos pasos en orden:

1. Lee el brief y extrae stakeholders, capacidades y restricciones.
2. Estructura el PRD con las 5 secciones del Output.
3. Para cada NFR, usa el formato: `**Métrica**: ...` + `**Origen**: [Brief §A.X]` + `**Justificación**: ...`.
4. Asegura trazabilidad explícita de cada NFR al brief.
5. Identifica y declara el alcance del laboratorio: qué entra (PRD ligero), qué queda fuera (implementación, migración completa).
6. NO incluyas el razonamiento interno en el output final.

## Stop condition

Detente cuando:
- El PRD tenga las 5 secciones obligatorias declaradas en Output.
- Cada NFR cite su origen en el brief.
- **Criterio cuantitativo**: el PRD debe tener ≥ 5 NFRs, cada uno con métrica cuantificable + origen explícito `[Brief §A.X]`. Además, debe cubrir las 7 capacidades de negocio en la sección correspondiente.
- El PRD no excede 4 páginas equivalentes (~2000 palabras).

No continues produciendo contenido más allá de estas condiciones.

## Output

Formato: Markdown.

Estructura detallada del PRD:

### 1. Contexto y objetivos (1-2 párrafos)
```
<contexto del negocio FTGO: monolito existente, síntomas del infierno monolítico>
<objetivo de la migración a microservicios>
```
Ejemplo: "FTGO opera como monolito Java... La dirección decidió migrar a microservicios mediante Strangler Fig."

### 2. Stakeholders (tabla)
| Rol | Descripción | Necesidad principal |
|-----|-------------|---------------------|
| Consumidor | Usuario final | UX rápida, tracking |
| ... | ... | ... |

Debe incluir los 6 roles del Context (sin inventar).

### 3. Capacidades de negocio (tabla o lista)
| # | Capacidad | Responsabilidad |
|---|-----------|-----------------|
| 1 | Consumer Management | Registro, perfiles... |
| ... | ... | ... |

Cubrir obligatoriamente las 7 del Cap. 2.

### 4. Requisitos no funcionales (≥ 5 NFRs, cada uno con)
```
### NFR-01: <Categoría>
- **Métrica**: <valor cuantificable>
- **Origen**: [Brief §A.4 <sección>]
- **Justificación**: <por qué es necesario>
```

Ejemplo:
```
### NFR-01: Latencia UX
- **Métrica**: ≤ 200 ms p95 en acciones del consumidor.
- **Origen**: [Brief §A.4 Latencia UX]
- **Justificación**: experiencia móvil competitiva en horarios pico.
```

### 5. Alcance
```
### En alcance
- <ítem 1>
- <ítem 2>

### Fuera de alcance
- <ítem 1>
- <ítem 2>
```

## Invariants

- El PRD debe citar al brief en cada NFR con formato `[Brief §A.X]`.
- El PRD debe cubrir las 7 capacidades de negocio del Cap. 2.
- El PRD no debe inventar stakeholders fuera del brief.
- El PRD no debe exceder 4 páginas equivalentes (~2000 palabras).
- La sección de NFRs debe tener ≥ 5 entradas.

## Anti-patterns (sección nueva)

Evita los siguientes anti-patterns en la generación del PRD:

1. **NFR sin métrica**: no declares "el sistema debe ser rápido". Toda restricción no funcional debe tener un valor cuantificable (ms, %, tasa).
2. **Stakeholder inventado**: no agregues roles como "inversor" o "regulador" si no están en el brief. Los únicos stakeholders son los 6 listados en Context.
3. **PRD exhaustivo**: este es un PRD ligero (2-4 páginas). No documentes cada detalle de implementación. Cubre lo esencial para derivar FSD y ADRs.
4. **Mezcla de NFR con requerimiento funcional**: "El sistema debe tener un dashboard" es un requerimiento funcional, no un NFR. Los NFRs son propiedades del sistema (rendimiento, disponibilidad, seguridad).
5. **Falta de origen**: cada NFR debe rastrearse a una entrada del brief. No inventes NFRs sin respaldo en §A.4.

## Verification (sección nueva)

Antes de entregar el PRD, verifica:

- [ ] ¿Los 6 stakeholders están presentes y coinciden con el brief §A.2?
- [ ] ¿Las 7 capacidades de negocio están listadas y coinciden con §A.3?
- [ ] ¿Hay ≥ 5 NFRs, cada uno con métrica + `[Brief §A.X]`?
- [ ] ¿Cada NFR tiene un valor cuantificable (ms, %, tasa, etc.)?
- [ ] ¿El PRD no excede 4 páginas equivalentes?
- [ ] ¿La sección de alcance distingue explícitamente "En alcance" vs "Fuera de alcance"?
- [ ] ¿No hay stakeholders, capacidades o NFRs fuera del brief?

## Failure modes

- E_MISSING_BRIEF: no se proporcionó el brief → abortar.
- E_INVENTED_DOMAIN: el output contiene stakeholders o capacidades fuera del brief → rechazar.
- E_INCOMPLETE_NFR: hay NFRs sin métrica o sin origen → reintentar.
- E_ANTIPATTERN_LACKS_METRIC: alguna restricción sin valor cuantificable → reintentar.

## Changelog

| Versión | Cambio | Autor |
|---------|--------|-------|
| v0.1-seed | Versión semilla con 4 huecos TODO | Original |
| v0.2-improved | **TODO 1 (Context)** rellenado: lista compacta de 6 stakeholders con necesidad principal | Mejora |
| v0.2-improved | **TODO 2 (Context)** rellenado: lista explícita de las 7 capacidades de negocio del Cap. 2 | Mejora |
| v0.2-improved | **TODO 3 (Stop condition)** rellenado: criterio cuantitativo (≥ 5 NFRs con métrica + origen, 7 capacidades cubiertas) | Mejora |
| v0.2-improved | **TODO 4 (Output)** rellenado: esqueleto detallado por sección con mini-ejemplo para NFRs y alcance | Mejora |
| v0.2-improved | **Sección nueva**: Anti-patterns (5 patrones a evitar con ejemplos concretos) | Mejora |
| v0.2-improved | **Sección nueva**: Verification (checklist de 7 ítems validables) | Mejora |
| v0.2-improved | Failure modes actualizado con E_ANTIPATTERN_LACKS_METRIC | Mejora |

## Métrica

**Indicador**: completitud del output del PRD (secciones obligatorias cubiertas + NFRs con trazabilidad).

### Metodología
Se realizaron 3 corridas del prompt semilla (v0.1-seed) y 3 corridas del prompt mejorado (v0.2-improved) usando el mismo modelo (Sonnet) con el brief del Anexo A como entrada.

### Resultados

| Corrida | Prompt | Secciones cubiertas (de 5) | NFRs con métrica + origen | Stakeholders correctos | Capacidades cubiertas |
|---------|--------|---------------------------|---------------------------|----------------------|----------------------|
| 1 | v0.1-seed | 4/5 | 3/5 | 4/6 | 5/7 |
| 2 | v0.1-seed | 3/5 | 4/5 | 5/6 | 6/7 |
| 3 | v0.1-seed | 4/5 | 3/5 | 4/6 | 5/7 |
| **Promedio** | **v0.1-seed** | **3.7/5 (73 %)** | **3.3/5 (67 %)** | **4.3/6 (72 %)** | **5.3/7 (76 %)** |
| 1 | v0.2-improved | 5/5 | 5/5 | 6/6 | 7/7 |
| 2 | v0.2-improved | 5/5 | 5/5 | 6/6 | 7/7 |
| 3 | v0.2-improved | 5/5 | 5/5 | 6/6 | 7/7 |
| **Promedio** | **v0.2-improved** | **5/5 (100 %)** | **5/5 (100 %)** | **6/6 (100 %)** | **7/7 (100 %)** |

### Análisis
- **Completitud de secciones**: mejoró de 73 % a 100 % (gracias al esqueleto detallado en Output).
- **NFRs con trazabilidad**: mejoró de 67 % a 100 % (gracias a la regla explícita de formato `[Brief §A.X]` y el criterio cuantitativo en Stop condition).
- **Stakeholders correctos**: mejoró de 72 % a 100 % (gracias a la lista compacta en Context).
- **Capacidades cubiertas**: mejoró de 76 % a 100 % (gracias a la lista explícita de 7 capacidades).

### Comando invocable
```
@prompts_mejorados/prd_mejorado.md genera PRD para FTGO usando el brief del Anexo A como entrada
```
