# DISCOVERY OUTPUTS — Artefactos que genera el Discovery Engine

---

## ¿Qué es este documento?

DISCOVERY_OUTPUTS.md define **todos los artefactos** que el Discovery Engine genera durante su ejecución. Cada artefacto tiene un propósito específico dentro del sistema SRIE y es consumido por otros Engines o por humanos.

---

## Catálogo de artefactos

| # | Artefacto | Ruta | Modo | ¿Obligatorio? |
|---|-----------|------|------|---------------|
| 1 | PROJECT_STATE.md | `SDOS/PROJECT_STATE.md` | full | Sí |
| 2 | PROJECT_TIMELINE.md | `SDOS/PROJECT_TIMELINE.md` | full + incremental | Sí |
| 3 | GRAPH.md | `SDOS/GRAPH.md` | full + incremental | Sí |
| 4 | DISCOVERY_REPORT.md | `SDOS/discovery/DISCOVERY_REPORT.md` | full + incremental | Sí |
| 5 | PROJECT_GRAPH.json | `SDOS/discovery/PROJECT_GRAPH.json` | full + incremental | Sí |
| 6 | DISCOVERY_INDEX.md | `SDOS/discovery/DISCOVERY_INDEX.md` | full + incremental | Sí |
| 7 | DISCOVERY_EVIDENCE.json | `SDOS/discovery/DISCOVERY_EVIDENCE.json` | full + incremental | Sí |
| 8 | DISCOVERY_CACHE.yaml | `SDOS/discovery/DISCOVERY_CACHE.yaml` | incremental | Sí |
| 9 | REGISTRY.yaml | `SDOS/REGISTRY.yaml` | full + incremental | Sí (actualización) |

---

## 1. PROJECT_STATE.md

**Ruta:** `SDOS/PROJECT_STATE.md`
**Propósito:** Inventario completo y actualizado del proyecto. Es la fuente de verdad sobre qué existe.
**Consumido por:** Indicators Engine, DX Engine, Planner Engine

### Formato

```yaml
# Project State
ultima_actualizacion: "2026-07-15T10:00:20Z"
modo: "full"
discovery_id: "disc_001"

proyecto:
  nombre: "mi-proyecto"
  estado: "BOOTSTRAPPED"
  nivel_madurez: "L0"

lenguajes:
  - nombre: "Python"
    version: "3.12"
    confianza: 0.99

frameworks:
  - nombre: "FastAPI"
    version: "0.100.0"
    fuente: "pyproject.toml"
    confianza: 0.97

arquitectura:
  patrones:
    - "FastAPI Routers"
    - "SQLAlchemy ORM"
  confianza: 0.78
  nota: "Estructura de routers detectada. No se confirma DDD completo."

documentacion:
  - archivo: "README.md"
    existe: true
    dominio: "business"
  - archivo: "ARCHITECTURE.md"
    existe: false
    dominio: "technical"
  - archivo: "CHANGELOG.md"
    existe: true
    dominio: "documentation"

git:
  detectado: true
  ramas:
    - "main"
    - "develop"
  commits_totales: 47
  autores: ["usuario1", "usuario2"]
  ultimo_commit:
    hash: "abc789"
    mensaje: "feat: add calendar capability"
    fecha: "2026-07-14"

tests:
  detectado: true
  framework: "pytest"
  config: "pyproject.toml"

docker:
  detectado: true
  archivos:
    - "Dockerfile"
    - "docker-compose.yml"

ci_cd:
  detectado: false

variables_entorno:
  archivos:
    - ".env.example"
  variables_detectadas:
    - "DATABASE_URL"
    - "SECRET_KEY"

capabilities:
  total: 1
  lista:
    - nombre: "Calendar"
      path: "capabilities/calendar/"
      estado: "DRAFT"

archivos_totales: 128
archivos_codigo: 85
archivos_configuracion: 12
archivos_documentacion: 5
```

---

## 2. PROJECT_TIMELINE.md

**Ruta:** `SDOS/PROJECT_TIMELINE.md`
**Propósito:** Línea temporal del proyecto basada en git. Responde a: ¿cuándo nació? ¿cuándo cambió? ¿quién cambió qué?
**Consumido por:** DX Engine, Knowledge Engine, humanos

### Formato

```yaml
# Project Timeline
proyecto: "mi-proyecto"
ultima_actualizacion: "2026-07-15T10:00:20Z"

origen:
  fecha: "2026-01-15"
  commit: "a1b2c3"
  autor: "usuario1"
  mensaje: "Initial commit"

hitos:
  - fecha: "2026-01-15"
    tipo: "INICIO"
    descripcion: "Creación del proyecto"
    commit: "a1b2c3"

  - fecha: "2026-02-01"
    tipo: "FRAMEWORK"
    descripcion: "Migración de Flask a FastAPI"
    commit: "d4e5f6"
    autor: "usuario1"
    archivos_afectados: 23

  - fecha: "2026-03-15"
    tipo: "CAPABILITY"
    descripcion: "Implementación de Auth"
    commit: "g7h8i9"
    autor: "usuario2"
    archivos_afectados: 15

  - fecha: "2026-07-14"
    tipo: "CAPABILITY"
    descripcion: "Implementación de Calendar"
    commit: "abc789"
    autor: "usuario2"
    archivos_afectados: 12

linea_temporal_completa:
  - fecha: "2026-01-15"
    eventos:
      - tipo: "commit"
        hash: "a1b2c3"
        autor: "usuario1"
        mensaje: "Initial commit"
  - fecha: "2026-02-01"
    eventos:
      - tipo: "commit"
        hash: "d4e5f6"
        autor: "usuario1"
        mensaje: "refactor: migrate from Flask to FastAPI"

metricas_temporales:
  edad_dias: 181
  commits_por_mes: 7.8
  autores_activos: 2
  periodo_mayor_actividad: "2026-03"
```

---

## 3. GRAPH.md

**Ruta:** `SDOS/GRAPH.md`
**Propósito:** Knowledge Graph actualizado con todos los nodos, relaciones, estados y eventos.
**Consumido por:** Todos los Engines

### Formato

(Ver GRAPH.md para el esquema completo)

El archivo `SDOS/GRAPH.md` es una representación actualizada del grafo. Discovery actualiza:

- Nodos existentes (actualiza `ultima_modificacion`, incrementa `version`)
- Nuevos nodos descubiertos
- Relaciones nuevas
- Estados de nodos (SYNCED / DESYNCED / STALE / ORPHAN)
- Eventos de cambio entre nodos

---

## 4. DISCOVERY_REPORT.md

**Ruta:** `SDOS/discovery/DISCOVERY_REPORT.md`
**Propósito:** Reporte legible del descubrimiento. Resume qué se encontró, qué no, y con qué confianza.
**Consumido por:** Humanos (desarrolladores), DX Engine

### Formato

```markdown
# Discovery Report — disc_001

**Fecha:** 2026-07-15T10:00:20Z
**Modo:** full
**Duración:** 18.4s

## Resumen

- Archivos escaneados: 128
- Entidades descubiertas: 15
- Relaciones encontradas: 23
- Hipótesis confirmadas: 12 / 13
- Confianza promedio: 0.84

## Entidades nuevas

| Entidad | Tipo | Dominio | Confianza |
|---------|------|---------|-----------|
| README.md | Document | business | 0.99 |
| app/main.py | Document | functional | 0.97 |
| Calendar | Capability | functional | 0.95 |

## Entidades no encontradas (esperadas)

| Entidad | Dominio | Impacto |
|---------|---------|---------|
| ARCHITECTURE.md | technical | Alto |
| CHANGELOG.md | documentation | Medio |

## Afirmaciones especulativas

| Afirmación | Confianza | Riesgo |
|------------|-----------|--------|
| Arquitectura hexagonal | 0.35 | 0.65 |

## Recomendaciones

- Crear ARCHITECTURE.md para mejorar gobernanza
- Documentar Calendar capability (falta memory)
```

---

## 5. PROJECT_GRAPH.json

**Ruta:** `SDOS/discovery/PROJECT_GRAPH.json`
**Propósito:** Knowledge Graph en formato JSON para consumo programático por otros Engines.
**Consumido por:** Indicators Engine, Planner Engine, Registry

### Formato

```json
{
  "version": 1,
  "timestamp": "2026-07-15T10:00:20Z",
  "nodes": [
    {"id": "PROJECT", "type": "root", "state": "SYNCED"},
    {"id": "business", "type": "domain", "state": "SYNCED"},
    {"id": "technical", "type": "domain", "state": "PENDING"}
  ],
  "edges": [
    {"source": "PROJECT", "target": "business", "type": "contains"}
  ],
  "metrics": {
    "node_count": 15,
    "edge_count": 23,
    "synced": 12,
    "pending": 2,
    "desynced": 1
  }
}
```

---

## 6. DISCOVERY_INDEX.md

**Ruta:** `SDOS/discovery/DISCOVERY_INDEX.md`
**Propósito:** Índice de todos los descubrimientos realizados. Permite saber cuándo se ejecutó Discovery, en qué modo y qué encontró.
**Consumido por:** Governance Engine, humanos

### Formato

```yaml
# Discovery Index

ejecuciones:
  - id: "disc_001"
    fecha: "2026-07-15T10:00:20Z"
    modo: "full"
    duracion_ms: 18400
    resultado: "SUCCESS"
    entidades_descubiertas: 15
    confianza_promedio: 0.84

  - id: "disc_002"
    fecha: "2026-07-15T11:30:00Z"
    modo: "incremental"
    duracion_ms: 1200
    resultado: "NO_ACTION_NEEDED"
    cambios_desde_anterior: 0

ultimo_discovery: "disc_002"
proximo_discovery_programado: null
```

---

## 7. DISCOVERY_EVIDENCE.json

**Ruta:** `SDOS/discovery/DISCOVERY_EVIDENCE.json`
**Propósito:** Registro completo de toda la evidencia recolectada y las afirmaciones validadas.
**Consumido por:** Memory Engine, Knowledge Engine, auditoría

### Formato

(Ver DISCOVERY_EVIDENCE.md para el formato completo)

---

## 8. DISCOVERY_CACHE.yaml

**Ruta:** `SDOS/discovery/DISCOVERY_CACHE.yaml`
**Propósito:** Caché para ejecución incremental. Almacena el hash del proyecto y metadatos para detectar cambios.
**Consumido por:** Discovery Engine (modo incremental)

### Formato

```yaml
# Discovery Cache
ultima_actualizacion: "2026-07-15T10:00:20Z"
project_hash: "a1b2c3d4e5f6..."
modo_anterior: "full"
archivos:
  - path: "pyproject.toml"
    hash: "abc123"
    ultima_modificacion: "2026-07-14T15:00:00Z"
  - path: "app/main.py"
    hash: "def456"
    ultima_modificacion: "2026-07-14T15:00:00Z"
git:
  ultimo_commit: "abc789"
  branch: "main"
metricas_anteriores:
  archivos_totales: 128
  entidades_descubiertas: 15
  confianza_promedio: 0.84
```

---

## 9. REGISTRY.yaml (actualización)

**Ruta:** `SDOS/REGISTRY.yaml`
**Propósito:** Registry actualizado con nuevas entidades descubiertas.
**Consumido por:** Todos los Engines

### Formato

(Ver REGISTRY.md para el formato completo)

Discovery actualiza el Registry con:

- Nuevas entidades descubiertas (documentos, capabilities, etc.)
- Actualización de versiones de entidades existentes
- Nuevas relaciones registradas

---

## Resumen de artefactos por modo

| Artefacto | FULL | INCREMENTAL | WATCH |
|-----------|------|-------------|-------|
| PROJECT_STATE.md | ✓ Generar completo | ✓ Solo secciones afectadas | Solo si cambios estructurales |
| PROJECT_TIMELINE.md | ✓ Generar completo | ✓ Actualizar | ✓ Actualizar |
| GRAPH.md | ✓ Generar completo | ✓ Actualizar nodos afectados | ✓ Actualizar |
| DISCOVERY_REPORT.md | ✓ Generar | ✓ Generar (resumido) | ✗ (solo eventos) |
| PROJECT_GRAPH.json | ✓ Generar | ✓ Actualizar | Solo si cambios |
| DISCOVERY_INDEX.md | ✓ Actualizar | ✓ Actualizar | ✓ Actualizar |
| DISCOVERY_EVIDENCE.json | ✓ Generar | ✓ Solo nuevas afirmaciones | Solo nuevas |
| DISCOVERY_CACHE.yaml | ✓ Generar | ✓ Actualizar | ✓ Actualizar |
| REGISTRY.yaml | ✓ Actualizar | ✓ Actualizar | ✓ Actualizar |
