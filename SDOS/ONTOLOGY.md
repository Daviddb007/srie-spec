# ONTOLOGY — Ontología del Sistema SRIE

---

## ¿Qué es este documento?

ONTOLOGY.md define la **ontología del sistema SRIE**: qué entidades existen, qué relaciones son válidas, qué relaciones están prohibidas, qué cardinalidades aplican, y cómo se heredan propiedades entre entidades.

Mientras que GRAPH.md describe el grafo *de un proyecto concreto*, ONTOLOGY.md describe **el esquema que todo grafo debe seguir**. Es el metamodelo del metamodelo — el lenguaje con el que todos los Engines piensan.

---

## ¿Qué es una ontología?

Una ontología es una especificación explícita de una conceptualización. En SRIE, la ontología define:

- **Tipos de entidades** que pueden existir en el sistema
- **Atributos** que cada tipo de entidad tiene
- **Relaciones válidas** entre tipos de entidades
- **Relaciones prohibidas** que no tienen sentido semántico
- **Cardinalidades** de las relaciones (1:1, 1:N, N:M)
- **Herencia** entre tipos de entidades
- **Restricciones** que el sistema debe validar

---

## Tipos de entidades (clases)

### Jerarquía de tipos

```
Entity (abstracta)
├── Project
├── Engine
├── Capability
├── Artifact (abstracta)
│   ├── Document
│   ├── Evidence
│   ├── Log
│   ├── Memory
│   ├── Release
│   ├── Test
│   ├── Patch
│   ├── Adr
│   ├── Indicator
│   └── Manifest
├── Domain
├── GraphNode
├── Relationship
├── Event
└── Ontology
```

### Entity (abstracta — no se instancia)

```yaml
Entity:
  abstract: true
  atributos:
    - nombre: id
      tipo: string
      requerido: true
      descripcion: "Identificador único en el Registry"
    - nombre: nombre
      tipo: string
      requerido: true
      descripcion: "Nombre legible"
    - nombre: version
      tipo: string
      requerido: true
      descripcion: "Versión semver"
    - nombre: estado
      tipo: string
      requerido: true
      descripcion: "Estado actual (ver STATES.md)"
      valores_posibles:
        - CREATED
        - ACTIVE
        - UPDATED
        - DEPRECATED
        - SUPERSEDED
        - ARCHIVED
    - nombre: propietario
      tipo: string
      requerido: false
      descripcion: "Engine o equipo responsable"
    - nombre: dominio
      tipo: string
      requerido: true
      descripcion: "Dominio al que pertenece (ver DOMAINS.md)"
    - nombre: descripcion
      tipo: string
      requerido: false
      descripcion: "Breve descripción"
    - nombre: tags
      tipo: string[]
      requerido: false
      descripcion: "Etiquetas de clasificación"
    - nombre: creado
      tipo: datetime
      requerido: true
      descripcion: "Timestamp de creación"
    - nombre: actualizado
      tipo: datetime
      requerido: true
      descripcion: "Timestamp de última modificación"
```

### Project

```yaml
Project:
  hereda: Entity
  atributos_adicionales:
    - nombre: nivel_madurez
      tipo: string
      requerido: true
      valores_posibles: [L0, L1, L2, L3, L4, L5]
    - nombre: srie_score
      tipo: float
      requerido: false
    - nombre: lenguajes
      tipo: string[]
      requerido: false
    - nombre: frameworks
      tipo: string[]
      requerido: false
  restricciones:
    - "Un proyecto no puede depender de otro proyecto"
    - "Un proyecto es la raíz del grafo"
```

### Engine

```yaml
Engine:
  hereda: Entity
  atributos_adicionales:
    - nombre: tipo_engine
      tipo: string
      requerido: true
      valores_posibles:
        - discovery
        - indicators
        - dx
        - planner
        - capability
        - sandbox
        - repair
        - memory
        - knowledge
        - governance
        - deployment
        - evolution
    - nombre: ciclo_vida
      tipo: string
      requerido: true
      valores_posibles: [REGISTERED, IDLE, RUNNING, SUCCESS, FAILED, ERROR]
    - nombre: tiempo_esperado_ms
      tipo: integer
      requerido: false
    - nombre: nivel_srie_requerido
      tipo: string
      requerido: true
      valores_posibles: [L0, L1, L2, L3, L4, L5]
  restricciones:
    - "Un Engine no puede depender de sí mismo"
    - "El ciclo de vida debe seguir: REGISTERED → IDLE → RUNNING → SUCCESS/FAILED/ERROR → IDLE"
```

### Capability

```yaml
Capability:
  hereda: Entity
  atributos_adicionales:
    - nombre: tipo_capability
      tipo: string
      requerido: false
      valores_posibles: [new, modification, repair]
    - nombre: ciclo_vida
      tipo: string
      requerido: true
      valores_posibles:
        - DRAFT
        - BUILDING
        - TESTING
        - REJECTED
        - STABLE
        - DEPLOYED
        - UPDATING
        - DEPRECATED
        - ARCHIVED
        - BROKEN
        - REPAIR
    - nombre: manifest_path
      tipo: string
      requerido: true
      descripcion: "Ruta al archivo .manifest"
  restricciones:
    - "Una Capability debe tener al menos un documento asociado"
    - "No pueden existir dependencias circulares entre capabilities"
    - "Si A depende de B, B debe estar en estado STABLE o DEPLOYED"
```

### Artifact (abstracta)

```yaml
Artifact:
  abstract: true
  hereda: Entity
  atributos_adicionales:
    - nombre: path
      tipo: string
      requerido: true
      descripcion: "Ruta del archivo en el sistema de archivos"
    - nombre: engine_origen
      tipo: string
      requerido: true
      descripcion: "Engine que generó el artefacto"
    - nombre: ejecucion_id
      tipo: string
      requerido: true
      descripcion: "ID de la ejecución que lo generó"
    - nombre: tipo_artefacto
      tipo: string
      requerido: true
      valores_posibles:
        - document
        - evidence
        - log
        - memory
        - release
        - test
        - patch
        - adr
        - indicator
        - manifest
  restricciones:
    - "Un artifact es inmutable (no se modifica después de creado)"
    - "Cada artifact debe referenciar su Engine de origen"
```

### Document (hereda de Artifact)

```yaml
Document:
  hereda: Artifact
  atributos_adicionales:
    - nombre: tipo_contenido
      tipo: string
      requerido: true
      valores_posibles:
        - especificacion
        - diseño
        - guia
        - referencia
        - reporte
    - nombre: formato
      tipo: string
      requerido: true
      valores_posibles: [markdown, yaml, json]
```

### Domain

```yaml
Domain:
  hereda: Entity
  atributos_adicionales:
    - nombre: color
      tipo: string
      requerido: false
      descripcion: "Color para representación visual"
    - nombre: nivel_minimo
      tipo: string
      requerido: true
      descripcion: "Nivel SRIE mínimo para que este dominio sea relevante"
      valores_posibles: [L0, L1, L2, L3, L4, L5]
    - nombre: patrones_archivo
      tipo: string[]
      requerido: false
      descripcion: "Glob patterns de archivos típicos de este dominio"
  restricciones:
    - "Cada archivo pertenece a un único dominio"
    - "Un dominio no puede depender de sí mismo"
```

### GraphNode

```yaml
GraphNode:
  hereda: Entity
  atributos_adicionales:
    - nombre: tipo_nodo
      tipo: string
      requerido: true
      valores_posibles: [raiz, dominio, capability, engine, artifact, state]
    - nombre: archivos_asociados
      tipo: string[]
      requerido: false
    - nombre: estado_nodo
      tipo: string
      requerido: true
      valores_posibles: [PENDING, SYNCED, DESYNCED, STALE, ORPHAN]
    - nombre: version_nodo
      tipo: integer
      requerido: true
  restricciones:
    - "Un nodo raíz no puede depender de ningún otro nodo"
    - "Un nodo huérfano (ORPHAN) debe ser revisado o eliminado"
```

### Relationship

```yaml
Relationship:
  hereda: Entity
  atributos_adicionales:
    - nombre: origen
      tipo: string
      requerido: true
      descripcion: "ID del nodo origen"
    - nombre: destino
      tipo: string
      requerido: true
      descripcion: "ID del nodo destino"
    - nombre: tipo_relacion
      tipo: string
      requerido: true
      valores_posibles:
        - contiene
        - depende_de
        - implementa
        - alimenta
        - valida
        - despliega
        - documenta
        - afecta
    - nombre: peso
      tipo: integer
      requerido: false
      minimo: 1
      maximo: 10
    - nombre: label
      tipo: string
      requerido: false
  restricciones:
    - "Origen y destino deben ser entidades existentes"
    - "No pueden existir relaciones duplicadas (mismo origen, destino y tipo)"
    - "Una relación no puede tener como origen y destino la misma entidad"
```

### Event

```yaml
Event:
  hereda: Entity
  atributos_adicionales:
    - nombre: tipo_evento
      tipo: string
      requerido: true
    - nombre: origen
      tipo: object
      requerido: true
      estructura:
        engine: string
        ejecucion_id: string
    - nombre: entidad
      tipo: object
      requerido: true
      estructura:
        tipo: string
        id: string
        version: integer
    - nombre: severidad
      tipo: string
      requerido: true
      valores_posibles: [info, warning, error, critical]
    - nombre: datos
      tipo: object
      requerido: false
  restricciones:
    - "Un evento es inmutable después de creado"
    - "La entidad referenciada debe existir en el Registry"
```

---

## Relaciones válidas entre tipos

### Matriz de relaciones permitidas

| Tipo origen | Relación | Tipo destino | Cardinalidad | ¿Obligatoria? |
|-------------|----------|--------------|--------------|----------------|
| Project | `contiene` | Domain | 1:N | Sí (al menos 1) |
| Project | `contiene` | GraphNode | 1:N | Sí |
| Project | `contiene` | Engine | 1:N | No |
| Project | `contiene` | Capability | 1:N | No |
| Domain | `contiene` | GraphNode | 1:N | No |
| GraphNode | `depende_de` | GraphNode | N:M | No |
| GraphNode | `afecta` | GraphNode | N:M | No |
| GraphNode | `alimenta` | GraphNode | N:M | No |
| Capability | `depende_de` | Capability | N:M | No |
| Capability | `alimenta` | GraphNode | 1:N | Sí |
| Capability | `alimenta` | Artifact | 1:N | Sí |
| Engine | `implementa` | Domain | 1:N | Sí |
| Engine | `genera` | Artifact | 1:N | Sí |
| Engine | `valida` | Capability | 1:N | No |
| Engine | `despliega` | Capability | 1:N | No |
| Artifact | `documenta` | Capability | N:1 | No |
| Artifact | `documenta` | GraphNode | N:1 | No |
| Event | `referencia` | Entity | 1:1 | Sí |
| Relationship | `conecta` | GraphNode | 2:1 | Sí |

### Relaciones prohibidas

| Tipo origen | Relación | Tipo destino | ¿Por qué? |
|-------------|----------|--------------|-----------|
| Project | `depende_de` | Project | Un proyecto no puede depender de otro proyecto |
| Engine | `depende_de` | Capability | Los engines no dependen de capabilities |
| Capability | `implementa` | Engine | Las capabilities no implementan engines |
| Event | `depende_de` | Event | Los eventos no tienen dependencias |
| Domain | `depende_de` | Capability | Los dominios no dependen de capabilities |
| Artifact | `despliega` | Artifact | Los artefactos no se despliegan |
| Relationship | `contiene` | Relationship | Las relaciones no contienen relaciones |
| Ontology | `depende_de` | Cualquiera | La ontología es autorreferencial |

### Cardinalidades

| Cardinalidad | Significado | Ejemplo |
|--------------|-------------|---------|
| 1:1 | Una instancia de A se relaciona con exactamente una instancia de B | Event → Entity referenciada |
| 1:N | Una instancia de A se relaciona con múltiples instancias de B | Project → Domains |
| N:M | Múltiples instancias de A se relacionan con múltiples instancias de B | Capability → Capability (dependencias) |
| 2:1 | Dos instancias de A (origen y destino) se relacionan con una instancia de B | Relationship → GraphNode |

---

## Herencia

### Reglas de herencia

1. **Todas las entidades heredan de Entity.** Los atributos de Entity están presentes en toda entidad del sistema.
2. **Herencia simple.** Una entidad hereda de exactamente una clase padre.
3. **Sobreescritura.** Una entidad puede especificar valores por defecto diferentes a los del padre para atributos heredados.
4. **Restricción acumulativa.** Las restricciones del padre se suman a las del hijo. No se eliminan.

### Ejemplo de cadena de herencia

```
Entity (id, nombre, version, estado, propietario, dominio, descripcion, tags, creado, actualizado)
  └── Artifact (path, engine_origen, ejecucion_id, tipo_artefacto)
        └── Document (tipo_contenido, formato)
        └── Evidence (ningún atributo adicional)
        └── Adr (contexto, decision, alternativas, justificacion)
```

---

## Restricciones globales

### Restricción 1: Existencia

Ninguna entidad puede ser referenciada antes de existir en el Registry. Esto aplica a relaciones, dependencias y eventos.

### Restricción 2: Aciclicidad

No pueden existir ciclos en las dependencias. Si A depende de B y B depende de C, C no puede depender de A.

### Restricción 3: Unicidad de ID

Cada entidad tiene un ID único en todo el sistema. No puede haber dos entidades con el mismo ID.

### Restricción 4: Consistencia de estado

Una entidad no puede cambiar a un estado que no esté en su máquina de estados definida (ver STATES.md).

### Restricción 5: Herencia de dominio

Si una entidad no especifica un dominio explícito, hereda el dominio de su entidad contenedora.

### Restricción 6: Versionado semver

Toda entidad debe usar versionado semver (MAJOR.MINOR.PATCH) para cambios en su contrato.

### Restricción 7: Trazabilidad de eventos

Toda transición de estado debe ir acompañada de un evento. No se permiten transiciones silenciosas.

---

## Validación ontológica

Cada vez que se modifica el grafo, el sistema debe ejecutar una validación ontológica:

```yaml
validacion_ontologica:
  pasos:
    - nombre: validar_existencia
      descripcion: "Todas las entidades referenciadas existen en el Registry"

    - nombre: validar_relaciones
      descripcion: "Todas las relaciones son válidas según la matriz"

    - nombre: validar_cardinalidades
      descripcion: "Ninguna cardinalidad es excedida"

    - nombre: validar_prohibiciones
      descripcion: "No existen relaciones prohibidas"

    - nombre: validar_ciclos
      descripcion: "No existen dependencias circulares"

    - nombre: validar_estados
      descripcion: "Todas las entidades están en estados válidos"

    - nombre: validar_herencia
      descripcion: "Todas las herencias son válidas"

  resultado:
    - exito: "ONTOLOGY_VALID — Todas las validaciones pasaron"
    - error: "ONTOLOGY_INVALID — Lista de violaciones"
```

---

## Ontología como archivo vivo

La ontología se materializa en `SDOS/ONTOLOGY.yaml`:

```yaml
# SDOS/ONTOLOGY.yaml
ontology:
  version: "1.0.0"
  ultima_actualizacion: "2026-07-15T12:00:00Z"
  entidades:
    Engine:
      hereda: Entity
      atributos_adicionales:
        - nombre: tipo_engine
          tipo: string
          ...
  relaciones:
    permitidas:
      - origen: Project
        tipo: contiene
        destino: Domain
        cardinalidad: "1:N"
    prohibidas:
      - origen: Project
        tipo: depende_de
        destino: Project
  restricciones:
    - "No pueden existir ciclos en las dependencias"
```
