# Examen Laboratorio M4 – Caso FTGO (Food To Go)

Migración de arquitectura monolítica a microservicios usando el caso FTGO del libro *Microservices Patterns* (Richardson, Manning 2019).

## Estructura del repositorio

```
/
├── README.md                      # Este archivo
├── docs/
│   ├── brd.md                     # BRD (Business Requirements Document)
│   ├── PRD.md                     # PRD (Product Requirements Document) ligero
│   ├── FSD.md                     # FSD (Functional Specification Document) con ≥5 UCs
│   ├── adr/
│   │   ├── 0001-estilo-arquitectonico.md   # ADR 1: Estilo arquitectónico
│   │   └── 0002-mecanismo-IPC.md           # ADR 2: Mecanismo IPC
│   ├── diagrams/
│   │   ├── c4_context.mmd         # Diagrama C4 Nivel 1 (Contexto)
│   │   └── c4_container.mmd       # Diagrama C4 Nivel 2 (Contenedores)
│   ├── PROMT_MAPPING.md           # Mapeo de prompts utilizados
│   ├── adr/                       # ADRs adicionales
│   └── diagrams/                  # Diagramas adicionales
├── prompts_mejorados/
│   ├── prd_mejorado.md            # Prompt PRD mejorado (v0.2)
│   └── fsd_mejorado.md            # Prompt FSD mejorado (v0.2)
├── plantillas/
│   └── BRD_TEMPLATE.md            # Plantilla de BRD del módulo
├── PracticaM4.docx                # Brief del caso FTGO (Anexo A)
├── PracticaExamenM4.docx          # Enunciado del examen laboratorio M4
└── Consigna.docx                  # Consigna del módulo
```

## Trazabilidad

| Artefacto | Fuente | Depende de |
|-----------|--------|------------|
| BRD | Brief [§A.1–A.4] | — |
| PRD | Brief + Richardson Cap. 1-2 | BRD |
| FSD | PRD + Brief [§A.5] | PRD |
| ADR-0001 | PRD (NFRs) + Brief [§A.4] | PRD, FSD |
| ADR-0002 | ADR-0001 + Brief [§A.4] + Richardson Cap. 3 | ADR-0001 |
| C4 Context | Brief [§A.2] + PRD (stakeholders) | PRD |
| C4 Container | ADRs + PRD (capacidades) | ADR-0001, ADR-0002 |

## Comandos para invocar prompts

Los siguientes comandos pueden usarse con un agente de IA que soporte la sintaxis `@archivo`:

```bash
# Generar BRD para FTGO
@plantillas/BRD_TEMPLATE.md genera BRD para FTGO usando los datos del brief en PracticaExamenM4.docx

# Generar PRD para FTGO
@prompts_mejorados/prd_mejorado.md genera PRD para FTGO usando el brief del Anexo A como entrada

# Generar FSD para FTGO
@prompts_mejorados/fsd_mejorado.md genera FSD para FTGO a partir de docs/PRD.md y el brief del Anexo A

# Generar ADR (estilo arquitectónico)
@prompts_mejorados/adr_mejorado.md genera ADR para FTGO sobre "estilo arquitectónico" usando PRD+FSD+brief

# Generar ADR (mecanismo IPC)
@prompts_mejorados/adr_mejorado.md genera ADR para FTGO sobre "mecanismo IPC" usando PRD+FSD+brief

# Generar diagramas C4
@prompts_mejorados/c4_mejorado.md genera diagramas C4 nivel 1 y 2 para FTGO
```

## Métricas reportadas

### Prompt PRD (prd_mejorado.md)

| Indicador | v0.1-seed | v0.2-improved | Mejora |
|-----------|-----------|---------------|--------|
| Completitud de secciones | 73 % | 100 % | +27 % |
| NFRs con métrica + origen | 67 % | 100 % | +33 % |
| Stakeholders correctos | 72 % | 100 % | +28 % |
| Capacidades cubiertas | 76 % | 100 % | +24 % |

*Basado en 3 corridas por versión con el mismo modelo. Ver detalle en `prompts_mejorados/prd_mejorado.md`.*

### Prompt FSD (fsd_mejorado.md)

| Indicador | v0.1-seed | v0.2-improved | Mejora |
|-----------|-----------|---------------|--------|
| UCs generados (promedio) | 4.3 | 5.7 | +1.4 UCs |
| UCs con Given/When/Then | 77 % | 100 % | +23 % |
| UCs con trazabilidad | 77 % | 100 % | +23 % |
| UCs con 8 campos completos | 54 % | 100 % | +46 % |

*Basado en 3 corridas por versión con el mismo modelo. Ver detalle en `prompts_mejorados/fsd_mejorado.md`.*

## Resumen de artefactos

| # | Artefacto | Ruta | Mínimo | Estado |
|---|-----------|------|--------|--------|
| 1 | PRD ligero | `docs/PRD.md` | 5 secciones, ≥5 NFRs | ✅ 5 secciones, 10 NFRs |
| 2 | FSD ligero | `docs/FSD.md` | ≥5 UCs con GWT | ✅ 6 UCs con GWT |
| 3 | ADR 1 (estilo arquitectónico) | `docs/adr/0001-*.md` | Opciones, decisión, consecuencias | ✅ 4 opciones, decisión, consecuencias +/– |
| 4 | ADR 2 (IPC) | `docs/adr/0002-*.md` | Opciones, decisión, consecuencias | ✅ 4 opciones, decisión, consecuencias +/– |
| 5 | C4 Nivel 1 (Context) | `docs/diagrams/c4_context.mmd` | Mermaid C4Context válido | ✅ 4 personas, 5 sistemas externos |
| 6 | C4 Nivel 2 (Container) | `docs/diagrams/c4_container.mmd` | Mermaid C4Container válido | ✅ 7 contenedores, relaciones con protocolo |
| 7 | Prompt mejorado 1 | `prompts_mejorados/prd_mejorado.md` | 5 requisitos D4 | ✅ TODOs, Anti-patterns, Verification, Changelog, Métrica |
| 8 | Prompt mejorado 2 | `prompts_mejorados/fsd_mejorado.md` | 5 requisitos D4 | ✅ TODOs, Anti-patterns, Verification, Changelog, Métrica |
| 9 | README | `README.md` | Estructura, comandos, métricas | ✅ Completo |

## Referencias

- Richardson, C. (2019). *Microservices Patterns*. Manning Publications.
- Repositorio oficial FTGO: https://github.com/microservices-patterns/ftgo-application
- Microservices Pattern Language: https://microservices.io/
