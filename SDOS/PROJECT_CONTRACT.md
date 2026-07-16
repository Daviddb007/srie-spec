# PROJECT CONTRACT — Contrato del Proyecto SRIE

---

## ¿Qué es este documento?

PROJECT_CONTRACT.md define **qué significa ser un proyecto gobernado por SRIE**. Describe la estructura obligatoria, los archivos de estado, el Knowledge Graph, los niveles de INIT y las reglas que todo proyecto debe cumplir para estar bajo el paraguas de SRIE.

---

## ¿Qué es un proyecto SRIE?

Un proyecto SRIE no es solo un repositorio con código. Es un **sistema gobernado** donde:

1. Existe una estructura de directorios predecible (`SDOS/`, `capabilities/`, `memory/`, `knowledge/`)
2. El estado del proyecto está siempre disponible en `SDOS/STATE.md`
3. Las decisiones de ingeniería se registran en ADR dentro de `memory/`
4. Las capacidades se organizan como microproyectos autocontenidos en `capabilities/`
5. Los patrones y antipatrones se consolidan en `knowledge/`
6. El ciclo SDOS se ejecuta de forma repetible y medible

---

## Estructura obligatoria del proyecto

```
<proyecto>/
├── SDOS/                           # Sistema Operativo de Desarrollo
│   ├── STATE.md                    # Estado del ciclo SDOS (OBLIGATORIO)
│   ├── PROJECT_STATE.md            # Inventario del proyecto (OBLIGATORIO)
│   ├── PROJECT.yaml                # Manifest del proyecto (OBLIGATORIO)
│   ├── GRAPH.md                    # Knowledge Graph (OBLIGATORIO)
│   ├── SRIE_REPORT.md              # Reporte de indicadores
│   ├── DX_REPORT.md                # Diagnóstico de experiencia de desarrollo
│   ├── PLAN.md                     # Plan de evolución
│   ├── VERIFICATION_REPORT.md      # Resultados de verificación
│   ├── DEPLOY_REPORT.md            # Reportes de despliegue
│   ├── GOVERNANCE_REPORT.md        # Auditoría de gobierno
│   ├── REPAIR_REPORT.md            # Reportes de reparación
│   ├── evidence/                   # Evidencia de ejecución de Engines
│   │   ├── discovery/
│   │   ├── indicators/
│   │   ├── dx/
│   │   ├── planner/
│   │   ├── sandbox/
│   │   └── deployment/
│   └── SRIE_ENGINE.md              # Copia de la constitución
│
├── capabilities/                   # Capacidades del proyecto
│   └── <nombre>/                   # Una carpeta por capability
│       ├── <nombre>.manifest
│       ├── <nombre>.md
│       ├── <nombre>_design.md
│       ├── <nombre>_api.md
│       ├── <nombre>_tests.md
│       ├── <nombre>_memory.md
│       ├── <nombre>_metrics.md
│       ├── <nombre>_release.md
│       ├── src/
│       └── tests/
│
├── engines/                        # Implementación de Engines (FASE 5+)
│   ├── discovery/
│   │   ├── engine.yaml
│   │   └── ...
│   ├── indicators/
│   │   ├── engine.yaml
│   │   └── ...
│   └── .../
│
├── memory/                         # Memoria del proyecto
│   ├── index.md                    # Índice de memoria (OBLIGATORIO)
│   ├── ADR/                        # Architectural Decision Records
│   │   ├── 2026-07-15-decision-1.md
│   │   └── ...
│   ├── context/                    # Contexto por iteración
│   │   ├── iteracion-001.md
│   │   └── ...
│   └── evolution.md                # Evolución del proyecto
│
├── knowledge/                      # Conocimiento acumulado
│   ├── index.md                    # Índice de knowledge (OBLIGATORIO)
│   ├── patterns/                   # Patrones exitosos
│   ├── anti_patterns/              # Antipatrones identificados
│   ├── performance_patterns/       # Patrones de rendimiento
│   └── security_patterns/          # Patrones de seguridad
│
└── ...                             # Código del proyecto
```

---

## Project Manifest (`SDOS/PROJECT.yaml`)

```yaml
# SDOS/PROJECT.yaml
project:
  nombre: "mi-proyecto"
  version: "1.0.0"
  descripcion: "Descripción del proyecto"

  srie_version: "1.0.0"

  nivel_actual: "L2"

  lenguajes:
    - "Python 3.12"
    - "JavaScript ES2024"

  frameworks:
    - "Flask 3.1"
    - "React 18"

  engines_implementados:
    - "discovery"
    - "indicators"

  capabilities_count: 5

  dependencias_principales:
    - "flask==3.1.0"
    - "sqlalchemy==2.0.0"

  init:
    nivel: 3                        # 1 = inspect, 2 = analyze, 3 = bootstrap
    fecha: "2026-07-15"
    ejecutado_por: "srie bootstrap"

  contactos:
    - rol: "mantenedor"
      nombre: "Equipo SRIE"
      email: "equipo@srie.dev"
```

---

## Knowledge Graph (`SDOS/GRAPH.md`)

El Knowledge Graph es un mapa de todos los nodos del proyecto y sus relaciones. No es un archivo markdown tradicional — es una estructura de datos que INIT construye y los Engines actualizan.

### Formato

```yaml
# SDOS/GRAPH.md
version: 1.0
ultima_actualizacion: "2026-07-15T10:00:00Z"

nodos:
  - id: "PROJECT"
    tipo: "raiz"
    descripcion: "Nodo raíz del proyecto"
    archivos:
      - "SDOS/PROJECT.yaml"
      - "SDOS/PROJECT_STATE.md"
    depende_de: []
    dependientes:
      - "BUSINESS"
      - "ARCHITECTURE"
      - "CAPABILITIES"
      - "MEMORY"
      - "INDICATORS"
      - "DEPLOYMENT"
      - "DESIGN"
      - "SECURITY"
      - "SDOS"
    ultima_modificacion: "2026-07-15T10:00:00Z"

  - id: "BUSINESS"
    tipo: "dominio"
    descripcion: "Dominio de negocio del proyecto"
    archivos:
      - "README.md"
    depende_de:
      - "PROJECT"
    dependientes:
      - "ARCHITECTURE"
    ultima_modificacion: "2026-07-14T00:00:00Z"

  - id: "ARCHITECTURE"
    tipo: "dominio"
    descripcion: "Arquitectura del proyecto"
    archivos:
      - "ARCHITECTURE.md"
      - "memory/ADR/"
    depende_de:
      - "PROJECT"
      - "BUSINESS"
    dependientes:
      - "CAPABILITIES"
      - "DESIGN"
      - "SECURITY"
    ultima_modificacion: "2026-07-15T08:00:00Z"

  - id: "CAPABILITIES"
    tipo: "grupo"
    descripcion: "Capacidades del proyecto"
    archivos:
      - "capabilities/"
    depende_de:
      - "PROJECT"
      - "ARCHITECTURE"
    dependientes:
      - "MEMORY"
      - "INDICATORS"
    hijos:
      - "CAPABILITY:calendar"
      - "CAPABILITY:auth"
    ultima_modificacion: "2026-07-15T09:00:00Z"

  - id: "CAPABILITY:calendar"
    tipo: "capability"
    descripcion: "Capacidad de calendario"
    archivos:
      - "capabilities/calendar/"
    depende_de:
      - "CAPABILITIES"
      - "CAPABILITY:auth"
    dependientes:
      - "MEMORY"
    ultima_modificacion: "2026-07-15T09:30:00Z"

  - id: "CAPABILITY:auth"
    tipo: "capability"
    descripcion: "Capacidad de autenticación"
    archivos:
      - "capabilities/auth/"
    depende_de:
      - "CAPABILITIES"
    dependientes:
      - "CAPABILITY:calendar"
    ultima_modificacion: "2026-07-14T00:00:00Z"

  - id: "MEMORY"
    tipo: "dominio"
    descripcion: "Memoria del proyecto"
    archivos:
      - "memory/"
    depende_de:
      - "PROJECT"
      - "CAPABILITIES"
    dependientes:
      - "INDICATORS"
    ultima_modificacion: "2026-07-15T10:00:00Z"

  - id: "INDICATORS"
    tipo: "dominio"
    descripcion: "Indicadores de madurez"
    archivos:
      - "SDOS/SRIE_REPORT.md"
    depende_de:
      - "PROJECT"
      - "MEMORY"
      - "CAPABILITIES"
    dependientes: []
    ultima_modificacion: "2026-07-15T10:00:00Z"

  - id: "DEPLOYMENT"
    tipo: "dominio"
    descripcion: "Despliegue y operación"
    archivos:
      - "Dockerfile"
      - "docker-compose.yml"
      - ".github/"
    depende_de:
      - "PROJECT"
    dependientes: []
    ultima_modificacion: "2026-07-13T00:00:00Z"

  - id: "DESIGN"
    tipo: "dominio"
    descripcion: "Diseño del sistema"
    archivos:
      - "DESIGN.md"
      - "capabilities/*/*_design.md"
    depende_de:
      - "PROJECT"
      - "ARCHITECTURE"
    dependientes: []
    ultima_modificacion: "2026-07-14T00:00:00Z"

  - id: "SECURITY"
    tipo: "dominio"
    descripcion: "Seguridad del proyecto"
    archivos:
      - ".env.example"
      - "SECURITY.md"
    depende_de:
      - "PROJECT"
      - "ARCHITECTURE"
    dependientes: []
    ultima_modificacion: "2026-07-12T00:00:00Z"

  - id: "SDOS"
    tipo: "sistema"
    descripcion: "Estado del sistema operativo"
    archivos:
      - "SDOS/STATE.md"
    depende_de:
      - "PROJECT"
    dependientes: []
    ultima_modificacion: "2026-07-15T10:00:00Z"

eventos:
  - origen: "ARCHITECTURE"
    tipo: "modificacion"
    destino: "CAPABILITIES"
    mensaje: "ARCHITECTURE modificado — revisar contratos de capabilities"
    fecha: "2026-07-15T08:00:00Z"

  - origen: "CAPABILITY:calendar"
    tipo: "actualizacion"
    destino: "MEMORY"
    mensaje: "Calendar actualizado a v1.2.0 — registrar decisiones"
    fecha: "2026-07-15T09:30:00Z"
```

### Propagación de cambios

Cuando un nodo se modifica, el sistema debe:

1. Identificar todos los nodos que dependen del nodo modificado
2. Marcar esos nodos como `DESINCronizado`
3. Notificar a los Engines responsables de actualizar los nodos afectados
4. Una vez actualizados, marcarlos como `SINCRONIZADO`

---

## Los cuatro niveles de INIT

`INIT` no es un solo comando. Es un proceso de cuatro niveles que nunca modifica un proyecto sin antes entenderlo.

### Nivel 0: `srie identity`

**Establece quién es el nodo antes de tocar cualquier archivo.**

```
srie identity [ruta]
```

| Elemento | Descripción |
|----------|-------------|
| Entrada | Ruta del proyecto (o directorio actual) |
| Salida | `SDOS/IDENTITY.yaml` |
| Acción | Crea o valida la identidad del nodo: domain_id, tipo, cadena de firmas |
| Modifica | Crea `SDOS/IDENTITY.yaml` si no existe |
| Código salida | 0 = identidad lista, 1 = identidad inválida, 2 = identidad revocada |

**Uso típico:**

```bash
$ srie identity ./mi-proyecto
🆔 SRIE Identity — ./mi-proyecto
  Domain ID: DOM-001
  Tipo: standalone
  Estado: ACTIVE
  Cadena de firmas: VÁLIDA
  Fingerprint: sha256:abc123...
```

### Nivel 1: `srie inspect`

**No toca nada. Solo observa.**

```
srie inspect [ruta]
```

| Elemento | Descripción |
|----------|-------------|
| Entrada | Ruta del proyecto (o directorio actual) |
| Salida | Reporte en terminal (no genera archivos) |
| Acción | Escanea y muestra: lenguaje, framework, estructura, documentación, git, tests |
| Modifica | NADA. Cero archivos. Cero directorios. |
| Código salida | 0 = proyecto detectado, 1 = directorio vacío, 2 = no existe |

**Uso típico:**

```bash
$ srie inspect ./mi-proyecto
📋 SRIE Inspect — ./mi-proyecto
  Lenguaje: Python 3.12
  Framework: Flask
  Documentación: README.md ✓
  Tests: pytest ✓
  Git: main (3 commits)
  Docker: docker-compose.yml ✓
  SDOS: No detectado
```

### Nivel 2: `srie analyze`

**Calcula indicadores. No modifica nada.**

```
srie analyze [ruta]
```

| Elemento | Descripción |
|----------|-------------|
| Entrada | Ruta del proyecto |
| Salida | Reporte en terminal (no genera archivos) |
| Acción | Ejecuta Indicators Engine en modo read-only |
| Modifica | NADA. Solo genera reporte en stdout. |
| Código salida | 0 = análisis completo, 1 = error de análisis |

**Uso típico:**

```bash
$ srie analyze ./mi-proyecto
📊 SRIE Analysis — ./mi-proyecto
  SRIE_SCORE: 45.2
  Nivel: L2
  Dominios:
    Gobernanza:  55 (L2)
    Desarrollo:  60 (L2)
    Arquitectura: 40 (L1)
    Memoria:     30 (L1)
    Seguridad:   50 (L2)
    Operación:   70 (L3)
    UX:          35 (L1)
    IA:          20 (L1)
  Próximo nivel: L3 requiere:
    - Arquitectura ≥ 50
    - Memoria ≥ 50
```

### Nivel 3: `srie bootstrap`

**Ahora sí crea SDOS.**

```
srie bootstrap [ruta] [--force]
```

| Elemento | Descripción |
|----------|-------------|
| Entrada | Ruta del proyecto |
| Salida | `SDOS/` con todos los archivos iniciales |
| Acción | Crea estructura SDOS, genera PROJECT_STATE.md y STATE.md |
| Modifica | Solo crea archivos dentro de `SDOS/` |
| Flags | `--force` sobreescribe SDOS/ si existe |
| Código salida | 0 = bootstrap completo, 1 = error, 2 = ya existe (usar --force) |

**Uso típico:**

```bash
$ srie bootstrap ./mi-proyecto
🚀 SRIE Bootstrap — ./mi-proyecto
  ✔ SDOS/STATE.md creado
  ✔ SDOS/PROJECT_STATE.md creado
  ✔ SDOS/PROJECT.yaml creado
  ✔ SDOS/GRAPH.md creado (8 nodos)
  ✔ memory/index.md creado
  ✔ knowledge/index.md creado
  Nivel inicial: L2
  Proyecto listo. Ejecuta 'srie discover' para comenzar el ciclo.
```

### Regla fundamental

**Nunca ejecutes `srie bootstrap` sin antes ejecutar `srie identity`, `srie inspect` y `srie analyze`.** El sistema debe saber quién es, entender el proyecto y analizarlo antes de tocarlo.

---

## Contrato de proyecto

```yaml
project_contract:
  nombre: "Proyecto SRIE"
  version: "1.0.0"

  estructura_obligatoria:
    - "SDOS/STATE.md"
    - "SDOS/PROJECT_STATE.md"
    - "SDOS/PROJECT.yaml"
    - "SDOS/GRAPH.md"
    - "memory/index.md"
    - "knowledge/index.md"

  estructura_recomendada:
    - "README.md"
    - "ARCHITECTURE.md"
    - "DESIGN.md"
    - "capabilities/"
    - "memory/ADR/"
    - "knowledge/patterns/"
    - "knowledge/anti_patterns/"

  reglas:
    - nombre: "No modificacion fuera de SDOS"
      descripcion: "NINGÚN Engine puede modificar archivos fuera de SDOS/ sin autorización explícita en su manifest"
      sancion: "El Engine se marca como VIOLATOR y requiere revisión"

    - nombre: "Estado siempre actualizado"
      descripcion: "STATE.md debe reflejar la fase actual del ciclo SDOS en todo momento"
      sancion: "El ciclo se detiene hasta que STATE.md sea correcto"

    - nombre: "Evidencia siempre presente"
      descripcion: "Toda ejecución de Engine debe generar evidencia en SDOS/evidence/"
      sancion: "La iteración se marca como INCOMPLETA"

    - nombre: "Grafo siempre sincronizado"
      descripcion: "GRAPH.md debe actualizarse cuando se modifica cualquier nodo del proyecto"
      sancion: "El nodo se marca como DESINCronizado"

    - nombre: "Progresión de niveles"
      descripcion: "No se puede saltar niveles de madurez. L0 → L1 → L2 → L3 → L4 → L5"
      sancion: "El nivel se recalcula y se rechaza el avance"

  niveles_init:
    - nivel: 0
      comando: "srie identity"
      accion: "Crea o valida la identidad del nodo. Genera SDOS/IDENTITY.yaml."
    - nivel: 1
      comando: "srie inspect"
      accion: "Solo observa. No genera archivos."
    - nivel: 2
      comando: "srie analyze"
      accion: "Calcula indicadores. No genera archivos."
    - nivel: 3
      comando: "srie bootstrap"
      accion: "Crea estructura SDOS completa."

  codigos_salida_globales:
    - codigo: 0
      significado: "Operación exitosa"
    - codigo: 1
      significado: "Error de entrada"
    - codigo: 2
      significado: "Error de estado"
    - codigo: 99
      significado: "Error desconocido"
```
