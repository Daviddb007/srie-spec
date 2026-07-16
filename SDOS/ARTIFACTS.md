# ARTIFACTS — Tipos de Artefactos del Sistema SRIE

---

## ¿Qué es este documento?

ARTIFACTS.md define **todos los tipos de artefactos** que existen en el sistema SRIE. Los Engines no producen "archivos". Producen **artefactos**. Un artefacto es un archivo con un tipo, un formato, un ciclo de vida y un propósito definidos.

Este documento es el catálogo de artefactos. Cualquier Engine nuevo debe producir artefactos de los tipos aquí definidos.

---

## ¿Qué es un artefacto?

Un **artefacto** es la unidad de output del sistema SRIE. Es un archivo que:

- Tiene un **tipo** definido (ver catálogo)
- Tiene un **formato** conocido y parseable
- Tiene un **propósito** específico dentro del sistema
- Tiene un **ciclo de vida** (creación, validación, obsolescencia)
- Tiene un **propietario** (el Engine que lo genera)
- Es **inmutable** o **versionado** (depende del tipo)

---

## Catálogo de artefactos

### 1. DOCUMENT

| Campo | Valor |
|-------|-------|
| Tipo | `document` |
| Descripción | Documentación legible por humanos que describe aspectos del proyecto |
| Formato | Markdown (`.md`) |
| Extensión | `.md` |
| Ciclo de vida | Se actualiza, no se reemplaza. Versionado por fecha. |
| Propietario | Discovery Engine, Capability Engine |
| Ejemplos | `README.md`, `ARCHITECTURE.md`, `capabilities/calendar/calendar.md` |

**Estructura:**

```yaml
document:
  titulo: string
  tipo_contenido: string    # especificacion | diseño | guia | referencia
  dominio: string           # ver DOMAINS.md
  ultima_revision: string   # timestamp ISO
  version: integer
  tags: [string]
```

---

### 2. CAPABILITY

| Campo | Valor |
|-------|-------|
| Tipo | `capability` |
| Descripción | Capacidad funcional del sistema como microproyecto autocontenido |
| Formato | Directorio con estructura definida |
| Extensión | Directorio (`capabilities/<nombre>/`) |
| Ciclo de vida | DRAFT → BUILDING → TESTING → STABLE → DEPLOYED → DEPRECATED → ARCHIVED |
| Propietario | Capability Engine |

**Estructura del directorio** (ver CAPABILITY_MANIFEST.md para detalle):

```
capabilities/<nombre>/
├── <nombre>.manifest
├── <nombre>.md
├── <nombre>_design.md
├── <nombre>_api.md              # Opcional
├── <nombre>_tests.md
├── <nombre>_memory.md
├── <nombre>_metrics.md
├── <nombre>_release.md          # Opcional
├── src/
└── tests/
```

---

### 3. EVIDENCE

| Campo | Valor |
|-------|-------|
| Tipo | `evidence` |
| Descripción | Registro detallado de la ejecución de un Engine: qué se hizo, con qué entradas, qué reglas, qué resultado |
| Formato | JSON |
| Extensión | `.json` |
| Ciclo de vida | Inmutable. Una vez generado, no se modifica. |
| Propietario | Cualquier Engine (en etapa EVIDENCE) |
| Ubicación | `SDOS/evidence/<engine>/<ejecucion_id>.json` |

**Estructura:**

```json
{
  "engine": "Discovery Engine",
  "ejecucion_id": "exec_001",
  "input_summary": {},
  "validation_result": {},
  "discovery_result": {},
  "plan": {},
  "execution_result": {},
  "verification_result": {},
  "errores": [],
  "metricas": {
    "tiempo_ejecucion_ms": 2340,
    "acciones_ejecutadas": 12,
    "archivos_generados": 3
  },
  "timestamp_inicio": "2026-07-15T10:00:00Z",
  "timestamp_fin": "2026-07-15T10:00:39Z",
  "estado_final": "SUCCESS"
}
```

---

### 4. LOG

| Campo | Valor |
|-------|-------|
| Tipo | `log` |
| Descripción | Registro secuencial de eventos durante la ejecución de un Engine (no confundir con evidencia, que es un resumen estructurado) |
| Formato | Texto estructurado o JSON Lines |
| Extensión | `.log` o `.jsonl` |
| Ciclo de vida | Append-only. Se agregan líneas, no se modifican. |
| Propietario | Cualquier Engine |
| Ubicación | `SDOS/logs/<engine>/<ejecucion_id>.log` |

**Formato (JSON Lines):**

```json
{"ts":"2026-07-15T10:00:01Z","level":"INFO","engine":"Discovery","msg":"Iniciando escaneo de directorios"}
{"ts":"2026-07-15T10:00:02Z","level":"INFO","engine":"Discovery","msg":"Encontrados 47 archivos en src/"}
{"ts":"2026-07-15T10:00:03Z","level":"WARN","engine":"Discovery","msg":"No se encontró ARCHITECTURE.md"}
{"ts":"2026-07-15T10:00:39Z","level":"INFO","engine":"Discovery","msg":"Escaneo completado"}
```

---

### 5. MEMORY

| Campo | Valor |
|-------|-------|
| Tipo | `memory` |
| Descripción | Conocimiento estructurado del proyecto: decisiones de ingeniería, contexto, evolución |
| Formato | Markdown |
| Extensión | `.md` |
| Ciclo de vida | Append-only. Se agregan entradas, no se modifican. |
| Propietario | Memory Engine |
| Ubicación | `memory/` |

**Subtipos:**

| Subtipo | Ubicación | Contenido |
|---------|-----------|-----------|
| ADR | `memory/ADR/<fecha>-<titulo>.md` | Decisiones de ingeniería |
| Contexto | `memory/context/<iteracion>.md` | Estado del proyecto por iteración |
| Evolución | `memory/evolution.md` | Cambios significativos en el proyecto |
| Índice | `memory/index.md` | Mapa de toda la memoria disponible |

---

### 6. RELEASE

| Campo | Valor |
|-------|-------|
| Tipo | `release` |
| Descripción | Notas de release de una capability o del proyecto completo |
| Formato | Markdown |
| Extensión | `.md` |
| Ciclo de vida | Inmutable. Release publicadas no se modifican. |
| Propietario | Capability Engine, Deployment Engine |
| Ubicación | `capabilities/<nombre>/<nombre>_release.md` o `SDOS/DEPLOY_REPORT.md` |

**Estructura:**

```markdown
# Release v1.2.0

## Fecha
2026-07-15

## Cambios
- [Feature] Nueva funcionalidad X
- [Fix] Corrección de bug Y
- [Perf] Mejora de rendimiento Z

## Breaking changes
- Ninguno

## Upgrade path
- Actualizar dependencias
```

---

### 7. TEST

| Campo | Valor |
|-------|-------|
| Tipo | `test` |
| Descripción | Especificación de tests para una capability o para el proyecto |
| Formato | Markdown |
| Extensión | `.md` |
| Ciclo de vida | Se actualiza cuando cambia la capability asociada |
| Propietario | Capability Engine, Sandbox Engine |
| Ubicación | `capabilities/<nombre>/<nombre>_tests.md` |

**Estructura:**

```markdown
# Tests — Calendar

## Estrategia
Tests unitarios + integración

## Tests unitarios
- test_crear_evento: Verifica creación correcta
- test_eliminar_evento: Verifica eliminación

## Tests de integración
- test_api_calendar: CRUD completo vía API
```

---

### 8. PATCH

| Campo | Valor |
|-------|-------|
| Tipo | `patch` |
| Descripción | Corrección específica aplicada a una capability o al proyecto |
| Formato | Git diff + metadatos |
| Extensión | `.patch` |
| Ciclo de vida | Inmutable. Una vez aplicado, se archiva. |
| Propietario | Repair Engine |
| Ubicación | `SDOS/patches/<fecha>-<descripcion>.patch` |

**Estructura:**

```yaml
patch:
  id: string
  fecha: string
  engine: string
  capability_afectada: string
  descripcion: string
  diff: string                  # Contenido del diff
  verificacion: string          # Resultado de verificación post-patch
```

---

### 9. ADR

| Campo | Valor |
|-------|-------|
| Tipo | `adr` |
| Descripción | Architectural Decision Record: documenta una decisión de ingeniería con contexto, alternativas y justificación |
| Formato | Markdown con frontmatter YAML |
| Extensión | `.md` |
| Ciclo de vida | Inmutable. Una vez registrada, no se modifica. Se puede superceder. |
| Propietario | Memory Engine |
| Ubicación | `memory/ADR/<fecha>-<titulo>.md` |

**Estructura:**

```yaml
---
id: ADR-001
fecha: 2026-07-15
estado: Aceptado                   # Propuesto | Aceptado | Reemplazado | Obsoleto
supercede: null                    # ADR que este reemplaza
supercedido_por: null              # ADR que reemplaza a este
---
# ADR-001: Elección del framework de tests

## Contexto
Necesitamos elegir un framework de tests para el proyecto.

## Decisión
Usaremos pytest.

## Alternativas consideradas
- unittest (nativo, menos flexible)
- nose2 (menos mantenido)

## Justificación
pytest tiene el ecosistema más activo, mejor integración con CI/CD,
y la comunidad más grande.

## Consecuencias
- Los tests deben seguir convenciones pytest
- Se puede usar pytest-cov para cobertura
```

---

### 10. INDICATOR

| Campo | Valor |
|-------|-------|
| Tipo | `indicator` |
| Descripción | Medición de un indicador específico en un momento dado |
| Formato | JSON o YAML |
| Extensión | `.json` o `.yaml` |
| Ciclo de vida | Serie temporal. Cada medición se agrega al historial. |
| Propietario | Indicators Engine |
| Ubicación | `SDOS/evidence/indicators/<fecha>-<dominio>.json` |

**Estructura:**

```json
{
  "indicador": "cobertura_tests",
  "dominio": "desarrollo",
  "valor": 85.0,
  "unidad": "porcentaje",
  "timestamp": "2026-07-15T10:00:00Z",
  "fuente": "pytest-cov",
  "engine": "Indicators Engine",
  "historial": [
    {"fecha": "2026-06-01", "valor": 72.0},
    {"fecha": "2026-07-01", "valor": 80.0}
  ]
}
```

---

### 11. MANIFEST

| Campo | Valor |
|-------|-------|
| Tipo | `manifest` |
| Descripción | Contrato explícito de un componente del sistema: Engine, Capability o Proyecto |
| Formato | YAML |
| Extensión | `.yaml` (Engine, Proyecto) o `.manifest` (Capability) |
| Ciclo de vida | Se actualiza cuando cambia el componente |
| Propietario | El componente mismo |
| Ubicación | `SDOS/PROJECT.yaml`, `engines/*/engine.yaml`, `capabilities/*/*.manifest` |

---

### 12. EVENT

| Campo | Valor |
|-------|-------|
| Tipo | `event` |
| Descripción | Notificación estructurada de que algo ocurrió en el sistema |
| Formato | JSON |
| Extensión | `.json` |
| Ciclo de vida | Inmutable. Una vez registrado, no se modifica. |
| Propietario | Cualquier Engine |
| Ubicación | `SDOS/events/<fecha>/<id>.json` |

---

## Relación entre artefactos

```
                      ┌──────────────┐
                      │   MANIFEST    │
                      │ (contrato)    │
                      └──────┬───────┘
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
    ┌─────▼─────┐    ┌──────▼──────┐    ┌──────▼──────┐
    │ DOCUMENT  │    │ CAPABILITY  │    │   ENGINE    │
    │ (espec)   │    │ (implementa)│    │ (ejecuta)   │
    └─────┬─────┘    └──────┬──────┘    └──────┬──────┘
          │                 │                  │
          │           ┌─────┼─────┐            │
          │           │     │     │            │
          │     ┌─────▼┐ ┌──▼──┐ ┌▼────┐      │
          │     │ TEST │ │RELE │ │PATCH│      │
          │     └──────┘ │ -ASE│ └─────┘      │
          │              └─────┘              │
          │                 │                  │
          └─────────────────┼──────────────────┘
                            │
                      ┌─────▼─────┐
                      │  EVIDENCE  │
                      │  + LOG     │
                      └─────┬─────┘
                            │
                      ┌─────▼─────┐
                      │  MEMORY   │
                      │  + ADR    │
                      └─────┬─────┘
                            │
                      ┌─────▼─────┐
                      │ INDICATOR │
                      └───────────┘
```

---

## Reglas de artefactos

### Regla 1: Todo output es un artefacto

Cualquier archivo que un Engine genere debe ser un artefacto de un tipo definido en este catálogo. No se permiten archivos sin tipo.

### Regla 2: Inmutabilidad de evidencia

Los artefactos de tipo `evidence`, `event`, `adr` y `log` son inmutables. No se modifican después de creados.

### Regla 3: Versionado de contrato

Los artefactos de tipo `manifest` deben ser versionados (semver). Un cambio en el manifest debe incrementar la versión.

### Regla 4: Ciclo de vida explícito

Cada artefacto debe declarar su ciclo de vida. No puede pasar de CREATED a ARCHIVED sin pasar por los estados intermedios.

### Regla 5: Trazabilidad

Cada artefacto debe referenciar el Engine y la ejecución que lo generó.

### Regla 6: Unicidad

Cada artefacto tiene un path único en el sistema de archivos. No puede haber dos artefactos con el mismo path.

### Regla 7: Serialización

Los artefactos deben ser serializables a texto (Markdown, JSON o YAML). No se permiten formatos binarios no documentados.
