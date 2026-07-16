# SRIE COMPILER — De Especificación a Intermediate Representation

---

## ¿Qué es este documento?

COMPILER.md define el **compilador de SRIE** — el proceso que transforma los 64 documentos de especificación (.md) en el Intermediate Representation (.ir.yaml) que el Runtime ejecuta.

No es un parser. Es un **compilador**: lee, valida, resuelve dependencias, optimiza y produce un IR normalizado.

---

## Arquitectura del Compiler

```
┌──────────────────────────────────────────────────────────────┐
│                      SRIE COMPILER                           │
│                                                              │
│  SPECIFICATION (64 .md files)                                │
│       │                                                       │
│       ▼                                                       │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────┐ │
│  │  READER  │─►│ PARSER   │─►│ VALIDATOR│─►│  RESOLVER    │ │
│  │(load all │  │(extract  │  │(check    │  │(dependencies, │ │
│  │ .md)     │  │ YAML +   │  │ ontology,│  │ cross-refs)  │ │
│  │          │  │ tables)  │  │ types)   │  │              │ │
│  └──────────┘  └──────────┘  └────┬─────┘  └──────┬───────┘ │
│                                    │                │         │
│                                    ▼                ▼         │
│                             ┌──────────────────────────────┐ │
│                             │       IR GENERATOR           │ │
│                             │  (build SRIE.ir.yaml)        │ │
│                             └──────────────┬───────────────┘ │
│                                             │                 │
│                                             ▼                 │
│                                      ┌──────────────┐        │
│                                      │  IR WRITER   │        │
│                                      │ (SDOS/ir/)   │        │
│                                      └──────────────┘        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
        │
        ▼
SDOS/ir/SRIE.ir.yaml
```

---

## Componentes

### 1. Reader

```yaml
reader:
  entrada: "64 documentos .md en SDOS/"
  proceso:
    - "Leer cada archivo .md"
    - "Extraer frontmatter YAML (si existe)"
    - "Extraer tablas, listas, code blocks"
    - "Clasificar por tipo de documento (spec | contract | schema | process)"
  salida: "Documentos parseados en memoria"
  si_error: "Reportar archivo no encontrado o ilegible"
```

### 2. Parser

```yaml
parser:
  entrada: "Documentos crudos del Reader"
  proceso:
    - "Parsear YAML frontmatter → dict"
    - "Parsear tablas → list[dict]"
    - "Parsear code blocks YAML/JSON → dict"
    - "Parsear listas → list"
    - "Extraer referencias a otros documentos"
  salida: "Estructuras de datos por documento"
  schemas:
    - "Cada tipo de documento tiene un schema de parseo definido"
    - "Si el documento no cumple el schema → error de parseo"
```

### 3. Validator

```yaml
validator:
  entrada: "Estructuras parseadas"
  validaciones:
    ontologia:
      - "Todas las entidades referenciadas existen en ONTOLOGY.md"
      - "Todas las relaciones son válidas"
      - "No hay relaciones prohibidas"
    registry:
      - "Todos los IDs son únicos"
      - "Todas las dependencias existen"
    graph:
      - "Los nodos del grafo son tipos ontológicos válidos"
      - "Las aristas son relaciones válidas"
    eventos:
      - "Todos los eventos referenciados están definidos en EVENTS.md"
    indicadores:
      - "Las fórmulas de indicadores referencian indicadores existentes"
      - "Los thresholds son coherentes"
    contratos:
      - "Los contratos referencian capacidades y eventos existentes"
    recursos:
      - "Los recursos referencian capacidades existentes"
    políticas:
      - "Las condiciones referencian variables y acciones existentes"
  salida: "Documentos validados (o lista de errores)"
  si_error: "Compilación falla. Se reportan todos los errores encontrados."
```

### 4. Resolver

```yaml
resolver:
  entrada: "Documentos validados"
  proceso:
    - "Resolver referencias cruzadas entre documentos"
    - "Ej: INDICATOR_CATALOG.md referencia INDICATOR_MODEL.md"
    - "Completar información implícita"
    - "Ej: Un Domain sin nivel explícito hereda el mínimo de DOMAINS.md"
    - "Calcular dependencias entre componentes"
    - "Detectar ciclos en dependencias"
  salida: "Documentos resueltos (sin referencias pendientes)"
```

### 5. IR Generator

```yaml
ir_generator:
  entrada: "Documentos resueltos"
  proceso:
    - "Construir cada sección del IR desde los documentos correspondientes"
    - "Normalizar formatos (unificar YAML de diferentes orígenes)"
    - "Aplicar reglas de optimización:"
      - "Datos duplicados → una sola fuente"
      - "Referencias → IDs directos"
      - "Herencia → atributos expandidos"
    - "Construir índice de componentes"
  salida: "IR completo en memoria"
  mapeo:
    SRIE_ENGINE.md: "foundation"
    ONTOLOGY.md: "ontology"
    IDENTITY.md: "identity"
    REGISTRY.md: "registry"
    GRAPH.md: "graph"
    DOMAINS.md: "domains"
    EVENTS.md: "events"
    STATES.md: "states"
    ARTIFACTS.md: "artifacts"
    INDICATOR_CATALOG.md: "indicators.catalog"
    DIAGNOSIS_PATTERNS.md: "diagnosis.patterns"
    CASE_REGISTRY.md: "diagnosis.cases"
    RESOURCES.md: "resources.types"
    RESOURCE_CONTRACT.md: "resources.contract"
    POLICY_ENGINE.md: "policies"
    RUNTIME.md: "runtime"
    LIFECYCLE.md: "lifecycle"
```

### 6. IR Writer

```yaml
ir_writer:
  entrada: "IR completo en memoria"
  proceso:
    - "Serializar a YAML"
    - "Escribir SDOS/ir/SRIE.ir.yaml"
    - "Escribir SDOS/ir/IR_MANIFEST.yaml (metadatos de la compilación)"
  salida: "SDOS/ir/SRIE.ir.yaml + SDOS/ir/IR_MANIFEST.yaml"
```

---

## Manifest de compilación

```yaml
# SDOS/ir/IR_MANIFEST.yaml
compilation:
  version: "1.0.0"
  fecha: "2026-07-15T10:00:00Z"
  compiler_version: "1.0.0"
  spec_version: "1.0.0"

  documentos:
    total: 64
    leidos: 64
    parseados: 64
    validados: 64
    resueltos: 64

  validaciones:
    total: 128
    pasadas: 128
    fallidas: 0

  errores: []
  advertencias:
    - "IND-041 (tech_debt_30d_probability): requiere 10 mediciones históricas para predictivo"

  output:
    ir_path: "SDOS/ir/SRIE.ir.yaml"
    ir_size_kb: 245
    secciones: 14
```

---

## Modos del Compiler

```yaml
compiler_modes:
  full:
    descripcion: "Compilar toda la especificación desde cero"
    uso: "Después de cambios en la Foundation"
    duracion_estimada_ms: 5000

  incremental:
    descripcion: "Compilar solo documentos modificados desde última compilación"
    uso: "Uso diario, cambios en un documento"
    duracion_estimada_ms: 500
    detector_cambios: "Hash del contenido del archivo"

  validate:
    descripcion: "Solo validar, no generar IR"
    uso: "Pre-commit hook, CI"
    salida: "Lista de errores de validación"
```

---

## Commands

```yaml
commands:
  srie compile:
    descripcion: "Compilar especificación completa"
    flags:
      --mode: "full | incremental | validate"
      --watch: "Re-compilar ante cambios en archivos .md"
      --output: "Ruta del IR (default: SDOS/ir/)"
    ejemplos:
      - "srie compile"
      - "srie compile --mode validate"
      - "srie compile --watch"
```

---

## Integración con el Runtime

```yaml
runtime_integration:
  arranque:
    1. "Runtime verifica que SDOS/ir/SRIE.ir.yaml existe"
    2. "Si no existe → ejecutar srie compile"
    3. "Si existe pero desactualizado → ejecutar srie compile --mode incremental"
    4. "Runtime carga IR en memoria"
    5. "Runtime inicia operación"

  recarga:
    trigger: "Archivo .md modificado + watcher activo"
    accion: "Re-compilar incremental, recargar IR en caliente"
    riesgo: "Si la recarga falla, Runtime sigue con IR anterior"
```
