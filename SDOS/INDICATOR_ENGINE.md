# INDICATOR ENGINE — Especificación del Motor de Indicadores

---

## ¿Qué es el Indicators Engine?

El Indicators Engine es el **sistema nervioso** de SRIE. No calcula números: mide el estado del Digital Twin, detecta tendencias, emite eventos y alimenta de información a todos los demás Engines.

Es el tercer Engine del ciclo SDOS, después de Discovery:

```
BOOT → DISCOVERY → INDICATORS → DX → PLAN → BUILD → VERIFY → DEPLOY → LEARN
```

---

## Rol en el ciclo SDOS

```yaml
posicion_en_ciclo: 3
despues_de: "Discovery Engine"
antes_de: "DX Engine"

ejecucion:
  - "Completa: después de Discovery completo"
  - "Incremental: cuando Discovery emite DISCOVERY_UPDATED"
  - "Continua: modo watch, reacciona a eventos del sistema"
  - "Bajo demanda: srie indicators"
```

---

## Contrato del Engine

```yaml
engine:
  nombre: "Indicators Engine"
  version: "1.0.0"
  tipo_engine: indicators

  contrato:
    input:
      archivos_requeridos:
        - path: "SDOS/INDICATOR_CATALOG.yaml"
          opcional: false
          descripcion: "Catálogo declarativo de indicadores"
        - path: "SDOS/PROJECT_STATE.md"
          opcional: false
        - path: "SDOS/GRAPH.md"
          opcional: false
        - path: "SDOS/REGISTRY.yaml"
          opcional: false
      parametros:
        - nombre: modo
          tipo: string
          requerido: false
          defecto: "full"
          valores: ["full", "incremental", "watch"]
        - nombre: dominio
          tipo: string
          requerido: false
          descripcion: "Calcular solo indicadores de un dominio específico"
        - nombre: entidad
          tipo: string
          requerido: false
          descripcion: "Calcular solo indicadores de una entidad específica"

    output:
      archivos_generados:
        - path: "SDOS/SRIE_REPORT.md"
          condicion: "siempre"
        - path: "SDOS/evidence/indicators/<fecha>/<indicador_id>.json"
          condicion: "por cada indicador medido"
        - path: "SDOS/evidence/indicators/trends/<score_id>.json"
          condicion: "por cada score calculado"
        - path: "SDOS/SRIE_DIGITAL_TWIN.json"
          condicion: "siempre (actualización del Twin)"
      estado_salida:
        exito: "INDICATORS_CALCULATED"
        parcial: "INDICATORS_PARTIAL"
        error: "INDICATORS_FAILED"

    archivos:
      puede_modificar:
        - "SDOS/SRIE_REPORT.md"
        - "SDOS/evidence/indicators/"
        - "SDOS/SRIE_DIGITAL_TWIN.json"
        - "SDOS/REGISTRY.yaml"    # Actualizar métricas en entidades
      prohibido_modificar:
        - "Archivos del proyecto fuente"
        - "SDOS/GRAPH.md"         # Solo Discovery modifica el grafo
        - "SDOS/PROJECT_STATE.md"

    reglas:
      - "Todos los indicadores se calculan desde el Digital Twin, no del proyecto físico"
      - "Cada indicador debe incluir confidence_score"
      - "Cada indicador debe calcular su tendencia (si hay historial suficiente)"
      - "Si un indicador cruza un threshold, debe emitir el evento correspondiente"
      - "Los indicadores de nivel predictivo requieren mínimo 10 mediciones históricas"
      - "No modifica archivos fuera de SDOS/evidence/indicators/"

    dependencias:
      engines: ["Discovery Engine"]
      archivos:
        - "SDOS/INDICATOR_CATALOG.md"
        - "SDOS/INDICATOR_MODEL.md"
      sistema: []

    indicadores_propios:
      - nombre: "indicators_coverage"
        descripcion: "Porcentaje de indicadores del catálogo que pudieron calcularse"
      - nombre: "indicators_confidence_avg"
        descripcion: "Confianza promedio de todos los indicadores calculados"

    tiempo_esperado_ms: 15000
    nivel_srie_requerido: "L0"
```

---

## Arquitectura interna

```
┌───────────────────────────────────────────────────────────────┐
│                     INDICATORS ENGINE                         │
│                                                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐    │
│  │ CATALOG      │  │ TWIN READER  │  │ FORMULA ENGINE   │    │
│  │ LOADER       │──► (lee Digital │──► (evalúa fórmula   │    │
│  │ (INDICATOR_  │  │  Twin)       │  │  de cada indic.) │    │
│  │ CATALOG.yaml)│  └──────────────┘  └────────┬─────────┘    │
│  └──────────────┘                              │              │
│                                                ▼              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐    │
│  │ TREND        │  │ CONFIDENCE   │  │ THRESHOLD        │    │
│  │ CALCULATOR   │◄─┤ CALCULATOR   │◄─┤ DETECTOR         │    │
│  │ (historial)  │  │ (evidencias) │  │ (eventos)        │    │
│  └──────┬───────┘  └──────┬───────┘  └────────┬─────────┘    │
│         │                 │                    │              │
│         └─────────────────┼────────────────────┘              │
│                           ▼                                   │
│                 ┌──────────────────┐                         │
│                 │   PUBLISHER      │                         │
│                 │ (artefactos +     │                         │
│                 │  eventos + twin) │                          │
│                 └──────────────────┘                         │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

### Componentes

1. **Catalog Loader** — Carga el catálogo de indicadores desde `INDICATOR_CATALOG.yaml`
2. **Twin Reader** — Lee el Digital Twin actual
3. **Formula Engine** — Evalúa la fórmula de cada indicador contra el Twin
4. **Confidence Calculator** — Calcula la confianza de cada medición
5. **Trend Calculator** — Calcula tendencia desde el historial de mediciones
6. **Threshold Detector** — Compara valor contra thresholds y emite eventos
7. **Publisher** — Genera artefactos, actualiza el Twin, emite eventos

---

## Modos de ejecución

### FULL

Calcula **todos** los indicadores del catálogo desde cero. Recorre cada nivel (observación → salud → riesgo → predictivo) en orden, porque los niveles superiores dependen de los inferiores.

```yaml
full:
  orden:
    1. "Calcular indicadores de OBSERVACIÓN (IND-001 a IND-013)"
    2. "Calcular indicadores de SALUD (IND-020 a IND-025)"
    3. "Calcular indicadores de RIESGO (IND-030 a IND-034)"
    4. "Calcular indicadores PREDICTIVOS (IND-040 a IND-043)"
    5. "Calcular scores (engineering, organizational, knowledge, evolution)"
    6. "Calcular Global SRIE Score"
    7. "Actualizar Digital Twin"
    8. "Publicar todos los artefactos"
```

### INCREMENTAL

Solo recalcula indicadores afectados por cambios en el Twin.

```yaml
incremental:
  disparador: "Evento DISCOVERY_UPDATED"
  proceso:
    1. "Identificar qué nodos del Twin cambiaron"
    2. "Seleccionar solo indicadores que dependen de esos nodos"
    3. "Recalcular en orden (observación → salud → riesgo → predictivo)"
    4. "Actualizar tendencias de los indicadores recalculados"
    5. "Actualizar scores"
    6. "Actualizar Twin"
    7. "Publicar solo artefactos afectados"
```

### WATCH (servicio continuo)

```yaml
watch:
  disparador: "Cualquier cambio en el Digital Twin"
  proceso:
    1. "Esperar 1s (debounce)"
    2. "Ejecutar incremental"
    3. "Si algún threshold se cruza, emitir evento inmediatamente"
```

---

## Códigos de salida

| Código | Estado | Significado |
|--------|--------|-------------|
| 0 | `INDICATORS_CALCULATED` | Todos los indicadores calculados exitosamente |
| 1 | `INDICATORS_PARTIAL` | Algunos indicadores no pudieron calcularse (falta de datos) |
| 2 | `INDICATORS_FAILED` | Error en el cálculo |
| 10 | `NO_ACTION_NEEDED` | Incremental: sin cambios desde la última medición |
