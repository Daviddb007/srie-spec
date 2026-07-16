# INDICATOR PIPELINE — Pipeline de Medición Continua

---

## ¿Qué es este documento?

INDICATOR_PIPELINE.md define **cómo se ejecuta Indicators** en el tiempo. No es un comando único. Es un **pipeline continuo** que se activa por eventos, mantiene el Digital Twin actualizado y genera flujo constante de datos para todos los Engines.

---

## Arquitectura del pipeline

```
┌──────────────────────────────────────────────────────────────────┐
│                      INDICATORS PIPELINE                         │
│                                                                   │
│  EVENT BUS ◄─────── Discovery, cambios externos, schedule         │
│     │                                                             │
│     ▼                                                             │
│  DISPATCHER ──── Determina modo (full / incremental / watch)      │
│     │                                                             │
│     ▼                                                             │
│  TWIN DIFF DETECTOR ──── ¿Qué cambió en el Digital Twin?          │
│     │                                                             │
│     ├── NADA ────► NO_ACTION_NEEDED (exit 10)                     │
│     │                                                             │
│     └── CAMBIOS ──► INDICATOR SELECTOR                            │
│                          │                                         │
│                          ▼                                         │
│                    FORMULA ENGINE ──► Por cada indicador           │
│                          │                                         │
│                          ▼                                         │
│                    TREND CALCULATOR ──► Historial + pendiente      │
│                          │                                         │
│                          ▼                                         │
│                    THRESHOLD DETECTOR ──► ¿Evento?                 │
│                          │                                         │
│                          ▼                                         │
│                    SCORE AGGREGATOR ──► Scores compuestos          │
│                          │                                         │
│                          ▼                                         │
│                    TWIN UPDATER ──► Digital Twin actualizado       │
│                          │                                         │
│                          ▼                                         │
│                    PUBLISHER ──► Artefactos + eventos              │
│                          │                                         │
│                          ▼                                         │
│                    CACHE UPDATER ──► Historial de mediciones       │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

---

## Componentes del pipeline

### 1. Dispatcher

```yaml
dispatcher:
  reglas:
    - evento: "PROJECT_DISCOVERED"
      modo: "full"
      prioridad: "alta"

    - evento: "DISCOVERY_UPDATED"
      modo: "incremental"
      prioridad: "media"

    - evento: "ENGINE_COMPLETED"
      modo: "incremental"
      prioridad: "baja"
      condicion: "solo si el Engine modificó el Twin"

    - evento: "SCHEDULED"
      modo: "incremental"
      prioridad: "baja"
      intervalo: "cada 6 horas"

    - evento: "MANUAL_REQUEST"
      modo: "segun parametro"
      prioridad: "alta"
```

### 2. Twin Diff Detector

Compara el Digital Twin actual con el de la última ejecución de Indicators para determinar qué cambió.

```yaml
twin_diff:
  entrada: "Digital Twin actual vs cached_twin.json"
  proceso:
    - "Comparar PROJECT_STATE.md (estructura)"
    - "Comparar GRAPH.md (nodos y relaciones)"
    - "Comparar REGISTRY.yaml (entidades)"
    - "Comparar IDENTITY.yaml (identidad)"
  salida:
    hay_cambios: boolean
    nodos_modificados: [string]
    nodos_creados: [string]
    nodos_eliminados: [string]
    entidades_afectadas: [string]
```

### 3. Indicator Selector

Selecciona qué indicadores recalcular según los cambios detectados.

```yaml
indicator_selector:
  full: "Todos los indicadores del catálogo"
  incremental:
    logica: |
      - Si cambió PROJECT_STATE.md → recalcular todos los de observación
      - Si cambió GRAPH.md → recalcular los de salud y riesgo estructural
      - Si cambiaron indicadores de observación → recalcular los de salud
      - Si cambiaron indicadores de salud → recalcular los de riesgo
      - Si cambiaron indicadores de riesgo → recalcular los predictivos
    dependencias:
      observacion: []
      salud: ["observacion"]
      riesgo: ["salud", "observacion"]
      predictivo: ["riesgo", "salud"]
```

### 4. Formula Engine

Evalúa la fórmula de cada indicador contra el Digital Twin actual.

```yaml
formula_engine:
  entrada: "Indicador + Digital Twin"
  proceso:
    - "Parsear fórmula del catálogo"
    - "Resolver referencias a otros indicadores (buscar en caché o calcular)"
    - "Resolver referencias al Twin (consultar Twin actual)"
    - "Ejecutar fórmula"
    - "Aplicar transformaciones (inversión, normalización)"
  salida:
    valor: float
    estado: "CALCULATED | ERROR | INSUFFICIENT_DATA"
```

### 5. Trend Calculator

Calcula la tendencia del indicador a partir de su historial.

```yaml
trend_calculator:
  entrada: "Indicador ID + nuevo valor"
  proceso:
    - "Cargar historial de SDOS/evidence/indicators/trends/<id>.json"
    - "Agregar nuevo valor al historial"
    - "Ejecutar regresión lineal simple (últimas N mediciones)"
    - "Clasificar tendencia"
  salida:
    tendencia: "IMPROVING | STABLE | DECLINING | VOLATILE"
    pendiente: float
    historial_actualizado: [float]
```

### 6. Threshold Detector

Compara el valor contra los thresholds del indicador y emite eventos.

```yaml
threshold_detector:
  entrada: "Indicador (valor + threshold)"
  proceso:
    - "Comparar valor contra threshold.warning"
    - "Comparar valor contra threshold.critical"
    - "Comparar tendencia contra eventos_asociados"
  eventos_emitidos:
    - condicion: "valor < threshold.warning AND valor >= threshold.critical"
      evento: "<INDICADOR>_WARNING"
    - condicion: "valor < threshold.critical"
      evento: "<INDICADOR>_CRITICAL"
    - condicion: "tendencia == DECLINING AND valor < threshold.warning"
      evento: "<INDICADOR>_DEGRADING"
    - condicion: "tendencia == IMPROVING AND valor >= threshold.warning"
      evento: "<INDICADOR>_IMPROVING"
```

### 7. Score Aggregator

Calcula los scores compuestos a partir de los indicadores ya medidos.

```yaml
score_aggregator:
  entrada: "Todos los indicadores calculados en esta ejecución"
  proceso:
    - "Cargar definición de scores (del modelo)"
    - "Para cada score: aplicar fórmula con pesos"
    - "Si algún componente no se calculó, usar último valor conocido"
    - "Si ningún componente se calculó, score = null"
  salida:
    scores:
      engineering_score: 78.3
      organizational_score: 82.1
      knowledge_score: 65.4
      evolution_score: 71.2
      global_srie_score: 74.8
```

### 8. Twin Updater

Actualiza el Digital Twin con los nuevos valores de indicadores.

```yaml
twin_updater:
  entrada: "Digital Twin + nuevos indicadores"
  proceso:
    - "Agregar indicadores al Twin"
    - "Actualizar SRIE_SCORE en SDOS/STATE.md"
    - "Actualizar nivel de madurez si cambió"
    - "Incrementar versión del Twin"
  salida: "Digital Twin actualizado"
```

### 9. Publisher

Genera los artefactos de salida y emite eventos.

```yaml
publisher:
  artefactos:
    - path: "SDOS/SRIE_REPORT.md"
      contenido: "Reporte completo de todos los indicadores y scores"
    - path: "SDOS/evidence/indicators/<fecha>/<id>.json"
      contenido: "Un archivo por indicador medido"
    - path: "SDOS/evidence/indicators/trends/<id>.json"
      contenido: "Historial de cada indicador"
    - path: "SDOS/SRIE_DIGITAL_TWIN.json"
      contenido: "Digital Twin actualizado"

  eventos:
    - "INDICATORS_CALCULATED"       # Después de cada ejecución exitosa
    - "INDICATORS_PARTIAL"          # Si algunos indicadores fallaron
    - "<INDICADOR>_WARNING"         # Por cada threshold cruzado
    - "<INDICADOR>_CRITICAL"
    - "<INDICADOR>_DEGRADING"
    - "<INDICADOR>_IMPROVING"
```

---

## Integración con el ciclo SDOS

```yaml
integracion_sdos:
  despues_de_discovery:
    - "Ejecutar Indicators completo"
    - "Si SRIE_SCORE bajó de nivel → notificar a Governance Engine"

  durante_dx:
    - "DX Engine consulta SRIE_REPORT.md para priorizar mejoras"
    - "Tendencias DECLINING son entradas prioritarias para DX"

  durante_plan:
    - "Planner Engine usa indicadores para priorizar capacidades"
    - "Indicadores CRITICAL generan tareas automáticas"

  durante_build:
    - "Capability Engine consulta indicadores del dominio afectado"
    - "No puede construir sobre un dominio con salud CRITICAL sin autorización"

  durante_learn:
    - "Knowledge Engine consolida tendencias como patrones"
    - "Si un indicador mejoró consistentemente, registrar como patrón"
    - "Si un indicador empeoró consistentemente, registrar como antipatrón"
```

---

## Caché y persistencia

```yaml
cache:
  archivo: "SDOS/evidence/indicators/INDICATORS_CACHE.yaml"
  contenido:
    ultima_ejecucion: "2026-07-15T10:00:00Z"
    modo: "full"
    indicadores_calculados: 25
    indicadores_fallidos: 0
    scores: {}
    twin_version: 12

historial:
  ubicacion: "SDOS/evidence/indicators/trends/"
  formato: "Un archivo por indicador, lista de valores con timestamps"
  retencion: "Últimos 365 días"
  poda: "Mediciones > 1 año se archivan en SDOS/evidence/indicators/archive/"
```
