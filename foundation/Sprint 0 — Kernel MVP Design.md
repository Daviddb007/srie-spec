# Sprint 0 — SRIE Kernel MVP: Module Loader, Identity, CLI

**Fecha:** 2026-07-15
**Estado:** Aprobado
**Autor:** Daviddb007 / SRIE Lab

---

## 1. Resumen

Este sprint construye el Kernel mínimo de SRIE — no como un conjunto de Engines, sino como un **sistema operativo de proyectos** con un Kernel minimalista, módulos auto-descriptivos, y una CLI que define el lenguaje del sistema.

La premisa central: **primero se estabilizan los contratos mediante evidencia, y solo después se congelan las fronteras arquitectónicas**.

---

## 2. Estructura de repositorios

### 2.1 srie-spec (repositorio independiente)

La Constitución de SRIE. Contenido exclusivamente documental. Muy estable, versionado semánticamente.

```
srie-spec/
├── SDOS/                     # 64+ documentos de especificación
├── CONSTITUTION.md           # Filosofía y principios fundacionales
├── CHARTER.md                # Charter del ecosistema
└── VERSION
```

Evolución: lentamente, mediante PRs con revisión.

### 2.2 srie-runtime (repositorio independiente)

El corazón ejecutable. Contiene Kernel, SDK, CLI, servicios de infraestructura, y módulos cargables. Todo junto hasta que los contratos se estabilicen en 1.0.

```
srie-runtime/
├── kernel/                   # Arranque del sistema (servicios del kernel)
│   ├── identity.py             Quién soy (whoami, validate, chain)
│   ├── registry.py             Qué existe (entidades, dominios, capacidades)
│   ├── manifest.py             Snapshot del estado (runtime.lock)
│   ├── runtime.py              Bootloader y ciclo de vida
│   └── lifecycle.py            STARTING → RUNNING → PAUSED → STOPPING → STOPPED
│
├── services/                 # Infraestructura
│   ├── persistence/           Repositorios — los módulos nunca escriben
│   │   ├── graph_repository.py
│   │   ├── registry_repository.py
│   │   ├── twin_repository.py
│   │   └── report_repository.py
│   └── loader/                Module Loader
│       ├── loader.py           Lee module.yaml, resuelve DAG, carga módulos
│       └── resolver.py         Resuelve dependencias entre módulos
│
├── modules/                  # Módulos auto-descriptivos (cada uno con module.yaml)
│   ├── discovery/
│   │   ├── module.yaml         id, version, depends_on, provides, produces, consumes
│   │   └── discovery.py        → produce DiscoveryResult (objeto de dominio)
│   ├── indicators/
│   │   ├── module.yaml
│   │   └── indicators.py       → produce IndicatorReport (objeto de dominio)
│   └── planner/
│       ├── module.yaml
│       └── planner.py          → produce Plan (objeto de dominio)
│
├── sdk/                      # API pública
│   ├── client.py              sdk.identity(), sdk.init(), sdk.discover(), ...
│   ├── models.py              Project, Identity, DiscoveryResult, DigitalTwin, ...
│   └── __init__.py
│
├── cli/                      # El lenguaje del sistema
│   ├── main.py                entry point: srie
│   └── commands/
│       ├── init.py
│       ├── identity.py
│       ├── discover.py
│       ├── indicators.py
│       ├── diagnose.py
│       ├── planner.py
│       └── runtime.py         status, modules, lock (modo debug)
│
├── compiler/                 # Placeholder (futuro)
├── pyproject.toml
└── README.md
```

### 2.3 stonelytics-studio (repositorio independiente)

Producto piloto. Consume srie-runtime como dependencia local.

```
stonelytics-studio/
├── app.py                      # Flask application factory
├── modules/
│   └── studio/
│       ├── routes.py           # Blueprint con salas de control
│       ├── mission_control.py  # Dashboard principal
│       ├── knowledge.py        # Knowledge Center
│       ├── runtime_view.py     # Runtime panel
│       ├── deployment.py       # Deployment panel
│       ├── engineering.py      # Engineering panel
│       └── observatory.py      # Observatory panel
├── templates/
├── static/
├── requirements.txt
└── srie-runtime/ → (dependencia local)
```

### 2.4 srie-domains (pospuesto)

Se crea cuando existan al menos 3 dominios reales (por ejemplo, Backend, Marketing, Producto) y se haya identificado un patrón común.

---

## 3. Principios arquitectónicos

### 3.1 Identity y Registry son servicios del Kernel, no módulos

Identity responde "¿quién soy?" antes de que exista cualquier módulo. Sin Identity, el sistema no puede arrancar. Registry es infraestructura pura (almacena qué entidades existen). Ambos viven en `kernel/`.

### 3.2 Los módulos son auto-descriptivos

Cada módulo tiene un `module.yaml` que declara:

```yaml
id: discovery
version: "1.0.0"
entrypoint: discovery.py
kind: module

depends_on:
  - registry
  - identity

provides:
  - discovery

produces:
  - type: DiscoveryResult
    fields: [languages, frameworks, files, git, dependencies, graph]

consumes:
  - type: Project
    via: project_path
```

El Module Loader escanea `modules/*/module.yaml`, resuelve dependencias como DAG, verifica ciclos, y carga en orden topológico. El Kernel **no conoce** los módulos individualmente.

### 3.3 Los módulos nunca escriben archivos

Un módulo produce **objetos de dominio**:

```python
class DiscoveryResult:
    languages: list[Language]
    frameworks: list[Framework]
    files: FileInventory
    git: GitState | None
    dependencies: list[Dependency]
    graph: Graph
    confidence: float
```

Una capa de persistencia (`services/persistence/`) decide cómo almacenar:

- `graph_repository.save(DiscoveryResult.graph)` → GRAPH.md, GRAPH.json, o Neo4j
- `twin_repository.update(DiscoveryResult)` → SRIE_DIGITAL_TWIN.json
- `report_repository.save(IndicatorReport)` → SRIE_REPORT.md

Esto desacopla completamente producción de conocimiento de su formato de almacenamiento.

**Formato por defecto:** YAML/JSON plano dentro de `SDOS/`. Las implementaciones concretas de cada repositorio (graph → GRAPH.md + GRAPH.json, twin → SRIE_DIGITAL_TWIN.json, report → SRIE_REPORT.md) son intercambiables sin modificar los módulos.

### 3.4 El Module Loader es el primer componente

Antes que Discovery, antes que Indicators, se construye el Module Loader. Porque:
- Discovery será un módulo cargado por el Loader
- Indicators será un módulo cargado por el Loader
- Planner, Marketing, Finance, Legal serán módulos cargados por el Loader
- El ecosistema crece por composición, no por modificación del Kernel

---

## 4. Boot sequence

```
srie init
    │
    ▼  CLI → SDK.init(project_path)
    │
    ▼  Kernel.runtime.boot()
    │
    ├── 1. kernel.identity.ensure()
    │       └── ¿Existe SDOS/IDENTITY.yaml?
    │             ├── NO  → Crear identity (domain_id, name, type, fingerprint)
    │             └── SÍ  → Validar cadena de firmas
    │
    ├── 2. kernel.registry.init()
    │       └── SDOS/REGISTRY.yaml — entidades, dominios, capacidades
    │
    ├── 3. kernel.manifest.create()
    │       └── SDOS/runtime.lock — snapshot del estado inicial
    │
    ├── 4. services.loader.load()
    │       └── Escanear modules/*/module.yaml
    │             ├── Resolver dependencias → DAG
    │             ├── Verificar ciclos
    │             └── Cargar en orden topológico
    │
    ├── 5. Module: discovery.scan(project)
    │       ├── Detectar lenguajes, frameworks, archivos, git, dependencias
    │       └── → return DiscoveryResult (objeto de dominio)
    │
    ├── 6. Persistence: graph_repository.save(result.graph)
    ├── 7. Persistence: twin_repository.update(result → DigitalTwin)
    │
    ├── 8. Module: indicators.calculate(twin)
    │       └── → return IndicatorReport (objeto de dominio)
    │
    ├── 9. Persistence: report_repository.save(report)
    ├── 10. Persistence: twin_repository.update_scores(report)
    │
    └── 11. kernel.manifest.update()
            └── runtime.lock actualizado con nuevo estado
```

### Estados del Runtime

```
STOPPED
    │
    ▼
STARTING ── kernel.identity → kernel.registry → loader
    │
    ▼
RUNNING ─── Módulos cargados, sistema operativo
    │
    ├──► PAUSED ── mantiene estado en memoria
    │       │
    │       ▼
    │   RUNNING
    │
    └──► FAILED ── error crítico
    │
    ▼
STOPPING ── guardar estado, cerrar módulos
    │
    ▼
STOPPED
```

---

## 5. Domain Objects (sdk/models.py)

```python
@dataclass
class Project:
    path: Path
    identity: Identity | None = None
    graph: Graph | None = None
    twin: DigitalTwin | None = None
    registry: Registry | None = None
    timeline: Timeline | None = None
    indicators: IndicatorReport | None = None
    manifest: Manifest | None = None

@dataclass
class Identity:
    domain_id: str
    name: str
    type: str                    # standalone | domain | federation
    version: str
    state: str                   # PENDING | ACTIVE | SUSPENDED | REVOKED | EXPIRED
    organization: dict | None
    authority: dict              # nivel, permisos
    fingerprint: str
    created: datetime
    updated: datetime

@dataclass
class DiscoveryResult:
    languages: list[Language]
    frameworks: list[Framework]
    files: FileInventory
    git: GitState | None
    dependencies: list[Dependency]
    graph: Graph
    confidence: float

@dataclass
class Graph:
    nodes: list[Node]
    relationships: list[Relationship]
    events: list[Event]
    version: int
    last_updated: datetime

@dataclass
class DigitalTwin:
    project_id: str
    nodes: list[Node]
    relationships: list[Relationship]
    indicators: IndicatorReport | None
    metrics: dict
    last_sync: datetime
    version: int

@dataclass
class IndicatorReport:
    srie_score: float
    maturity_level: str          # L0 | L1 | L2 | L3 | L4 | L5
    by_domain: dict[str, float]
    confidence: float
    timestamp: datetime
    trends: dict | None

@dataclass
class Manifest:
    runtime_version: str
    state: str
    kernel: dict
    modules: list[ModuleInfo]
    created: datetime
    updated: datetime
    uptime_seconds: int

@dataclass
class Plan:
    objectives: list[str]
    capabilities: list[Capability]
    priorities: list[PrioritizedItem]
    risks: list[Risk]
    created: datetime
```

---

## 6. SDK simplificado

```python
from srie import SDK

sdk = SDK("/path/to/project")

# Init completo (identity → registry → manifest → loader → discovery → indicators)
sdk.init()

# Pasos individuales
identity = sdk.identity()       # → Identity
result   = sdk.discover()       # → DiscoveryResult
report   = sdk.indicators()     # → IndicatorReport
plan     = sdk.plan()           # → Plan

# Estado
twin     = sdk.twin()           # → DigitalTwin
graph    = sdk.graph()          # → Graph
manifest = sdk.manifest()       # → Manifest (runtime.lock)
```

---

## 7. CLI en dos modos

```
Modo usuario:
  srie init                  → Init completo, salida limpia
  srie identity              → Mostrar identidad actual
  srie discover              → Solo discovery
  srie indicators            → Solo indicadores
  srie plan                  → Generar plan

Modo debug:
  srie init --verbose        → Cada paso detallado, tiempos, confianza
  srie runtime status        → Estado del kernel y módulos
  srie runtime modules       → Lista de módulos cargados con versiones
  srie runtime lock          → Contenido de runtime.lock
  srie inspect --verbose     → Inspección profunda del proyecto
```

---

## 8. Studio Mission Control (salas de control)

| Sala | Contenido | Blueprint |
|------|-----------|-----------|
| **Mission Control** | Health general, SRIE Score, módulos activos, última actividad | `mission_control.py` |
| **Knowledge Center** | Graph, Ontología, Casos, Memoria | `knowledge.py` |
| **Runtime** | Kernel, Módulos, Resources, Policies | `runtime_view.py` |
| **Deployment** | Versiones, Ambientes, Rollback, Health | `deployment.py` |
| **Engineering** | Planner, Capabilities, Sandbox, Repair | `engineering.py` |
| **Observatory** | Logs, Metrics, Traces, Timeline | `observatory.py` |

Cada sala es un blueprint de Flask separado. Ninguna sala llama al Runtime directamente — todas consumen exclusivamente el SDK.

---

## 9. Plan de ejecución del Sprint 0

### Fase 1: Fundación (repos + scaffolding)

| Paso | Acción |
|------|--------|
| 1.1 | Crear repo `srie-spec` en GitHub, migrar SDOS/ actual |
| 1.2 | Crear repo `srie-runtime` en GitHub, scaffolding Python |
| 1.3 | Crear repo `stonelytics-studio` en GitHub, migrar Flask App |

### Fase 2: Kernel

| Paso | Acción |
|------|--------|
| 2.1 | `kernel/identity.py` — crear/validar IDENTITY.yaml, whoami |
| 2.2 | `kernel/registry.py` — init, register, resolve |
| 2.3 | `kernel/manifest.py` — runtime.lock create/update/read |
| 2.4 | `kernel/runtime.py` — boot() sequence coordinator |
| 2.5 | `kernel/lifecycle.py` — estados STARTING→RUNNING→STOPPING |

### Fase 3: Module Loader

| Paso | Acción |
|------|--------|
| 3.1 | `services/loader/resolver.py` — leer module.yaml, resolver DAG |
| 3.2 | `services/loader/loader.py` — cargar módulos en orden, inyectar dependencias |

### Fase 4: Módulos

| Paso | Acción |
|------|--------|
| 4.1 | `modules/discovery/` — module.yaml + discovery.py (scanner) |
| 4.2 | `modules/indicators/` — module.yaml + indicators.py (maturity calculator) |
| 4.3 | `modules/planner/` — module.yaml + planner.py (placeholder) |

### Fase 5: Persistencia

| Paso | Acción |
|------|--------|
| 5.1 | `services/persistence/graph_repository.py` |
| 5.2 | `services/persistence/twin_repository.py` |
| 5.3 | `services/persistence/report_repository.py` |
| 5.4 | `services/persistence/registry_repository.py` |

### Fase 6: SDK

| Paso | Acción |
|------|--------|
| 6.1 | `sdk/models.py` — todos los domain objects |
| 6.2 | `sdk/client.py` — SDK público wrapping runtime |

### Fase 7: CLI

| Paso | Acción |
|------|--------|
| 7.1 | `cli/main.py` — entry point con click/typer |
| 7.2 | Comandos: init, identity, discover, indicators, plan |
| 7.3 | Comandos debug: runtime status, modules, lock |

### Fase 8: Studio Integration

| Paso | Acción |
|------|--------|
| 8.1 | Conectar SDK como dependencia local |
| 8.2 | Mission Control dashboard |
| 8.3 | Knowledge Center panel |

---

## 10. Criterios de éxito

- [ ] `srie init` en un directorio vacío crea SDOS/ con IDENTITY.yaml, REGISTRY.yaml, runtime.lock
- [ ] `srie init` en un proyecto existente (Stonelytics) descubre lenguajes, frameworks, archivos
- [ ] `srie discover` produce un DiscoveryResult que se persiste como GRAPH.md + Digital Twin
- [ ] `srie indicators` calcula SRIE Score y nivel de madurez desde el Twin
- [ ] `srie runtime status` muestra estado del kernel y módulos cargados
- [ ] El Module Loader carga 3 módulos (discovery, indicators, planner) en orden correcto
- [ ] Ningún módulo escribe archivos directamente (todo pasa por persistence/)
- [ ] Studio muestra Mission Control con health, score, y twin
- [ ] Los 3 repos existen en GitHub con su scaffolding inicial
