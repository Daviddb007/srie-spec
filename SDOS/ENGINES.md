# ENGINES — Catálogo de Motores, Responsabilidades y Contratos

---

## Contrato unificado de Engine

Todos los motores del sistema SRIE deben cumplir exactamente el mismo contrato de ejecución. Esto garantiza que cualquier Engine pueda ser reemplazado, probado o depurado de la misma manera.

### El ciclo de ejecución de un Engine

```
┌──────────────────────────────────────────────────┐
│                 ENGINE CYCLE                      │
│                                                    │
│   INPUT                                            │
│     │                                               │
│     ▼                                               │
│   VALIDATION                                        │
│     │                                               │
│     ▼                                               │
│   EXECUTION                                         │
│     │                                               │
│     ▼                                               │
│   VERIFICATION                                      │
│     │                                               │
│     ▼                                               │
│   EVIDENCE                                          │
│     │                                               │
│     ▼                                               │
│   MEMORY                                            │
│     │                                               │
│     ▼                                               │
│   EXIT                                              │
│                                                    │
└──────────────────────────────────────────────────┘
```

### Definición de cada etapa

| Etapa | Descripción | ¿Qué produce? |
|-------|-------------|----------------|
| **INPUT** | Recibe los archivos, parámetros y estado necesarios para ejecutarse | Contexto de ejecución validado |
| **VALIDATION** | Verifica que las entradas cumplen el contrato esperado | Resultado de validación (válido/error) |
| **EXECUTION** | Ejecuta la transformación principal del Engine | Salida del Engine |
| **VERIFICATION** | Confirma que la salida cumple los criterios de calidad | Resultado de verificación (OK/fallo) |
| **EVIDENCE** | Registra la ejecución en archivos de evidencia | Archivos de evidencia |
| **MEMORY** | Actualiza la memoria del sistema con decisiones y resultados | Actualización de Memory |
| **EXIT** | Finaliza la ejecución y devuelve el control al ciclo SDOS | Código de salida (0 = éxito, 1+ = error) |

### Formato del contrato

```yaml
engine:
  nombre: string
  version: string
  contrato:
    input:
      archivos_requeridos:
        - path: string          # Archivos que deben existir antes de ejecutar
          opcional: boolean
      parametros:
        - nombre: string
          tipo: string
          requerido: boolean
          descripcion: string
    output:
      archivos_generados:
        - path: string          # Archivos que genera el Engine
          condicion: string     # Bajo qué condición se genera
      estado_salida:
        exito: string           # Código de éxito
        error: string           # Código de error
    reglas:
      - string                  # Reglas de comportamiento
    validaciones:
      pre:                      # Validaciones antes de ejecutar
        - string
      post:                     # Validaciones después de ejecutar
        - string
    memoria:
      actualiza: [string]       # Qué archivos de memoria modifica
      registros: [string]       # Qué decisiones registra
```

---

## Catálogo de Engines

---

### 1. Discovery Engine

| Campo | Valor |
|-------|-------|
| **Objetivo** | Escanear el proyecto completo y generar un inventario estructurado de su estado actual |
| **Orden de ejecución** | 1 (primero después de BOOT) |
| **Dependencias** | Ninguna |

#### Contrato

```yaml
engine:
  nombre: Discovery Engine
  contrato:
    input:
      archivos_requeridos: []
      parametros:
        - nombre: project_path
          tipo: string
          requerido: true
          descripcion: Ruta absoluta del proyecto a descubrir
    output:
      archivos_generados:
        - path: SDOS/PROJECT_STATE.md
          condicion: siempre
      estado_salida:
        exito: PROJECT_DISCOVERED
        error: PROJECT_NOT_FOUND
    reglas:
      - No modifica ningún archivo del proyecto
      - Solo lee el sistema de archivos
      - Debe poder ejecutarse en cualquier momento
      - El resultado debe ser determinista
    validaciones:
      pre:
        - La ruta del proyecto existe y es accesible
      post:
        - PROJECT_STATE.md contiene todas las secciones del contrato
        - No se modificó ningún archivo del proyecto
    memoria:
      actualiza: []
      registros: []
```

#### Archivos que lee

```
<proyecto>/**                                # Sistema de archivos completo
<proyecto>/*.py, *.js, *.ts, *.go, *.rs      # Archivos de código fuente
<proyecto>/package.json, Cargo.toml, pyproject.toml  # Dependencias
<proyecto>/README.md, ARCHITECTURE.md        # Documentación
<proyecto>/.git                              # Repositorio git
<proyecto>/Dockerfile, docker-compose.yml    # Contenedores
<proyecto>/.github, .gitlab-ci.yml           # CI/CD
```

#### Archivos que genera

```
SDOS/PROJECT_STATE.md                        # Inventario del proyecto
```

#### Indicadores

- Tiempo de ejecución
- Cantidad de archivos descubiertos
- Profundidad del árbol de directorios

---

### 2. Indicators Engine

| Campo | Valor |
|-------|-------|
| **Objetivo** | Calcular el nivel de madurez SRIE del proyecto en los 8 dominios |
| **Orden de ejecución** | 2 |
| **Dependencias** | Discovery Engine |

#### Contrato

```yaml
engine:
  nombre: Indicators Engine
  contrato:
    input:
      archivos_requeridos:
        - path: SDOS/PROJECT_STATE.md
          opcional: false
      parametros:
        - nombre: dominios
          tipo: string[]
          requerido: false
          descripcion: Lista de dominios a evaluar (por defecto, todos)
    output:
      archivos_generados:
        - path: SDOS/SRIE_REPORT.md
          condicion: siempre
      estado_salida:
        exito: INDICATORS_CALCULATED
        error: PROJECT_STATE_NOT_FOUND
    reglas:
      - Todas las métricas deben ser deterministas
      - Misma entrada = misma salida
      - No modifica archivos del proyecto
    validaciones:
      pre:
        - PROJECT_STATE.md existe y es válido
      post:
        - SRIE_REPORT.md contiene puntuaciones para todos los dominios
        - El nivel de madurez está correctamente calculado
    memoria:
      actualiza: []
      registros: []
```

#### Archivos que lee

```
SDOS/PROJECT_STATE.md
SDOS/SRIE_INDICATORS.md                     # Definición de métricas
```

#### Archivos que genera

```
SDOS/SRIE_REPORT.md                          # Reporte de madurez
```

---

### 3. DX Engine

| Campo | Valor |
|-------|-------|
| **Objetivo** | Analizar la experiencia de desarrollo y generar un diagnóstico en 5 categorías |
| **Orden de ejecución** | 3 |
| **Dependencias** | Discovery Engine, Indicators Engine |

#### Contrato

```yaml
engine:
  nombre: DX Engine
  contrato:
    input:
      archivos_requeridos:
        - path: SDOS/PROJECT_STATE.md
          opcional: false
        - path: SDOS/SRIE_REPORT.md
          opcional: false
      parametros: []
    output:
      archivos_generados:
        - path: SDOS/DX_REPORT.md
          condicion: siempre
      estado_salida:
        exito: DX_COMPLETED
        error: INSUFFICIENT_DATA
    reglas:
      - Debe responder las 5 preguntas (KEEP, SIMPLIFY, REMOVE, CREATE, BLOCKERS)
      - Cada respuesta debe basarse en evidencia de PROJECT_STATE o SRIE_REPORT
      - No incluye recomendaciones de implementación (eso es del Planner)
    validaciones:
      pre:
        - PROJECT_STATE.md y SRIE_REPORT.md existen
      post:
        - DX_REPORT.md contiene las 5 categorías con al menos una entrada cada una
    memoria:
      actualiza: []
      registros: []
```

#### Archivos que genera

```
SDOS/DX_REPORT.md                            # Diagnóstico de experiencia de desarrollo
```

---

### 4. Planner Engine

| Campo | Valor |
|-------|-------|
| **Objetivo** | Generar un plan de evolución del proyecto basado en el diagnóstico DX |
| **Orden de ejecución** | 4 |
| **Dependencias** | DX Engine |

#### Contrato

```yaml
engine:
  nombre: Planner Engine
  contrato:
    input:
      archivos_requeridos:
        - path: SDOS/DX_REPORT.md
          opcional: false
        - path: SDOS/SRIE_REPORT.md
          opcional: false
      parametros:
        - nombre: objetivo_iteracion
          tipo: string
          requerido: false
          descripcion: Objetivo específico para esta iteración
    output:
      archivos_generados:
        - path: SDOS/PLAN.md
          condicion: siempre
      estado_salida:
        exito: PLAN_GENERATED
        error: DX_REPORT_NOT_FOUND
    reglas:
      - No genera tareas de código. Genera objetivos de evolución.
      - Cada objetivo debe tener un indicador de éxito medible.
      - Máximo 3 objetivos por iteración.
    validaciones:
      pre:
        - DX_REPORT.md y SRIE_REPORT.md existen
      post:
        - PLAN.md contiene objetivos con indicadores de éxito
        - Cada objetivo está priorizado
    memoria:
      actualiza: []
      registros: []
```

#### Archivos que genera

```
SDOS/PLAN.md                                 # Plan de evolución
```

---

### 5. Capability Engine

| Campo | Valor |
|-------|-------|
| **Objetivo** | Construir capacidades como microproyectos autocontenidos siguiendo el plan |
| **Orden de ejecución** | 5 |
| **Dependencias** | Planner Engine |

#### Contrato

```yaml
engine:
  nombre: Capability Engine
  contrato:
    input:
      archivos_requeridos:
        - path: SDOS/PLAN.md
          opcional: false
      parametros:
        - nombre: capability_name
          tipo: string
          requerido: true
          descripcion: Nombre de la capability a construir
        - nombre: capability_type
          tipo: string
          requerido: false
          descripcion: Tipo de capability (nueva, modificación, reparación)
    output:
      archivos_generados:
        - path: capabilities/<nombre>/<nombre>.md
          condicion: siempre
        - path: capabilities/<nombre>/<nombre>_design.md
          condicion: siempre
        - path: capabilities/<nombre>/<nombre>_api.md
          condicion: si aplica
        - path: capabilities/<nombre>/<nombre>_tests.md
          condicion: siempre
        - path: capabilities/<nombre>/<nombre>_metrics.md
          condicion: siempre
        - path: capabilities/<nombre>/<nombre>_memory.md
          condicion: siempre
        - path: capabilities/<nombre>/<nombre>_release.md
          condicion: siempre
      estado_salida:
        exito: CAPABILITY_BUILT
        error: PLAN_NOT_FOUND
    reglas:
      - Cada capability es un microproyecto autocontenido
      - No se mezclan capacidades en el mismo cambio
      - Debe generar todos los archivos de la estructura de capability
    validaciones:
      pre:
        - PLAN.md existe
        - La capability no existe ya (si es nueva)
      post:
        - Todos los archivos de la capability fueron generados
        - La capability tiene tests definidos
    memoria:
      actualiza: []
      registros:
        - Decisiones de diseño tomadas durante la construcción
```

#### Archivos que genera

```
capabilities/<nombre>/
├── <nombre>.md
├── <nombre>_design.md
├── <nombre>_api.md                          # Opcional
├── <nombre>_tests.md
├── <nombre>_metrics.md
├── <nombre>_memory.md
└── <nombre>_release.md
```

---

### 6. Sandbox Engine

| Campo | Valor |
|-------|-------|
| **Objetivo** | Validar capacidades en un entorno aislado antes de integrarlas al proyecto principal |
| **Orden de ejecución** | 6 |
| **Dependencias** | Capability Engine |

#### Contrato

```yaml
engine:
  nombre: Sandbox Engine
  contrato:
    input:
      archivos_requeridos:
        - path: capabilities/<nombre>/
          opcional: false
      parametros:
        - nombre: capability_name
          tipo: string
          requerido: true
          descripcion: Capacidad a validar
    output:
      archivos_generados:
        - path: SDOS/VERIFICATION_REPORT.md
          condicion: siempre
      estado_salida:
        exito: VERIFIED_READY
        error: VERIFICATION_FAILED
        alerta: VERIFIED_WITH_WARNINGS
    reglas:
      - Opera sobre un clon temporal del proyecto
      - Nunca modifica el proyecto principal
      - Debe ejecutar: tests, benchmarks, seguridad, estrés, impacto
    validaciones:
      pre:
        - La capability existe y está completa
        - Hay suficiente espacio en disco para el clon
      post:
        - VERIFICATION_REPORT.md contiene resultados de todas las validaciones
        - No hay cambios en el proyecto principal
    memoria:
      actualiza: []
      registros:
        - Resultados de validación
        - Problemas encontrados
```

#### Archivos que genera

```
SDOS/VERIFICATION_REPORT.md                  # Resultados de validación
```

---

### 7. Repair Engine (SIDC)

| Campo | Valor |
|-------|-------|
| **Objetivo** | Detectar y corregir desviaciones en el estado del proyecto cuando el ciclo SDOS se interrumpe |
| **Orden de ejecución** | 7 (bajo demanda, no forma parte del ciclo estándar) |
| **Dependencias** | Discovery Engine |

#### Contrato

```yaml
engine:
  nombre: Repair Engine (SIDC)
  contrato:
    input:
      archivos_requeridos:
        - path: SDOS/STATE.md
          opcional: false
      parametros:
        - nombre: modo
          tipo: string
          requerido: false
          descripcion: "auto | manual"
          valores_posibles: ["auto", "manual"]
    output:
      archivos_generados:
        - path: SDOS/REPAIR_REPORT.md
          condicion: siempre
      estado_salida:
        exito: PROJECT_REPAIRED
        error: REPAIR_FAILED
        advertencia: REPAIR_PARTIAL
    reglas:
      - Solo actúa cuando STATE.md indica un ciclo incompleto
      - En modo auto, intenta reparar automáticamente
      - En modo manual, genera un reporte de acciones necesarias
    validaciones:
      pre:
        - STATE.md existe
        - Se detecta una desviación en el ciclo SDOS
      post:
        - REPAIR_REPORT.md documenta todas las acciones tomadas o requeridas
    memoria:
      actualiza:
        - SDOS/STATE.md
      registros:
        - Desviaciones detectadas
        - Acciones de reparación ejecutadas
```

#### Archivos que lee

```
SDOS/STATE.md
SDOS/PROJECT_STATE.md
```

#### Archivos que genera

```
SDOS/REPAIR_REPORT.md                        # Reporte de reparación
```

---

### 8. Memory Engine

| Campo | Valor |
|-------|-------|
| **Objetivo** | Gestionar la memoria estructurada del proyecto: decisiones de ingeniería, contexto y evolución |
| **Orden de ejecución** | 8 (al final del ciclo, en LEARN) |
| **Dependencias** | Todos los Engines anteriores |

#### Contrato

```yaml
engine:
  nombre: Memory Engine
  contrato:
    input:
      archivos_requeridos:
        - path: SDOS/
          opcional: false
      parametros:
        - nombre: accion
          tipo: string
          requerido: false
          descripcion: "consolidar | actualizar | auditar"
    output:
      archivos_generados:
        - path: memory/ADR/<fecha>-<titulo>.md
          condicion: si hay nuevas decisiones
      estado_salida:
        exito: MEMORY_UPDATED
        error: MEMORY_CORRUPTED
    reglas:
      - No elimina memoria existente. Solo agrega o actualiza.
      - Cada decisión de ingeniería debe tener un ADR
      - Memory debe ser legible por humanos y por IA
    validaciones:
      pre:
        - La memoria existente es accesible
      post:
        - Las nuevas decisiones están registradas en ADR
        - El índice de memoria está actualizado
    memoria:
      actualiza:
        - memory/index.md
        - memory/ADR/
      registros:
        - Decisiones de ingeniería
        - Estado del proyecto al finalizar la iteración
```

#### Archivos que lee

```
SDOS/                                       # Toda la evidencia del ciclo
memory/                                      # Memoria existente
```

#### Archivos que genera

```
memory/ADR/<fecha>-<titulo>.md               # Nuevos ADR
memory/index.md                               # Índice actualizado
```

---

### 9. Knowledge Engine

| Campo | Valor |
|-------|-------|
| **Objetivo** | Consolidar la experiencia de la iteración en patrones y antipatrones reutilizables |
| **Orden de ejecución** | 9 (después de Memory Engine, en LEARN) |
| **Dependencias** | Memory Engine |

#### Contrato

```yaml
engine:
  nombre: Knowledge Engine
  contrato:
    input:
      archivos_requeridos:
        - path: memory/
          opcional: false
      parametros: []
    output:
      archivos_generados:
        - path: knowledge/patterns/<nombre>.md
          condicion: si se identificó un patrón exitoso
        - path: knowledge/anti_patterns/<nombre>.md
          condicion: si se identificó un antipatrón
      estado_salida:
        exito: KNOWLEDGE_UPDATED
        error: NO_PATTERNS_IDENTIFIED
    reglas:
      - Cada iteración debe producir al menos un patrón o antipatrón
      - Los patrones deben ser generalizables (no depender de un contexto específico)
      - Los antipatrones deben incluir la solución correcta
    validaciones:
      pre:
        - La memoria de la iteración está consolidada
      post:
        - Los patrones/antipatrones están en el formato definido
        - El índice de knowledge está actualizado
    memoria:
      actualiza:
        - knowledge/index.md
      registros:
        - Patrones y antipatrones identificados
```

#### Archivos que lee

```
memory/                                       # Decisiones de la iteración
knowledge/                                    # Conocimiento existente
```

#### Archivos que genera

```
knowledge/patterns/<nombre>.md               # Nuevos patrones
knowledge/anti_patterns/<nombre>.md           # Nuevos antipatrones
knowledge/index.md                             # Índice actualizado
```

---

### 10. Governance Engine

| Campo | Valor |
|-------|-------|
| **Objetivo** | Verificar que el proyecto cumple con los estándares de gobierno definidos para su nivel de madurez |
| **Orden de ejecución** | 10 (en LEARN, después de Knowledge) |
| **Dependencias** | Todos los Engines |

#### Contrato

```yaml
engine:
  nombre: Governance Engine
  contrato:
    input:
      archivos_requeridos:
        - path: SDOS/SRIE_REPORT.md
          opcional: false
      parametros: []
    output:
      archivos_generados:
        - path: SDOS/GOVERNANCE_REPORT.md
          condicion: siempre
      estado_salida:
        exito: GOVERNANCE_COMPLIANT
        error: GOVERNANCE_NON_COMPLIANT
    reglas:
      - Verifica que todos los archivos requeridos para el nivel actual existen
      - Verifica que los contratos se cumplen
      - No modifica archivos del proyecto
    validaciones:
      pre:
        - SRIE_REPORT.md existe
      post:
        - GOVERNANCE_REPORT.md lista todos los checks con su estado
    memoria:
      actualiza:
        - SDOS/STATE.md (nivel de madurez)
      registros:
        - Resultados de auditoría de gobierno
```

#### Archivos que genera

```
SDOS/GOVERNANCE_REPORT.md                    # Auditoría de gobierno
```

---

### 11. Deployment Engine

| Campo | Valor |
|-------|-------|
| **Objetivo** | Integrar capacidades validadas al proyecto principal y gestionar el despliegue |
| **Orden de ejecución** | 11 (después de VERIFY) |
| **Dependencias** | Sandbox Engine |

#### Contrato

```yaml
engine:
  nombre: Deployment Engine
  contrato:
    input:
      archivos_requeridos:
        - path: SDOS/VERIFICATION_REPORT.md
          opcional: false
          condicion: debe ser VERIFIED_READY
      parametros:
        - nombre: capability_name
          tipo: string
          requerido: true
          descripcion: Capacidad a desplegar
        - nombre: modo
          tipo: string
          requerido: false
          descripcion: "full | dry-run"
    output:
      archivos_generados:
        - path: SDOS/DEPLOY_REPORT.md
          condicion: siempre
      estado_salida:
        exito: DEPLOYED
        error: DEPLOY_FAILED
        advertencia: DEPLOYED_WITH_ROLLBACK
    reglas:
      - Solo despliega si VERIFICATION_REPORT.md indica READY
      - Cada deploy debe ser reversible
      - En modo dry-run, no modifica el proyecto
    validaciones:
      pre:
        - VERIFICATION_REPORT.md existe y es READY
        - Hay un plan de rollback definido
      post:
        - La capability está integrada al proyecto
        - DEPLOY_REPORT.md documenta el despliegue
    memoria:
      actualiza:
        - SDOS/STATE.md
      registros:
        - Historial de deploys
        - Rollbacks ejecutados
```

#### Archivos que genera

```
SDOS/DEPLOY_REPORT.md                        # Reporte de despliegue
```

---

## Mapa de dependencias entre Engines

```
Discovery  ◄──────────────────────────────────────────┐
    │                                                  │
    ▼                                                  │
Indicators                                             │
    │                                                  │
    ▼                                                  │
DX                                                     │
    │                                                  │
    ▼                                                  │
Planner                                                │
    │                                                  │
    ▼                                                  │
Capability                                             │
    │                                                  │
    ▼                                                  │
Sandbox                                                │
    │                                                  │
    ├──────────► Deployment                            │
    │                │                                 │
    ▼                ▼                                 │
Repair (SIDC)    Memory                                │
                     │                                 │
                     ▼                                 │
                  Knowledge                            │
                     │                                 │
                     ▼                                 │
                 Governance ───────────────────────────┘
                     │
                     ▼
                 STATE.md actualizado
                      │
                      ▼
                  REPEAT ◄── SDOS CYCLE
```

---

## Glosario de contratos

| Término | Definición |
|---------|------------|
| **Input** | Todo lo que un Engine necesita para ejecutarse: archivos, parámetros, estado |
| **Output** | Todo lo que un Engine produce al ejecutarse: archivos, estado, código de salida |
| **Validación pre** | Condiciones que deben cumplirse antes de la ejecución |
| **Validación post** | Condiciones que deben cumplirse después de la ejecución |
| **Evidencia** | Archivos generados que documentan la ejecución |
| **Memoria** | Archivos actualizados que registran decisiones y contexto |
| **Código de salida** | 0 = éxito, 1 = error, 2+ = estados específicos del Engine |
| **Regla** | Comportamiento inviolable que el Engine debe seguir |
