# EVENT LOOP — Bucle de Eventos del Runtime

---

## ¿Qué es este documento?

EVENT_LOOP.md define el **bucle de eventos** del Runtime SRIE. Es el corazón del sistema: recibe eventos, los encola, los despacha a los suscriptores correctos y gestiona la prioridad, el debounce y la entrega.

---

## Arquitectura del Event Loop

```
EVENT SOURCES
  ├── Engines (emiten eventos al completar)
  ├── Discovery (DESCOVERY_UPDATED)
  ├── Indicators (DOCUMENTATION_WARNING)
  ├── Intent Layer (INTENT_READY)
  ├── Scheduler (tareas programadas)
  ├── Resource Broker (HEALTH_CHANGED)
  ├── System (RUNTIME_STARTED, SHUTDOWN)
  └── External (cambios en archivos, git hooks)
        │
        ▼
┌────────────────────────────────────────────┐
│              EVENT LOOP                     │
│                                             │
│  ┌─────────┐  ┌────────┐  ┌─────────────┐  │
│  │  INPUT  │─►│  QUEUE │─►│  DISPATCHER │  │
│  │  QUEUE  │  │ (prior │  │  (pub/sub)  │  │
│  └─────────┘  │  ity)  │  └──────┬──────┘  │
│               └────────┘         │          │
│  ┌─────────┐                     ▼          │
│  │  DELAY  │              ┌──────────────┐  │
│  │  QUEUE  │              │  SUBSCRIBERS │  │
│  │(debounce│              │ (Engines +   │  │
│  │ + retry)│              │  Componentes)│  │
│  └─────────┘              └──────────────┘  │
│                                             │
└────────────────────────────────────────────┘
        │
        ▼
  EVENT PROCESSED / EVENT FAILED
```

---

## Flujo de un evento

```yaml
event_flow:
  1. RECEPCIÓN:
     - Validar formato del evento
     - Asignar ID único
     - Timestamp de recepción

  2. ENCOLAMIENTO:
     - Si el evento tiene prioridad CRÍTICA → cola prioritaria
     - Si no → cola normal
     - Si el evento requiere debounce → Delay Queue

  3. DESPACHO:
     - Consultar suscriptores del tipo de evento
     - Entregar a cada suscriptor en orden
     - Si el suscriptor falla → retry o dead letter

  4. POST-PROCESO:
     - Si todos los suscriptores procesaron → EVENT_PROCESSED
     - Si algún suscriptor falló → EVENT_FAILED (con metadata)
     - Registrar en evidencia
```

---

## Colas

```yaml
queues:
  principal:
    tipo: "FIFO con prioridad"
    prioridades:
      CRITICA: "Eventos de fallo, seguridad, identidad"
      ALTA: "Eventos de Discovery, Indicadores"
      MEDIA: "Eventos de diagnóstico, planificación"
      BAJA: "Heartbeats, métricas, logs"

  delay:
    tipo: "Cola con temporizador"
    uso: "Debounce de eventos (ej: cambios de archivos)"
    default_delay_ms: 2000

  dead_letter:
    tipo: "Eventos que no pudieron entregarse"
    max_retries: 3
    accion_si_agota: "Registrar como no entregado, notificar"
```

---

## Suscripciones

Los suscriptores se registran dinámicamente:

```yaml
suscripciones:
  - suscriptor: "Indicators Engine"
    eventos:
      - "DISCOVERY_UPDATED"
      - "PROJECT_DISCOVERED"
    modo: "síncrono"

  - suscriptor: "DX Engine"
    eventos:
      - "INDICATORS_CALCULATED"
      - "DOCUMENTATION_WARNING"
      - "ARCHITECTURE_CRITICAL"
    modo: "asíncrono"

  - suscriptor: "Planner Engine"
    eventos:
      - "DIAGNOSIS_READY"
      - "INTENT_READY"
    modo: "asíncrono"
```

---

## Métricas del Event Loop

```yaml
event_loop_metrics:
  - "eventos_por_segundo"
  - "cola_actual"
  - "cola_prioritaria_actual"
  - "tasa_entrega_exitosa"
  - "latencia_promedio_despacho_ms"
  - "eventos_fallidos"
  - "dead_letter_actual"
```
