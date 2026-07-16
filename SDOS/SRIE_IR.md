# SRIE IR — Intermediate Representation del Sistema

---

## ¿Qué es este documento?

SRIE_IR.md define el **formato intermedio (IR)** del sistema SRIE. El IR es una representación normalizada, compilada y optimizada de toda la especificación. El Runtime nunca lee los 64 documentos Markdown directamente. Lee el IR compilado desde `SDOS/ir/`.

Como LLVM para SRIE: los `.md` son la fuente, el Compiler produce el `.ir`, y el Runtime ejecuta sobre el IR.

---

## ¿Por qué un IR?

| Problema | Solución |
|----------|----------|
| 64 archivos Markdown para parsear | Un archivo IR normalizado |
| Formatos inconsistentes (YAML, JSON, MD) | Formato único (YAML) |
| Validación dispersa | Validación centralizada en el Compiler |
| Dependencias implícitas | Dependencias explícitas en el IR |
| Sin schema de tipos | Schema de tipos estricto en el IR |

---

## Estructura del IR

El IR se compila a `SDOS/ir/SRIE.ir.yaml`:

```yaml
# SDOS/ir/SRIE.ir.yaml
srie_ir:
  version: "1.0.0"
  spec_version: "1.0.0"
  compilado: "2026-07-15T10:00:00Z"
  compiler_version: "1.0.0"
  documentos_fuente: 64
  validaciones_pasadas: true

  foundation:
    version: "1.0.0"
    principios: [...]
    definiciones: {...}

  ontology:
    version: 3
    entity_types: {...}
    valid_relations: [...]
    prohibited_relations: [...]
    constraints: [...]

  identity: {...}
  authority: {...}
  governance: {...}
  registry: {...}
  graph: {...}
  domains: {...}
  states: {...}
  events: {...}
  artifacts: {...}
  contracts: {...}
  ecosystem: {...}
  timeline: {...}
  runtime: {...}
  lifecycle: {...}
  observability: {...}

  indicators:
    catalog: [...]
    scores: [...]
    events: [...]

  diagnosis:
    patterns: [...]
    cases: [...]

  resources:
    types: [...]
    contract: {...}

  policies: [...]
```

---

## Secciones del IR

### 1. Foundation

```yaml
foundation:
  version: "1.0.0"
  principios:
    - id: 1
      nombre: "Confianza trazable"
      enunciado: "Ningún Engine puede afirmar algo que no pueda demostrar..."
    - id: 2
      nombre: "Evidencia sobre memoria"
      enunciado: "Ninguna decisión de ingeniería se basa en lo que recuerda la conversación..."
  conceptos:
    - nombre: "Evidence"
      definicion: "Cualquier archivo generado por un Engine..."
    - nombre: "Confidence"
      definicion: "Medida de certeza (0.0-1.0) que acompaña a toda afirmación..."
```

### 2. Ontology

```yaml
ontology:
  version: 3
  entity_types:
    Project:
      inherits: Entity
      attributes:
        - name: nivel_madurez
          type: string
          enum: [L0, L1, L2, L3, L4, L5]
        - name: srie_score
          type: float
    Engine:
      inherits: Entity
      attributes:
        - name: tipo_engine
          type: string
          enum: [discovery, indicators, dx, planner, capability, sandbox, repair, memory, knowledge, governance, deployment, evolution]
  valid_relations:
    - source: Project
      type: contiene
      target: Domain
      cardinality: "1:N"
  prohibited_relations:
    - source: Project
      type: depende_de
      target: Project
  constraints:
    - "No pueden existir ciclos en las dependencias"
```

### 3. Identity

```yaml
identity:
  domain_id: "DOM-001"
  nombre: "SRIE Domain"
  tipo: "standalone"
  authority:
    nivel: "Domain"
    permissions:
      crear_domains: false
      crear_capabilities: true
      modificar_registry: "local"
  chain:
    - level: "CORE"
      id: "SRIE-CORE-001"
    - level: "DOMAIN"
      id: "DOM-001"
```

### 4. Registry

```yaml
registry:
  entities:
    ENG-001:
      type: engine
      name: "Discovery Engine"
      version: "1.0.0"
      state: "REGISTERED"
      dependencies: []
    CAP-001:
      type: capability
      name: "Calendar"
      version: "1.2.0"
      state: "DEPLOYED"
      dependencies: ["CAP-002"]
```

### 5. Graph

```yaml
graph:
  nodes:
    - id: "PROJECT"
      type: root
      state: "SYNCED"
    - id: "business"
      type: domain
      state: "SYNCED"
  edges:
    - source: "PROJECT"
      target: "business"
      type: "contiene"
      weight: 10
```

### 6. Domains

```yaml
domains:
  - id: "business"
    descripcion: "Dominio de negocio: propósito, objetivos, stakeholders"
    nivel_minimo: "L0"
    archivos_tipicos: ["README.md", "ROADMAP.md"]
    indicadores: ["readme_exists", "roadmap_exists"]
```

### 7. States

```yaml
states:
  project:
    - "UNKNOWN"
    - "DISCOVERED"
    - "ANALYZED"
    - "BOOTSTRAPPED"
    - "ACTIVE"
    - "STABLE"
    - "ARCHIVED"
  transitions:
    - from: "UNKNOWN"
      to: "DISCOVERED"
      trigger: "srie inspect"
  capability:
    - "DRAFT"
    - "BUILDING"
    - "TESTING"
    - "STABLE"
    - "DEPLOYED"
    - "DEPRECATED"
```

### 8. Events

```yaml
events:
  - tipo: "PROJECT_DISCOVERED"
    emitter: "INIT"
    severity: "info"
    description: "El proyecto fue descubierto por primera vez"
  - tipo: "ENGINE_STARTED"
    emitter: "Cualquier Engine"
    severity: "info"
  - tipo: "DOCUMENTATION_WARNING"
    emitter: "Indicators Engine"
    severity: "warning"
    indicador: "documentation_health"
```

### 9. Artifacts

```yaml
artifacts:
  - tipo: "document"
    formato: "markdown"
    extension: ".md"
    lifecycle: "versionado"
  - tipo: "evidence"
    formato: "json"
    extension: ".json"
    lifecycle: "inmutable"
  - tipo: "manifest"
    formato: "yaml"
    extension: ".yaml"
    lifecycle: "versionado"
```

### 10. Indicators

```yaml
indicators:
  catalog:
    - id: "IND-001"
      name: "readme_exists"
      level: "observacion"
      domain: "gobernanza"
      formula: "existe('README.md')"
      weight: 0.02
      threshold:
        warning: null
        critical: null
    - id: "IND-020"
      name: "documentation_health"
      level: "salud"
      domain: "gobernanza"
      formula: "(readme_exists × 0.30) + (architecture_exists × 0.25) + (changelog_exists × 0.15) + (memory_exists × 0.30)"
      range: [0, 100]
      threshold:
        warning: 70
        critical: 40
  scores:
    engineering_score:
      formula: "(architecture_health × 0.25) + (deployment_health × 0.20) + (security_health × 0.20) + (maintenance_risk_inv × 0.20) + (hallucination_risk_inv × 0.15)"
```

### 11. Diagnosis

```yaml
diagnosis:
  patterns:
    - id: "PAT-DOC-001"
      name: "DOCUMENTATION_DECAY"
      domain: "gobernanza"
      severity: "media"
      symptoms:
        - indicator: "documentation_health"
          condition: "valor < 70 AND tendencia = DECLINING"
          weight: 0.35
      intervention:
        actions: ["Crear ARCHITECTURE.md", "Actualizar memory/index.md"]
        impacto_esperado:
          documentation_health: "+15 puntos en 30 días"
  cases:
    - id: "CASE-003"
      titulo: "Deterioro de documentación por presión de features"
      domain: "gobernanza"
      symptoms: {...}
      diagnosis: "DOCUMENTATION_DECAY"
      intervention: {...}
      resultado:
        mejoria: "+17 puntos"
        satisfaccion: 0.90
```

### 12. Resources

```yaml
resources:
  types:
    - "tool"
    - "api"
    - "service"
    - "external"
    - "federated"
  contract:
    methods:
      - "execute(action, params) → Result"
      - "health() → HealthStatus"
      - "capabilities() → [string]"
      - "schema() → Schema"
```

### 13. Policies

```yaml
policies:
  - id: "POL-001"
    name: "No desplegar viernes"
    type: "pre"
    severity: "error"
    condition: "day_of_week = 'Friday' AND action = 'deploy'"
```

### 14. Runtime

```yaml
runtime:
  state: "STOPPED"
  kernel:
    event_loop: true
    scheduler: true
  engines_cargados: []
  resources_disponibles: 0
  observabilidad:
    logs: true
    metrics: true
    traces: true
```

---

## Compilación al IR

El proceso de compilación:
1. Leer cada documento `.md` de la especificación
2. Validar contra el schema del IR
3. Extraer datos estructurados (YAML frontmatter, tablas, listas)
4. Resolver referencias entre documentos
5. Verificar consistencia (ontología → graph → registry)
6. Escribir `SDOS/ir/SRIE.ir.yaml`

El Compiler se define en COMPILER.md.

---

## Validación del IR

```yaml
ir_validation:
  reglas:
    - "Todos los entity_types referenciados existen en ontology"
    - "Todas las relaciones son válidas según ontology.valid_relations"
    - "No hay relaciones prohibidas (ontology.prohibited_relations)"
    - "Todos los eventos referenciados existen en events"
    - "Todos los indicadores referenciados existen en indicators.catalog"
    - "Todos los recursos referenciados existen en resources"
    - "Las versiones son compatibles según ecosystem"

  estado: "VÁLIDO | INVÁLIDO | PARCIAL"
  si_invalido: "El Runtime no arranca. El Compiler reporta los errores."
```
