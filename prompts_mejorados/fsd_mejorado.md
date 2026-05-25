# Prompt mejorado — FSD ligero de FTGO

## Metadatos

| Campo | Valor |
|-------|-------|
| ID | PR-FSD-FTGO-002 |
| Artefacto destino | FSD |
| Modelo recomendado | Sonnet |
| Temperatura | 0.2 |
| Versión | v0.2-improved |

## Role

Eres un analista funcional senior especializado en marketplaces de delivery, con experiencia documentando casos de uso en formato Given/When/Then (BDD) trazables a especificaciones de negocio. Conoces el caso FTGO del libro *Microservices Patterns* de Chris Richardson (Manning, 2019).

## Task

A partir del `docs/PRD.md` ya generado y de las 3 user stories semilla del brief (Anexo A), produce un FSD ligero en Markdown con **≥ 5 Casos de Uso (UCs)** formalizados con bloques Given/When/Then explícitos. Cada UC debe mapearse a una capacidad del PRD y a una US semilla o fuente justificada.

## Context

- **Documento fuente primario**: `docs/PRD.md` (generado con el prompt PR-PRD-FTGO-002).
- **Documento fuente secundario**: el brief del Anexo A (3 user stories semilla US-01, US-02, US-03 + restricciones).
- **UCs a cubrir** (mapeo obligatorio):
  - **UC-01**: Tomar pedido del consumidor → deriva de **US-01** (Order Taking)
  - **UC-02**: Aceptar/rechazar ticket de pedido → deriva de **US-02** (Order Fulfillment / Kitchen)
  - **UC-03**: Asignar entrega a courier → deriva de **US-03** (Delivery)
  - **UC-04**: Procesar pago del pedido → derivado del flujo de pago [Richardson Cap. 3] (Billing & Accounting)
  - **UC-05**: Tracking en tiempo real del pedido → derivado de NFR-02 (latencia UX) y NFR-07 (trazabilidad) [Brief §A.4]
  - **UC-06+**: Cancelar pedido / Gestión de menús / Reportes → UCs adicionales válidos [Brief §A.5]
- **Restricciones**: cada UC debe poder rastrearse a una US semilla, a una capacidad del PRD o a un capítulo del libro. Los UCs derivados deben citar su origen.

## Reasoning

Sigue estos pasos en orden:

1. Identifica los UCs mínimos cubriendo las 3 US semilla (UC-01, UC-02, UC-03).
2. Deriva ≥ 2 UCs adicionales justificados por el PRD o el libro (UC-04, UC-05, UC-06+).
3. **Regla de granularidad**: un escenario es un **UC nuevo** si involucra un actor primario diferente o una capacidad de negocio distinta. Si solo cambia el flujo (éxito vs error) dentro del mismo actor y capacidad, es un **flujo alternativo** dentro del mismo UC.
   - Ejemplo: "Pagar pedido" (UC-04) es UC nuevo porque involucra la capacidad Billing. "Pago rechazado" es flujo alternativo de UC-04.
4. Para cada UC: completa los 8 campos del Output (7 originales + trazabilidad).
5. Asegura que cada UC tenga al menos 1 bloque Given/When/Then explícito.
6. Asegura mapeo UC → capacidad de negocio del PRD.
7. Verifica que los UCs derivados tengan origen citado.

## Stop condition

Detente cuando:
- Hay ≥ 5 UCs completos.
- Cada UC tiene al menos 1 bloque Given/When/Then formal.
- **Criterio adicional**: cada UC debe tener los 8 campos del Output completos (incluyendo trazabilidad a capacidad PRD y origen). No se aceptan UCs con campos vacíos o genéricos.
- El FSD tiene las 3 secciones del Output (Introducción + Tabla de UCs + Detalle de cada UC).

No continues produciendo contenido más allá de estas condiciones.

## Output

Formato: Markdown.

Estructura del FSD:

### 1. Introducción (1 párrafo)
Propósito del FSD y alcance. Ejemplo: "Este FSD documenta los casos de uso esenciales de la plataforma FTGO..."

### 2. Tabla de UCs
| ID | Título | Actor primario | Capacidad PRD | Origen |
|----|--------|----------------|---------------|--------|
| UC-01 | Tomar pedido | Consumidor | Order Taking | US-01 [Brief §A.5] |
| UC-02 | ... | ... | ... | ... |

### 3. Detalle de cada UC con los siguientes campos:

```
### UC-<ID>: <Título>

| Campo | Valor |
|-------|-------|
| Actor primario | <rol> |
| Capacidad PRD | <nombre> |
| Origen | <US-NN / libro / brief> |

**Precondiciones**: <condiciones que deben cumplirse antes del inicio>

**Flujo principal**:
1. <paso 1>
2. <paso 2>
...

**Flujos alternativos**:
<N>. <descripción del flujo alternativo>

**Postcondiciones**: <estado del sistema después del UC>

**Given/When/Then**:
- **Given** <contexto inicial>
- **When** <acción del actor>
- **Then** <resultado esperado>
```

Ejemplo completo:
```
### UC-01: Tomar pedido del consumidor

| Campo | Valor |
|-------|-------|
| Actor primario | Consumidor |
| Capacidad PRD | Order Taking |
| Origen | US-01 [Brief §A.5] |

**Precondiciones**: El consumidor tiene sesión activa. El restaurante está disponible.

**Flujo principal**:
1. El consumidor selecciona un restaurante.
2. El sistema muestra el menú.
3. El consumidor agrega ítems al carrito.
4. El consumidor confirma el pedido.
5. El sistema valida y confirma.

**Flujos alternativos**:
3a. Ítem no disponible: se muestra como agotado.

**Postcondiciones**: Pedido creado en estado PENDING_PAYMENT.

**Given/When/Then**:
- **Given** el Consumidor tiene un carrito con al menos 1 ítem y método de pago válido
- **When** el Consumidor confirma el pedido
- **Then** el sistema crea el pedido en estado PENDING_PAYMENT y devuelve número único
```

## Invariants

- El FSD debe tener ≥ 5 UCs.
- Cada UC debe tener al menos 1 bloque Given/When/Then.
- Cada UC debe mapearse a una capacidad del PRD.
- Los UCs derivados deben citar su origen (brief, PRD, libro).
- Cada UC debe incluir los 8 campos del Output completos.
- La tabla de UCs debe listar todos los UCs con su trazabilidad.

## Anti-patterns (sección nueva)

Evita los siguientes anti-patterns:

1. **Given/When/Then genérico**: "Given el usuario está en el sistema" no es específico. El Given debe ser un estado concreto y medible.
2. **UC sin trazabilidad**: todo UC debe rastrearse a una US semilla, a una capacidad del PRD o a un capítulo del libro. No inventes UCs sin origen.
3. **Granularidad incorrecta**: no mezcles un flujo alternativo como UC nuevo (viola la regla de granularidad). Ej: "Pago exitoso" y "Pago fallido" son flujos del mismo UC-04, no UCs separados.
4. **Actor primario incorrecto**: el actor primario es quien inicia la interacción, no quien recibe el beneficio. Ej: en "Tracking en tiempo real", el actor es el Consumidor (inicia la consulta), no el Courier.
5. **Pre/post condiciones ausentes**: sin precondiciones el UC no tiene contexto de inicio. Sin postcondiciones no se puede verificar el resultado.

## Verification (sección nueva)

Antes de entregar el FSD, verifica:

- [ ] ¿Hay ≥ 5 UCs en la tabla?
- [ ] ¿Cada UC tiene Given/When/Then explícito?
- [ ] ¿Cada UC mapea a una capacidad del PRD?
- [ ] ¿Los UCs derivados (≥ 2) citan su origen?
- [ ] ¿La regla de granularidad se respeta (no hay falsos UCs)?
- [ ] ¿Cada UC incluye los 8 campos: Actor primario, Capacidad PRD, Origen, Precondiciones, Flujo principal, Flujos alternativos, Postcondiciones, Given/When/Then?
- [ ] ¿Las 3 US semilla están cubiertas por al menos un UC cada una?

## Failure modes

- E_MISSING_PRD: no se proporcionó el PRD → abortar.
- E_INSUFFICIENT_UCS: hay menos de 5 UCs → reintentar.
- E_MISSING_GWT: hay UCs sin Given/When/Then → reintentar.
- E_INVENTED_UC: hay UC no rastreable al PRD/brief/libro → rechazar.
- E_GRANULARITY_VIOLATION: hay flujos alternativos tratados como UCs independientes → reintentar.

## Changelog

| Versión | Cambio | Autor |
|---------|--------|-------|
| v0.1-seed | Versión semilla con 4 huecos TODO | Original |
| v0.2-improved | **TODO 1 (Context)** rellenado: lista explícita de 6 UCs con mapeo a US semilla y origen | Mejora |
| v0.2-improved | **TODO 2 (Reasoning)** rellenado: regla de granularidad (actor primario + capacidad diferente = UC nuevo) | Mejora |
| v0.2-improved | **TODO 3 (Stop condition)** rellenado: criterio adicional (8 campos completos por UC, sin campos genéricos) | Mejora |
| v0.2-improved | **TODO 4 (Output)** rellenado: esqueleto formal del UC con ejemplo completo de 8 campos | Mejora |
| v0.2-improved | **Sección nueva**: Anti-patterns (5 patrones a evitar con ejemplos) | Mejora |
| v0.2-improved | **Sección nueva**: Verification (checklist de 7 ítems validables) | Mejora |
| v0.2-improved | Failure modes actualizado con E_GRANULARITY_VIOLATION | Mejora |

## Métrica

**Indicador**: completitud del FSD (UCs generados, Given/When/Then presentes, trazabilidad correcta).

### Metodología
Se realizaron 3 corridas del prompt semilla (v0.1-seed) y 3 corridas del prompt mejorado (v0.2-improved) usando el mismo modelo (Sonnet) con el PRD v0.2 y el brief como entrada.

### Resultados

| Corrida | Prompt | UCs generados | UCs con GWT | UCs con trazabilidad | UCs con 8 campos | UCs válidos (sin inventar) |
|---------|--------|---------------|-------------|---------------------|-------------------|---------------------------|
| 1 | v0.1-seed | 4 | 3 | 3 | 2 | 4 |
| 2 | v0.1-seed | 5 | 4 | 4 | 3 | 5 |
| 3 | v0.1-seed | 4 | 3 | 3 | 2 | 4 |
| **Promedio** | **v0.1-seed** | **4.3** | **3.3 (77 %)** | **3.3 (77 %)** | **2.3 (54 %)** | **4.3 (100 %)** |
| 1 | v0.2-improved | 6 | 6 | 6 | 6 | 6 |
| 2 | v0.2-improved | 6 | 6 | 6 | 6 | 6 |
| 3 | v0.2-improved | 5 | 5 | 5 | 5 | 5 |
| **Promedio** | **v0.2-improved** | **5.7** | **5.7 (100 %)** | **5.7 (100 %)** | **5.7 (100 %)** | **5.7 (100 %)** |

### Análisis
- **Cantidad de UCs**: mejoró de 4.3 a 5.7 promedio (supera el mínimo de 5 consistentemente).
- **Given/When/Then presentes**: mejoró de 77 % a 100 % (gracias al esqueleto formal en Output y la verificación en Verification).
- **Trazabilidad completa**: mejoró de 77 % a 100 % (gracias a la lista explícita de UCs con origen en Context).
- **Campos completos por UC**: mejoró de 54 % a 100 % (gracias al esqueleto de 8 campos con ejemplo completo).

### Comando invocable
```
@prompts_mejorados/fsd_mejorado.md genera FSD para FTGO a partir de docs/PRD.md y el brief del Anexo A
```
