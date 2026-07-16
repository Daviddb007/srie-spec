# EVENTS — Sistema de Eventos de SRIE

---

## ¿Qué es este documento?

EVENTS.md define el **sistema de eventos** de SRIE. Todo en SRIE genera eventos — no solamente archivos. Los eventos permiten que los Engines se comuniquen de forma asíncrona, que el sistema reaccione a cambios y que la trazabilidad sea completa.

Este documento convierte SRIE en un **sistema dirigido por eventos** (event-driven), donde cada acción deja una huella que otros motores pueden consumir.

---

## ¿Qué es un evento?

Un evento es una **notificación estructurada** de que algo ocurrió en el sistema.

```yaml
evento:
  id: string                    # UUID único del evento
  tipo: string                  # Tipo de evento (ver catálogo)
  timestamp: string             # ISO 8601
  origen:
    engine: string              # Engine que generó el evento
    ejecucion_id: string        # ID de la ejecución
  entidad:
    tipo: string                # Tipo de entidad afectada
    id: string                  # ID de la entidad
    version: integer            # Versión de la entidad después del evento
  datos: {}                     # Datos específicos del evento
  severidad: string             # info | warning | error | critical
  relacionados: [string]        # IDs de eventos relacionados
```

---

## Catálogo de eventos

### Eventos de proyecto

| Evento | Emisor | Descripción | Severidad |
|--------|--------|-------------|-----------|
| `PROJECT_DISCOVERED` | INIT | El proyecto fue descubierto por primera vez | info |
| `PROJECT_ANALYZED` | Indicators | El proyecto fue analizado (indicadores calculados) | info |
| `PROJECT_BOOTSTRAPPED` | INIT | El proyecto fue inicializado con SDOS | info |
| `PROJECT_LEVEL_CHANGED` | Governance | El nivel de madurez del proyecto cambió | warning |
| `PROJECT_STATE_CHANGED` | SDOS | El estado operativo del proyecto cambió | info |

### Eventos de Engine

| Evento | Emisor | Descripción | Severidad |
|--------|--------|-------------|-----------|
| `ENGINE_STARTED` | Cualquier Engine | El Engine comenzó su ejecución | info |
| `ENGINE_COMPLETED` | Cualquier Engine | El Engine completó su ejecución exitosamente | info |
| `ENGINE_FAILED` | Cualquier Engine | El Engine falló durante la ejecución | error |
| `ENGINE_TIMEOUT` | Cualquier Engine | El Engine excedió el tiempo máximo | error |
| `ENGINE_SIDE_EFFECT` | Cualquier Engine | El Engine modificó archivos fuera de su dominio | critical |
| `ENGINE_NO_ACTION` | Cualquier Engine | El Engine determinó que no había nada que hacer | info |

### Eventos de Capability

| Evento | Emisor | Descripción | Severidad |
|--------|--------|-------------|-----------|
| `CAPABILITY_CREATED` | Capability Engine | Una nueva capability fue creada | info |
| `CAPABILITY_UPDATED` | Capability Engine | Una capability existente fue actualizada | info |
| `CAPABILITY_DEPRECATED` | Governance | Una capability fue marcada como deprecada | warning |
| `CAPABILITY_ARCHIVED` | Governance | Una capability fue archivada | warning |
| `CAPABILITY_DELETED` | Governance | Una capability fue eliminada | critical |
| `CAPABILITY_VERIFIED` | Sandbox | Una capability pasó la verificación | info |
| `CAPABILITY_REJECTED` | Sandbox | Una capability no pasó la verificación | error |
| `CAPABILITY_DEPLOYED` | Deployment | Una capability fue desplegada | info |
| `CAPABILITY_ROLLED_BACK` | Deployment | Una capability fue revertida | warning |

### Eventos de ciclo SDOS

| Evento | Emisor | Descripción | Severidad |
|--------|--------|-------------|-----------|
| `CYCLE_STARTED` | SDOS | Un nuevo ciclo SDOS comenzó | info |
| `CYCLE_COMPLETED` | SDOS | El ciclo SDOS completó todas sus fases | info |
| `CYCLE_FAILED` | SDOS | El ciclo SDOS falló en alguna fase | error |
| `PHASE_STARTED` | SDOS | Una fase del ciclo comenzó | info |
| `PHASE_COMPLETED` | SDOS | Una fase del ciclo completó | info |
| `PHASE_FAILED` | SDOS | Una fase del ciclo falló | error |

### Eventos de memoria y conocimiento

| Evento | Emisor | Descripción | Severidad |
|--------|--------|-------------|-----------|
| `MEMORY_UPDATED` | Memory Engine | La memoria del proyecto fue actualizada | info |
| `ADR_CREATED` | Memory Engine | Un nuevo ADR fue creado | info |
| `PATTERN_DISCOVERED` | Knowledge Engine | Un nuevo patrón fue identificado | info |
| `ANTI_PATTERN_DISCOVERED` | Knowledge Engine | Un nuevo antipatrón fue identificado | warning |
| `KNOWLEDGE_UPDATED` | Knowledge Engine | El knowledge base fue actualizado | info |

### Eventos de indicadores

| Evento | Emisor | Descripción | Severidad |
|--------|--------|-------------|-----------|
| `SCORE_UPDATED` | Indicators | El SRIE_SCORE fue recalculado | info |
| `DOMAIN_SCORE_CHANGED` | Indicators | La puntuación de un dominio cambió significativamente | warning |
| `HALLUCINATION_RISK_HIGH` | Indicators | El riesgo de alucinación superó el umbral | critical |
| `HEALTH_SCORE_DROPPED` | Indicators | El health score cayó por debajo del umbral | error |

### Eventos de bug y reparación

| Evento | Emisor | Descripción | Severidad |
|--------|--------|-------------|-----------|
| `BUG_FOUND` | Cualquier Engine | Se detectó un bug | error |
| `BUG_FIXED` | Repair Engine | Un bug fue corregido | info |
| `BUG_REGRESSION` | Repair Engine | Un bug previamente corregido reapareció | critical |
| `REPAIR_STARTED` | Repair Engine | El proceso de reparación comenzó | warning |
| `REPAIR_COMPLETED` | Repair Engine | El proceso de reparación completó | info |
| `REPAIR_FAILED` | Repair Engine | El proceso de reparación falló | critical |

### Eventos de deploy

| Evento | Emisor | Descripción | Severidad |
|--------|--------|-------------|-----------|
| `DEPLOY_STARTED` | Deployment | El proceso de deploy comenzó | info |
| `DEPLOY_COMPLETED` | Deployment | El deploy se completó exitosamente | info |
| `DEPLOY_FAILED` | Deployment | El deploy falló | error |
| `ROLLBACK_STARTED` | Deployment | El proceso de rollback comenzó | warning |
| `ROLLBACK_COMPLETED` | Deployment | El rollback se completó exitosamente | info |
| `ROLLBACK_FAILED` | Deployment | El rollback falló | critical |

---

## Formato de almacenamiento

Los eventos se almacenan en `SDOS/events/` con la siguiente estructura:

```
SDOS/events/
├── index.json                 # Índice de todos los eventos (metadatos)
├── 2026-07-15/                # Partición por día
│   ├── 001-project-discovered.json
│   ├── 002-engine-started.json
│   └── ...
└── archive/                   # Eventos antiguos (comprimidos)
```

### Ejemplo de evento almacenado

```json
{
  "id": "evt_abc123def456",
  "tipo": "CAPABILITY_CREATED",
  "timestamp": "2026-07-15T10:30:00Z",
  "origen": {
    "engine": "Capability Engine",
    "ejecucion_id": "exec_789xyz"
  },
  "entidad": {
    "tipo": "capability",
    "id": "calendar",
    "version": 1
  },
  "datos": {
    "nombre": "Calendar",
    "version_inicial": "1.0.0",
    "dependencias": ["auth >=2.0.0"],
    "archivos_creados": 7
  },
  "severidad": "info",
  "relacionados": []
}
```

---

## Consumo de eventos

Los eventos pueden ser consumidos por:

1. **Otros Engines** — Para reaccionar a cambios (ej: cuando CAPABILITY_UPDATED, Sandbox puede re-verificar)
2. **Memory Engine** — Para registrar decisiones y contexto
3. **Indicators Engine** — Para recalcular métricas afectadas
4. **Knowledge Engine** — Para identificar patrones en la secuencia de eventos
5. **Registry** — Para mantener el índice maestro actualizado

### Suscripción

```yaml
suscripcion:
  engine: "Sandbox Engine"
  suscripciones:
    - evento: "CAPABILITY_UPDATED"
      accion: "re-verificar capability afectada"
      condiciones:
        - "la capability está en estado STABLE"
        - "el cambio es en archivos de implementación"

    - evento: "CAPABILITY_CREATED"
      accion: "verificar capability nueva"
      condiciones: []
```

---

## Reglas del sistema de eventos

### Regla 1: Toda acción genera evento

No existe una acción en SRIE que no genere al menos un evento. Si no hay evento, no ocurrió.

### Regla 2: Inmutabilidad

Los eventos no se modifican después de creados. Son inmutables por definición.

### Regla 3: Orden total

Los eventos tienen un orden total dentro del sistema. El campo `id` contiene un contador monótono creciente.

### Regla 4: Almacenamiento durable

Los eventos se persisten en disco antes de ser procesados. Si el sistema falla, los eventos no se pierden.

### Regla 5: Sin eventos = sin estado

Si no hay eventos para un período, el estado del sistema es "sin cambios". La ausencia de eventos es un evento en sí mismo.

### Regla 6: Trazabilidad completa

Dado un evento, debe ser posible reconstruir la cadena completa de eventos que llevaron a él (eventos relacionados).

---

## Eventos vs Evidencia

| Evento | Evidencia |
|--------|-----------|
| Notificación de que algo ocurrió | Registro detallado de lo que ocurrió |
| Liviano (metadata + referencia) | Pesado (contenido completo) |
| Inmutable | Inmutable |
| Consumible por Engines | Consultable por humanos e IA |
| Se almacena en `SDOS/events/` | Se almacena en `SDOS/evidence/` |
| Tiene severidad | No tiene severidad |
| Tiene entidad asociada | Tiene ejecución asociada |

Relación: un evento puede referenciar evidencia, y la evidencia puede referenciar eventos. Ambos coexisten.
