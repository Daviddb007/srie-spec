# SRIE RUNTIME — El Sistema Vivo

---

## ¿Qué es este documento?

RUNTIME.md define el **Runtime de SRIE** — el sistema vivo que hace que toda la especificación cobre existencia. No es un Engine. No es una Capability. Es el **entorno de ejecución** que mantiene el sistema operando: carga Engines, gestiona memoria, ejecuta el event loop, planifica tareas, mantiene el Digital Twin y se observa a sí mismo.

```
ESPECIFICACIÓN (lo que SRIE debe hacer)
        ↓
RUNTIME (cómo SRIE existe)
        ↓
IMPLEMENTACIÓN (código que lo ejecuta)
```

---

## Arquitectura del Runtime

```
┌────────────────────────────────────────────────────────────────┐
│                        SRIE RUNTIME                             │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                        KERNEL                            │  │
│  │  ┌─────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  │  │
│  │  │  EVENT  │  │ SCHEDULER │  │ ENGINE   │  │ LIFECYCLE│  │  │
│  │  │  LOOP   │  │           │  │ LOADER   │  │ MANAGER  │  │  │
│  │  └─────────┘  └──────────┘  └──────────┘  └──────────┘  │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌────────────────────────┐  │
│  │  MEMORY     │  │  KNOWLEDGE  │  │  DIGITAL TWIN          │  │
│  │  MANAGER    │  │  MANAGER    │  │  MANAGER               │  │
│  └─────────────┘  └─────────────┘  └────────────────────────┘  │
│                                                                 │
│  ┌─────────────────────┐  ┌───────────────────────────────┐    │
│  │  RESOURCE BROKER    │  │  EXECUTION LAYER              │    │
│  └─────────────────────┘  └───────────────────────────────┘    │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    OBSERVABILITY                          │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐               │  │
│  │  │  LOGS    │  │  METRICS │  │  TRACES  │               │  │
│  │  └──────────┘  └──────────┘  └──────────┘               │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
└────────────────────────────────────────────────────────────────┘
```

---

## Componentes del Runtime

| Componente | Responsabilidad | Documento |
|------------|-----------------|-----------|
| Kernel | Orquestar el ciclo de vida del Runtime | KERNEL.md |
| Event Loop | Procesar eventos del sistema y externos | EVENT_LOOP.md |
| Scheduler | Planificar ejecución de Engines y tareas | SCHEDULER.md |
| Engine Loader | Cargar, verificar y ejecutar Engines | KERNEL.md |
| Lifecycle Manager | Gestionar inicio, pausa, parada del sistema | LIFECYCLE.md |
| Memory Manager | Gestionar la memoria del sistema (ADR, contextos) | — |
| Knowledge Manager | Gestionar el conocimiento (patrones, casos) | — |
| Digital Twin Manager | Mantener el Twin actualizado | INDICATOR_MODEL.md |
| Resource Broker | Resolver capacidades → recursos | RESOURCE_BROKER.md |
| Execution Layer | Ejecutar acciones en recursos | EXECUTION_LAYER.md |
| Observability | Logs, métricas y trazas del propio Runtime | OBSERVABILITY.md |

---

## Runtime Manifest

Cada instancia de SRIE tiene `SDOS/RUNTIME.yaml`:

```yaml
# SDOS/RUNTIME.yaml
runtime:
  version: "1.0.0"
  estado: "RUNNING"               # STOPPED | STARTING | RUNNING | PAUSED | STOPPING | FAILED

  kernel:
    version: "1.0.0"
    event_loop: true
    scheduler: true

  engines_cargados:
    - nombre: "Discovery Engine"
      version: "1.0.0"
      estado: "IDLE"
    - nombre: "Indicators Engine"
      version: "1.0.0"
      estado: "IDLE"

  resources_disponibles: 15
  resources_activos: 12

  memoria:
    total_adrs: 12
    total_contextos: 3
    ultimo_acceso: "2026-07-15T10:00:00Z"

  observabilidad:
    logs: true
    metrics: true
    traces: true
    nivel_log: "INFO"

  ultimo_heartbeat: "2026-07-15T10:00:00Z"
  uptime_segundos: 86400
```

---

## Ciclo de vida del Runtime

```
STOPPED
    │
    ▼
STARTING ──── Cargar configuración, verificar Foundation, iniciar Kernel
    │
    ▼
RUNNING ───── Runtime operando normalmente
    │
    ├──► PAUSED ──── Pausa programada (mantiene estado en memoria)
    │       │
    │       ▼
    │   RUNNING
    │
    ├──► DEGRADED ── Error no crítico, funcionamiento parcial
    │       │
    │       ▼
    │   RUNNING (si se resuelve) o STOPPING (si empeora)
    │
    └──► FAILED ──── Error crítico, requiere intervención
    │
    ▼
STOPPING ──── Guardar estado, cerrar conexiones, detener Engines
    │
    ▼
STOPPED
```
