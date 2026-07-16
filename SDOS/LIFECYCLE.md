# LIFECYCLE — Gestión del Ciclo de Vida del Sistema

---

## ¿Qué es este documento?

LIFECYCLE.md define la **gestión del ciclo de vida** del Runtime SRIE. Desde que arranca hasta que se detiene, pasando por pausas, degradaciones, fallos y recuperaciones.

---

## Estados del Runtime

```yaml
runtime_states:
  STOPPED: "Sistema detenido. Sin procesos activos."
  STARTING: "Sistema iniciándose. Cargando componentes."
  RUNNING: "Sistema operando normalmente."
  PAUSED: "Sistema pausado. Mantiene estado en memoria."
  DEGRADED: "Sistema funcionando parcialmente. Algún componente falló."
  FAILED: "Sistema en estado de error crítico."
  STOPPING: "Sistema deteniéndose. Guardando estado."
```

---

## Transiciones de estado

```yaml
transiciones:
  STOPPED → STARTING:
    trigger: "srie start o arranque del sistema"
    accion: "Iniciar Kernel"

  STARTING → RUNNING:
    trigger: "Todos los componentes cargaron exitosamente"
    accion: "Comenzar Event Loop"

  RUNNING → PAUSED:
    trigger: "Pausa programada o manual"
    accion: "Detener Scheduler, preservar estado en memoria"
    permite: "Reanudar en < 1 hora sin pérdida"

  PAUSED → RUNNING:
    trigger: "Reanudación"
    accion: "Reiniciar Scheduler, continuar desde donde quedó"

  RUNNING → DEGRADED:
    trigger: "Engine o recurso crítico falla"
    accion: "Registrar fallo, intentar recuperación automática"
    recuperacion: "Reintentar Engine o recurso cada 30s (máx 5 intentos)"

  DEGRADED → RUNNING:
    trigger: "Componente recuperado"
    accion: "Reintegrar componente, normalizar estado"

  RUNNING → FAILED:
    trigger: "Error no recuperable en componente crítico"
    accion: "Intentar apagado seguro, si no es posible: forced stop"

  RUNNING → STOPPING:
    trigger: "srie stop o shutdown"
    accion: "Apagado seguro"

  DEGRADED → STOPPING:
    trigger: "srie stop o shutdown"
    accion: "Apagado seguro (guardando estado actual)"

  FAILED → STOPPING:
    trigger: "Intento de recuperación"
    accion: "Guardar lo que se pueda, reconstruir desde evidencia"

  STOPPING → STOPPED:
    trigger: "Apagado completado"
    accion: "Liberar recursos"
```

---

## Políticas de recuperación

```yaml
recovery_policies:
  engine_failure:
    reintentos: 3
    backoff_ms: [1000, 5000, 15000]
    si_agota: "Marcar Engine como ERROR, continuar sin él"

  resource_failure:
    reintentos: 5
    backoff_ms: [1000, 2000, 5000, 10000, 30000]
    si_agota: "Marcar recurso como OFFLINE, buscar alternativa"

  memory_corruption:
    accion: "Restaurar desde última evidencia válida"
    si_no_hay: "Reconstruir desde Discovery"

  kernel_failure:
    accion: "Reinicio completo del Runtime"
    preservar: "Evidencia, Registry, Memoria"
    reconstruir: "Digital Twin desde Discovery"
```

---

## Apagado seguro

```yaml
graceful_shutdown:
  secuencia:
    1. "Notificar a todos los Engines: prepare for shutdown"
       timeout_ms: 5000
    2. "Esperar tareas en curso del Scheduler"
       timeout_ms: 10000
    3. "Guardar estado del Runtime"
    4. "Detener Event Loop"
    5. "Cerrar conexiones de recursos"
    6. "Guardar evidencia del apagado"
    7. "Emitir evento RUNTIME_STOPPED"
    8. "Salir"

  forcado:
    trigger: "No responde después de timeout_total"
    accion: "Terminar procesos, reconstruir desde evidencia al reiniciar"
    riesgo: "Pérdida de estado no guardado"
```

---

## Lifecycle Manifest

```yaml
# SDOS/LIFECYCLE.yaml
lifecycle:
  estado_actual: "RUNNING"
  uptime_segundos: 86400
  ultimo_cambio_estado: "2026-07-15T10:00:00Z"
  historial_estados:
    - estado: "STARTING"
      desde: "2026-07-14T10:00:00Z"
      hasta: "2026-07-14T10:00:05Z"
    - estado: "RUNNING"
      desde: "2026-07-14T10:00:05Z"
      hasta: null
  fallos_recientes: []
  ultimo_apagado: null
```
