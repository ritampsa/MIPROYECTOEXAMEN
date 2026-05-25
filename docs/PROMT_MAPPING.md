# Prompt Mapping

> Mapeo de prompts utilizados en la generación de documentos del módulo.

---

## PR-BRD-001 — Generación de BRD desde plantilla

**Propósito**: Generar un Business Requirements Document (BRD) completo a partir de la plantilla del módulo.

**Modelo destino**: Cualquier agente de IA conversacional.

**Prompt**:

---

Eres un analista de negocio experto en metodologías ágiles y redacción de documentos de requisitos. Tu tarea es generar un **Business Requirements Document (BRD)** completo siguiendo la estructura detallada abajo.

**Contexto**: El BRD es el primer documento de la cadena `BRD → MRD → PRD → FSD → DTI`. Debe formalizar las **necesidades y restricciones de negocio**, respondiendo a **"¿qué necesita el negocio y por qué?"**, sin entrar en soluciones técnicas.

**Instrucciones generales**:
- Rellena TODAS las secciones numeradas (0–20).
- Usa marcadores `<...>` como guía; reemplázalos con contenido concreto.
- Usa datos ficticios pero realistas y coherentes entre secciones.
- Mantén un tono profesional, lenguaje de negocio (no técnico).
- Asegura trazabilidad entre IDs (BR-001, KPI-01, BO-01, etc.).
- Si no hay información para una sección, indícalo explícitamente.

---

### 0. Metadatos

Genera una tabla con: Producto, Grupo, Versión (v0.1), Fecha, Sponsor de negocio, Stakeholders, Autores, Revisores, Estado (Borrador), Insumo del Módulo Anterior, Prompts utilizados.

### 1. Resumen ejecutivo

Máximo ½ página. Debe incluir: Problema (una frase con evidencia), Propuesta (qué construiremos a alto nivel), Valor esperado (2–3 cifras objetivas), Métricas clave de éxito (2–3 KPIs con meta y horizonte), Llamada a la acción.

### 2. Contexto del negocio

Organización, unidad impactada, proceso(s) afectado(s), estrategia organizacional que justifica el proyecto.

### 3. Problema y oportunidad de negocio

#### 3.1 Problema
150–250 palabras describiendo dolor actual: síntomas, causas raíz, evidencia cuantitativa, consecuencia de no actuar.

#### 3.2 Oportunidad
Valor económico estimado, valor estratégico/reputacional, ventana de oportunidad.

#### 3.3 Evidencia de Continuous Discovery
Documento de Discovery, entrevistas realizadas, hipótesis validadas/refutadas, artefactos M2 (UI/UX), próxima cadencia de Discovery.

### 4. Usuarios objetivo / Personas clave

#### 4.1 Persona principal
Tabla con: Nombre/rol, Contexto, Jobs-to-be-done, Dolores principales, Ganancia esperada.

#### 4.2 Persona secundaria (opcional)
Misma estructura que 4.1.

### 5. Propuesta de valor

Formato Value Proposition Canvas: Para quién, Que necesita, Nuestra propuesta es, Que le aporta (3–5 bullets), A diferencia de, Nuestro diferencial es.

### 6. Panorama competitivo (resumen)

Tabla con: Competidor/alternativa, Tipo (directo/indirecto/do-nothing), Fortaleza percibida, Debilidad percibida. Mínimo 3 filas incluyendo do-nothing.

### 7. Business Model Canvas

Los 9 bloques de Osterwalder. Cada bloque debe tener **mínimo 3 elementos**: Segmentos de clientes, Propuesta de valor, Canales, Relación con clientes, Fuentes de ingresos, Recursos clave, Actividades clave, Socios clave, Estructura de costos.

### 8. Métricas clave de éxito

Tabla con: ID, KPI, ¿North Star?, Línea base, Meta, Horizonte, Fuente del dato. Mínimo 1 North Star + 2 KPIs de apoyo.

### 9. Objetivos de negocio (SMART)

Tabla con: ID, Objetivo, Métrica, Línea base, Meta, Horizonte. Mínimo 3 objetivos SMART.

### 10. Stakeholders y roles (RACI)

Tabla con: Stakeholder, Interés, R/A/C/I. Mínimo 4 stakeholders.

### 11. Requerimientos de negocio

Tabla con: ID (BR-001...), Requerimiento, Prioridad (MoSCoW), Justificación, Métrica de aceptación. Mínimo 8 requerimientos. NO describir soluciones técnicas.

### 12. Reglas de negocio y políticas

Tabla con: ID (RB-01...), Regla, Tipo (política/normativa), Origen. Mínimo 2 reglas.

### 13. Supuestos, restricciones y dependencias

Lista clara de cada tipo. Incluir marco normativo local (ej. leyes bolivianas si aplica).

### 14. Alcance de negocio

#### 14.1 En alcance
Procesos/áreas incluidas.

#### 14.2 Fuera de alcance
Procesos/áreas excluidas con justificación.

### 15. Beneficios esperados y business case

Tabla Año 1, 2, 3 con: Ahorro operativo, Ingresos adicionales, Inversión (CAPEX), Costo operación (OPEX), VAN, TIR.

### 16. Riesgos de negocio

Tabla con: Riesgo, Probabilidad, Impacto, Mitigación, Responsable. Mínimo 3 riesgos.

### 17. Criterios de éxito del proyecto de negocio

Mínimo 3 criterios medibles.

### 18. Trazabilidad a documentos hijos

Tabla cruzando BRD ID con MRD relacionado, PRD relacionado, Caso de uso FSD.

### 19. Aprobaciones

Tabla con: Rol, Nombre, Firma, Fecha (Sponsor, PM, Arquitecto).

### 20. Registro de cambios

Tabla con: Versión, Fecha, Autor, Cambio.

---

### Checklist de validación

- [ ] Resumen ejecutivo de ½ página con problema, propuesta, valor y métricas
- [ ] Problema de negocio con evidencia cuantitativa
- [ ] 1–2 personas caracterizadas (JTBD, dolores, ganancias)
- [ ] Propuesta de valor explícita (formato VPC)
- [ ] Panorama competitivo con ≥ 3 alternativas (incluyendo do-nothing)
- [ ] Business Model Canvas con 9 bloques, ≥ 3 elementos por bloque
- [ ] Métricas clave: ≥ 1 North Star + 2 KPIs de apoyo, con meta y horizonte
- [ ] ≥ 3 objetivos SMART
- [ ] Matriz RACI completa
- [ ] ≥ 8 requerimientos de negocio priorizados (MoSCoW)
- [ ] Reglas, restricciones, supuestos y dependencias explícitos
- [ ] Business case cuantitativo (aunque sea estimado)
- [ ] Trazabilidad a MRD/PRD iniciada
