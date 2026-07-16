# OBSERVABILITY — Auto-Observabilidad del Sistema SRIE

---

## ¿Qué es este documento?

OBSERVABILITY.md define **cómo SRIE se observa a sí mismo**. No la observabilidad del proyecto que SRIE gobierna (eso es INDICATORS.md). Es la **observabilidad del Runtime**: logs, métricas y trazas del propio sistema.

Cuando algo falle, debes poder responder:

- ¿Qué Engine estaba ejecutándose?
- ¿Qué evento lo disparó?
- ¿Qué resource eligió?
- ¿Qué adapter ejecutó?
- ¿Cuánto duró?
- ¿Dónde falló?

---

## Los tres pilares

```yaml
observability:
  logs: "Eventos discretos del sistema (quién, qué, cuándo)"
  metrics: "Mediciones agregadas (cuánto, cuántos, qué tan rápido)"
  traces: "Recorrido de una operación a través del sistema"
```

---

## 1. Logs

Cada componente del Runtime emite logs estructurados:

```yaml
log:
  formato: "JSON Lines"
  nivel: "DEBUG | INFO | WARN | ERROR | FATAL"
  campos:
    - timestamp: ISO8601
    - nivel: string
    - componente: string     # kernel | event_loop | scheduler | engine | broker
    - evento: string
    - mensaje: string
    - metadata: {}
    - trace_id: string       # Para correlacionar con traces
    - ejecucion_id: string   # ID de la ejecución del Engine

  ejemplos:
    - {"ts":"2026-07-15T10:00:01Z","nivel":"INFO","componente":"kernel","evento":"RUNTIME_STARTED","mensaje":"Runtime iniciado","metadata":{"version":"1.0.0"}}
    - {"ts":"2026-07-15T10:00:02Z","nivel":"INFO","componente":"engine_loader","evento":"ENGINE_LOADED","mensaje":"Discovery Engine cargado","metadata":{"engine":"discovery","version":"1.0.0"}}
    - {"ts":"2026-07-15T10:00:05Z","nivel":"WARN","componente":"resource_broker","evento":"RESOURCE_DEGRADED","mensaje":"GitHub MCP con latencia alta","metadata":{"resource":"RES-GH-001","latencia_ms":2300}}
    - {"ts":"2026-07-15T10:00:10Z","nivel":"ERROR","componente":"scheduler","evento":"TASK_FAILED","mensaje":"Discovery Engine falló","metadata":{"task_id":"T-001","error":"timeout"}}
```

**Almacenamiento:**
```
SDOS/logs/
├── runtime.log              # Logs del Runtime (rotación diaria)
├── kernel.log               # Logs del kernel
├── event_loop.log           # Logs del event loop
├── scheduler.log            # Logs del scheduler
├── engines/                 # Logs por Engine
│   ├── discovery.log
│   ├── indicators.log
│   └── ...
└── resources/               # Logs de recursos
    └── broker.log
```

---

## 2. Metrics

Métricas agregadas del Runtime:

```yaml
metrics:
  event_loop:
    - "srie_event_loop_events_received_total"
    - "srie_event_loop_events_processed_total"
    - "srie_event_loop_events_failed_total"
    - "srie_event_loop_queue_depth"
    - "srie_event_loop_processing_duration_ms"

  scheduler:
    - "srie_scheduler_tasks_completed_total"
    - "srie_scheduler_tasks_failed_total"
    - "srie_scheduler_tasks_timeout_total"
    - "srie_scheduler_queue_depth"
    - "srie_scheduler_wait_duration_ms"

  engines:
    - "srie_engine_execution_duration_ms{engine='discovery'}"
    - "srie_engine_execution_duration_ms{engine='indicators'}"
    - "srie_engine_executions_total{engine='discovery',status='success'}"
    - "srie_engine_executions_total{engine='discovery',status='failed'}"

  resources:
    - "srie_resource_health{resource='RES-GH-001'}"
    - "srie_resource_latency_ms{resource='RES-GH-001'}"
    - "srie_resource_executions_total{resource='RES-GH-001'}"

  runtime:
    - "srie_runtime_uptime_seconds"
    - "srie_runtime_state{state='running'}"
    - "srie_runtime_memory_usage_mb"
```

**Almacenamiento:**
```
SDOS/metrics/
├── counters/                # Contadores acumulativos
├── gauges/                  # Mediciones puntuales
└── histograms/              # Distribuciones de duración
```

---

## 3. Traces

Traza completa de una operación a través del sistema:

```yaml
trace:
  id: "TRACE-001"
  operacion: "srie discover"
  inicio: "2026-07-15T10:00:00Z"
  fin: "2026-07-15T10:00:20Z"
  duracion_ms: 20000

  spans:
    - span_id: "SPAN-001"
      padre: null
      nombre: "Discovery Engine.execute"
      inicio: "2026-07-15T10:00:00Z"
      fin: "2026-07-15T10:00:20Z"
      duracion_ms: 20000
      estado: "SUCCESS"

    - span_id: "SPAN-002"
      padre: "SPAN-001"
      nombre: "Scanner.scan"
      inicio: "2026-07-15T10:00:01Z"
      fin: "2026-07-15T10:00:03Z"
      duracion_ms: 2000
      estado: "SUCCESS"

    - span_id: "SPAN-003"
      padre: "SPAN-001"
      nombre: "Classifier.classify"
      inicio: "2026-07-15T10:00:03Z"
      fin: "2026-07-15T10:00:08Z"
      duracion_ms: 5000
      estado: "SUCCESS"

    - span_id: "SPAN-004"
      padre: "SPAN-001"
      nombre: "Correlator.correlate"
      inicio: "2026-07-15T10:00:08Z"
      fin: "2026-07-15T10:00:15Z"
      duracion_ms: 7000
      estado: "SUCCESS"
```

**Almacenamiento:**
```
SDOS/traces/
├── TRACE-001.json
├── TRACE-002.json
└── ...
```

---

## Health endpoint

El Runtime expone un health endpoint interno:

```yaml
health:
  endpoint: "SDOS/HEALTH.json"
  contenido:
    estado: "RUNNING | DEGRADED | FAILED"
    uptime_segundos: 86400
    componentes:
      kernel: "OK"
      event_loop: "OK"
      scheduler: "OK"
      engines: {"discovery": "IDLE", "indicators": "IDLE"}
      resources: {"total": 15, "activos": 12, "degradados": 2, "offline": 1}
      memory: {"total_adrs": 12, "uso_mb": 45}
    ultimo_heartbeat: "2026-07-15T10:00:00Z"
```
