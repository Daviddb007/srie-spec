# REGISTRY — Catálogo Central del Ecosistema SRIE

---

## ¿Qué es este documento?

REGISTRY.md define el **Registro Maestro** de SRIE. No es un Engine. Es una **infraestructura base** — un catálogo central donde toda entidad del ecosistema tiene un identificador único, una versión, un propietario, un estado y sus dependencias.

El Registry es la fuente de verdad para saber **qué existe** en el sistema. Ningún Engine, Capability o Artefacto existe oficialmente hasta que está registrado.

---

## ¿Por qué existe?

Sin un Registry, cada Engine termina construyendo su propio inventario. Discovery descubre archivos, pero no sabe si ya fueron registrados. Capability Engine crea capacidades, pero no hay un índice central.

El Registry resuelve:

- **Trazabilidad**: cada entidad tiene un ID único y su historial completo
- **Detección de duplicados**: no pueden existir dos entidades con el mismo ID
- **Gestión de dependencias**: cada entidad declara de qué depende
- **Gobierno**: cada entidad tiene un estado, un propietario y un ciclo de vida
- **Descubrimiento**: cualquier Engine puede consultar qué existe sin escanear archivos

---

## Formato del Registry

El Registry se materializa en `SDOS/REGISTRY.yaml` (el archivo vivo del proyecto) y se rige por el esquema definido en este documento.

```yaml
# SDOS/REGISTRY.yaml
registry:
  version: "1.0.0"
  ultima_actualizacion: "2026-07-15T10:00:00Z"
  entidades: {}
```

Cada entidad registrada:

```yaml
  <id>:
    tipo: string                    # Tipo de entidad (ver catálogo)
    nombre: string                  # Nombre legible
    version: string                 # Versión semver
    estado: string                  # Estado actual (ver STATES.md)
    propietario: string             # Engine o equipo responsable
    dominio: string                 # Dominio (ver DOMAINS.md)
    descripcion: string             # Breve descripción
    creado: string                  # Timestamp ISO
    actualizado: string             # Timestamp ISO
    tags: [string]                  # Etiquetas de clasificación
    dependencias:
      - id: string                  # ID de la entidad dependiente
        tipo: string                # Tipo de dependencia (requerido | opcional | heredado)
        version: string             # Versión mínima requerida
    documentacion:
      - path: string                # Ruta al documento asociado
        tipo: string                # Tipo de documento
    eventos:
      - tipo: string                # Últimos eventos relevantes
        timestamp: string
```

---

## Catálogo de entidades registrables

### 1. Proyecto

| Campo | Ejemplo |
|-------|---------|
| ID | `PROJECT-001` |
| Tipo | `project` |
| Propietario | — |
| Dominio | `governance` |

```yaml
PROJECT-001:
  tipo: project
  nombre: "Mi Proyecto"
  version: "1.0.0"
  estado: STABLE
  propietario: null
  dominio: governance
  descripcion: "Proyecto principal del sistema"
  tags: ["srie", "production"]
  dependencias: []
  documentacion:
    - path: "SDOS/PROJECT.yaml"
      tipo: manifest
```

---

### 2. Engine

| Campo | Ejemplo |
|-------|---------|
| ID | `ENG-001` |
| Tipo | `engine` |
| Propietario | SRIE Core |
| Dominio | `governance` |

```yaml
ENG-001:
  tipo: engine
  nombre: "Discovery Engine"
  version: "1.0.0"
  estado: REGISTERED
  propietario: "SRIE Core"
  dominio: governance
  descripcion: "Sistema sensorial de SRIE. Observa, interpreta y reporta el estado del proyecto."
  tags: ["core", "sensory"]
  dependencias:
    - id: "ONT-001"
      tipo: requerido
      version: ">=1.0.0"
    - id: "REG-001"
      tipo: requerido
      version: ">=1.0.0"
  documentacion:
    - path: "engines/discovery/engine.yaml"
      tipo: manifest
    - path: "SDOS/ENGINES.md"
      tipo: spec
```

---

### 3. Capability

| Campo | Ejemplo |
|-------|---------|
| ID | `CAP-001` |
| Tipo | `capability` |
| Propietario | Capability Engine |
| Dominio | `functional` |

```yaml
CAP-001:
  tipo: capability
  nombre: "Calendar"
  version: "1.2.0"
  estado: DEPLOYED
  propietario: "Capability Engine"
  dominio: functional
  descripcion: "Gestión de calendario y eventos"
  tags: ["core", "calendar"]
  dependencias:
    - id: "CAP-002"
      tipo: requerido
      version: ">=2.0.0"
  documentacion:
    - path: "capabilities/calendar/calendar.manifest"
      tipo: manifest
    - path: "capabilities/calendar/calendar.md"
      tipo: spec
    - path: "capabilities/calendar/calendar_design.md"
      tipo: design
```

---

### 4. Documento

| Campo | Ejemplo |
|-------|---------|
| ID | `DOC-001` |
| Tipo | `document` |
| Propietario | Discovery Engine |
| Dominio | `technical` |

```yaml
DOC-001:
  tipo: document
  nombre: "ARCHITECTURE.md"
  version: "1"
  estado: CURRENT
  propietario: "Discovery Engine"
  dominio: technical
  descripcion: "Documento de arquitectura del proyecto"
  tags: ["architecture"]
  dependencias: []
  documentacion: []
```

---

### 5. Manifest

| Campo | Ejemplo |
|-------|---------|
| ID | `MAN-001` |
| Tipo | `manifest` |
| Propietario | El componente que describe |
| Dominio | `governance` |

```yaml
MAN-001:
  tipo: manifest
  nombre: "PROJECT.yaml"
  version: "1"
  estado: CURRENT
  propietario: "SRIE Core"
  dominio: governance
  descripcion: "Manifest del proyecto"
  tags: ["manifest", "project"]
  dependencias: []
  documentacion:
    - path: "SDOS/PROJECT.yaml"
      tipo: manifest
```

---

### 6. Artefacto

| Campo | Ejemplo |
|-------|---------|
| ID | `ART-001` |
| Tipo | `artifact` |
| Propietario | El Engine que lo genera |
| Dominio | Variable según el artefacto |

```yaml
ART-001:
  tipo: artifact
  nombre: "PROJECT_STATE.md"
  version: "3"
  estado: CURRENT
  propietario: "Discovery Engine"
  dominio: governance
  descripcion: "Inventario del proyecto generado por Discovery"
  tags: ["state", "discovery"]
  dependencias: []
  documentacion:
    - path: "SDOS/PROJECT_STATE.md"
      tipo: evidence
```

---

### 7. Evento

| Campo | Ejemplo |
|-------|---------|
| ID | `EVT-001` |
| Tipo | `event` |
| Propietario | El Engine que lo emite |
| Dominio | `governance` |

```yaml
EVT-001:
  tipo: event
  nombre: "PROJECT_DISCOVERED"
  version: "1"
  estado: CURRENT
  propietario: "INIT"
  dominio: governance
  descripcion: "Evento de descubrimiento inicial del proyecto"
  tags: ["event", "discovery", "init"]
  dependencias: []
  documentacion:
    - path: "SDOS/EVENTS.md"
      tipo: spec
```

---

### 8. Indicador

| Campo | Ejemplo |
|-------|---------|
| ID | `IND-001` |
| Tipo | `indicator` |
| Propietario | Indicators Engine |
| Dominio | `observability` |

```yaml
IND-001:
  tipo: indicator
  nombre: "srie_score"
  version: "1"
  estado: CURRENT
  propietario: "Indicators Engine"
  dominio: observability
  descripcion: "Métrica compuesta de madurez del proyecto (0-100)"
  tags: ["score", "maturity", "core"]
  dependencias: []
  documentacion:
    - path: "SDOS/SRIE_INDICATORS.md"
      tipo: spec
```

---

### 9. Dominio

| Campo | Ejemplo |
|-------|---------|
| ID | `DOM-001` |
| Tipo | `domain` |
| Propietario | SRIE Core |
| Dominio | `governance` |

```yaml
DOM-001:
  tipo: domain
  nombre: "business"
  version: "1"
  estado: CURRENT
  propietario: "SRIE Core"
  dominio: governance
  descripcion: "Dominio de negocio: propósito, objetivos, stakeholders"
  tags: ["domain", "business"]
  dependencias: []
  documentacion:
    - path: "SDOS/DOMAINS.md"
      tipo: spec
```

---

### 10. Nodo del Grafo

| Campo | Ejemplo |
|-------|---------|
| ID | `NODE-001` |
| Tipo | `graph_node` |
| Propietario | Discovery Engine |
| Dominio | Variable según el nodo |

```yaml
NODE-001:
  tipo: graph_node
  nombre: "ARCHITECTURE"
  version: "5"
  estado: SYNCED
  propietario: "Discovery Engine"
  dominio: technical
  descripcion: "Nodo de arquitectura en el Knowledge Graph"
  tags: ["graph", "architecture"]
  dependencias:
    - id: "NODE-000"
      tipo: requerido
      version: ">=1"
  documentacion:
    - path: "SDOS/GRAPH.md"
      tipo: spec
```

---

### 11. Relación

| Campo | Ejemplo |
|-------|---------|
| ID | `REL-001` |
| Tipo | `relationship` |
| Propietario | Discovery Engine |
| Dominio | `governance` |

```yaml
REL-001:
  tipo: relationship
  nombre: "ARCHITECTURE_afecta_CAPABILITIES"
  version: "1"
  estado: CURRENT
  propietario: "Discovery Engine"
  dominio: governance
  descripcion: "Relación: Architecture afecta a Capabilities"
  tags: ["relationship", "ontology"]
  dependencias:
    - id: "ONT-001"
      tipo: requerido
      version: ">=1.0.0"
  documentacion:
    - path: "SDOS/ONTOLOGY.md"
      tipo: spec
```

---

### 12. Ontología

| Campo | Ejemplo |
|-------|---------|
| ID | `ONT-001` |
| Tipo | `ontology` |
| Propietario | SRIE Core |
| Dominio | `governance` |

```yaml
ONT-001:
  tipo: ontology
  nombre: "SRIE Ontology"
  version: "1.0.0"
  estado: CURRENT
  propietario: "SRIE Core"
  dominio: governance
  descripcion: "Ontología del sistema SRIE: tipos de entidades, relaciones válidas y prohibidas"
  tags: ["ontology", "core"]
  dependencias: []
  documentacion:
    - path: "SDOS/ONTOLOGY.md"
      tipo: spec
```

---

## Esquema de IDs

Cada entidad tiene un ID único en el formato:

```
<TIPO>-<NUMERO>
```

| Prefijo | Tipo |
|---------|------|
| `PROJECT` | Proyecto |
| `ENG` | Engine |
| `CAP` | Capability |
| `DOC` | Documento |
| `MAN` | Manifest |
| `ART` | Artefacto |
| `EVT` | Evento |
| `IND` | Indicador |
| `DOM` | Dominio |
| `NODE` | Nodo del grafo |
| `REL` | Relación |
| `ONT` | Ontología |

Los números son secuenciales y únicos dentro de cada tipo. El Registry garantiza que no haya IDs duplicados.

---

## Ciclo de vida de una entidad en el Registry

```
CREATED ────── Entidad registrada por primera vez
    │
    ▼
ACTIVE ─────── Entidad en uso, versión actual
    │
    ├──► UPDATED ─── Versión modificada (mismo ID, nueva version)
    │
    ├──► DEPRECATED ── Entidad en desuso
    │
    └──► SUPERSEDED ─ Reemplazada por otra entidad
    │
    ▼
ARCHIVED ────── Entidad histórica, solo consulta
```

---

## Operaciones del Registry

### Registrar

Dar de alta una nueva entidad en el sistema.

```yaml
input:
  tipo: string
  nombre: string
  propietario: string
  dominio: string
  dependencias: []
output:
  id: string
  version: "1"
  estado: CREATED
reglas:
  - El ID se asigna automáticamente según el tipo
  - Si ya existe una entidad con el mismo nombre y tipo, falla con DUPLICATE_ENTITY
  - Si las dependencias no existen, falla con DEPENDENCY_NOT_FOUND
```

### Actualizar

Modificar una entidad existente (incrementa versión).

```yaml
input:
  id: string
  cambios: {}
output:
  version: integer (incrementado)
  estado: UPDATED
reglas:
  - No se puede cambiar el tipo de una entidad
  - Si se cambian las dependencias, se validan antes
```

### Consultar

Obtener información de una entidad.

```yaml
Consultas soportadas:
  - Por ID: REGISTRY.entidades["ENG-001"]
  - Por tipo: todas las entidades de tipo "engine"
  - Por dominio: todas las entidades en dominio "functional"
  - Por estado: todas las entidades en estado "DEPLOYED"
  - Por dependencia: todas las entidades que dependen de "CAP-001"
```

### Archivar

Mover una entidad a estado ARCHIVED.

```yaml
input:
  id: string
  motivo: string
output:
  estado: ARCHIVED
reglas:
  - No se puede archivar una entidad de la que otras dependen
  - Antes de archivar, se deben reasignar o eliminar las dependencias
```

---

## Registry como archivo vivo

El Registry se materializa en `SDOS/REGISTRY.yaml`. Este archivo es la fuente de truth operativa del sistema.

```yaml
# SDOS/REGISTRY.yaml
registry:
  version: "1.0.0"
  ultima_actualizacion: "2026-07-15T12:00:00Z"
  entidades:
    ENG-001:
      tipo: engine
      nombre: "Discovery Engine"
      version: "1.0.0"
      estado: REGISTERED
      propietario: "SRIE Core"
      dominio: governance
      ...
    CAP-001:
      tipo: capability
      nombre: "Calendar"
      version: "1.2.0"
      estado: DEPLOYED
      ...
```

Cualquier cambio en el sistema debe reflejarse en `SDOS/REGISTRY.yaml`. Los Engines leen el Registry para saber qué existe, y escriben en él para registrar nuevas entidades o actualizar existentes.
