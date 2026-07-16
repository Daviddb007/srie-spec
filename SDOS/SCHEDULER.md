# SCHEDULER — Planificador del Runtime

---

## ¿Qué es este documento?

SCHEDULER.md define el **planificador** del Runtime SRIE. Gestiona qué se ejecuta, cuándo y con qué recursos. No confundir con el Planner Engine (que decide QUÉ hacer). El Scheduler decide CUÁNDO hacerlo.

---

## Responsabilidades

```yaml
scheduler:
  responsabilidades:
    - "Planificar ejecución de Engines en el ciclo SDOS"
    - "Ejecutar tareas programadas (discovery cada N horas)"
    - "Gestionar prioridad entre tareas"
    - "Respetar límite de tareas simultáneas"
    - "Reintentar tareas fallidas"
```

---

## Tipos de tareas

```yaml
task_types:
  - tipo: "engine"
    descripcion: "Ejecución de un Engine del ciclo SDOS"
    prioridad: "ALTA"
    max_simultaneas: 1          # Solo un Engine a la vez en el ciclo

  - tipo: "programada"
    descripcion: "Tarea recurrente (discovery, indicadores)"
    prioridad: "MEDIA"
    max_simultaneas: 2

  - tipo: "evento"
    descripcion: "Tarea disparada por un evento"
    prioridad: "variable"
    max_simultaneas: 3

  - tipo: "mantenimiento"
    descripcion: "Limpieza, archivo, poda de datos"
    prioridad: "BAJA"
    max_simultaneas: 1
```

---

## Programación SDOS

```yaml
sdos_schedule:
  fases:
    DISCOVERY:
      trigger: "BOOT completado o evento DISCOVERY_TRIGGERED"
      engine: "Discovery Engine"
      duracion_esperada_ms: 30000
      max_simultaneas: 1

    INDICATORS:
      trigger: "Evento PROJECT_DISCOVERED o DISCOVERY_UPDATED"
      engine: "Indicators Engine"
      duracion_esperada_ms: 15000
      max_simultaneas: 1

    DX:
      trigger: "Evento INDICATORS_CALCULATED"
      engine: "DX Engine"
      duracion_esperada_ms: 20000
      max_simultaneas: 1

    PLAN:
      trigger: "Evento DIAGNOSIS_READY o INTENT_READY"
      engine: "Planner Engine"
      duracion_esperada_ms: 30000
      max_simultaneas: 1
```

---

## Tareas programadas

```yaml
scheduled_tasks:
  - nombre: "discovery_incremental"
    cron: "*/30 * * * *"           # Cada 30 minutos
    accion: "Ejecutar Discovery incremental"
    si_no_hay_cambios: "NO_ACTION_NEEDED (no cuenta como ejecución)"

  - nombre: "indicators_refresh"
    cron: "*/15 * * * *"           # Cada 15 minutos
    accion: "Refrescar indicadores"

  - nombre: "health_check_resources"
    cron: "*/5 * * * *"            # Cada 5 minutos
    accion: "Verificar salud de recursos"

  - nombre: "knowledge_consolidation"
    cron: "0 2 * * *"              # Cada día a las 2 AM
    accion: "Consolidar patrones y casos"

  - nombre: "memory_purge"
    cron: "0 3 * * 0"              # Cada domingo a las 3 AM
    accion: "Archivar memoria antigua (>1 año)"
```

---

## Cola de tareas

```yaml
task_queue:
  formato:
    id: string
    tipo: string
    prioridad: string
    accion: string
    params: {}
    creado: timestamp
    timeout_ms: integer
    reintentos: integer
    max_reintentos: 3

  ordenamiento:
    - "Primero por prioridad (CRÍTICA > ALTA > MEDIA > BAJA)"
    - "Dentro de misma prioridad: FIFO"

  limites:
    engine_simultaneo: 1
    programadas_simultaneas: 2
    total_max: 5
```

---

## Estados de una tarea

```yaml
task_states:
  PENDING: "En cola, esperando ejecución"
  RUNNING: "En ejecución"
  SUCCESS: "Completada exitosamente"
  FAILED: "Falló (reintentando)"
  TIMEOUT: "Excedió tiempo máximo"
  CANCELLED: "Cancelada por scheduler"
```

---

## Métricas del Scheduler

```yaml
scheduler_metrics:
  - "tareas_completadas"
  - "tareas_fallidas"
  - "tareas_timeout"
  - "cola_actual"
  - "tiempo_espera_promedio_ms"
  - "utilización (tareas_activas / max_simultaneas)"
```
