# INDICATOR REPORTS — Artefactos del Sistema de Indicadores

---

## ¿Qué es este documento?

INDICATOR_REPORTS.md define **todos los artefactos** que genera el Indicators Engine. No son solo reportes. Son la **expresión del Digital Twin** en formatos consumibles por humanos y por otros Engines.

---

## Catálogo de artefactos

| # | Artefacto | Ruta | Propósito | Consumido por |
|---|-----------|------|-----------|---------------|
| 1 | SRIE_REPORT.md | `SDOS/SRIE_REPORT.md` | Reporte completo de indicadores | Humanos, DX, Planner |
| 2 | DIGITAL_TWIN.json | `SDOS/SRIE_DIGITAL_TWIN.json` | Twin completo en JSON | Todos los Engines |
| 3 | Indicator Evidence | `SDOS/evidence/indicators/<fecha>/<id>.json` | Medición individual | Knowledge, Memory |
| 4 | Trend History | `SDOS/evidence/indicators/trends/<id>.json` | Historial del indicador | Knowledge, Predictivo |
| 5 | INDICATORS_CACHE.yaml | `SDOS/evidence/indicators/INDICATORS_CACHE.yaml` | Estado de última ejecución | Indicators Pipeline |

---

## 1. SRIE_REPORT.md

**Ruta:** `SDOS/SRIE_REPORT.md`
**Propósito:** Reporte completo y legible del estado de todos los indicadores y scores.

### Formato

```markdown
# SRIE Report — 2026-07-15

**Global SRIE Score:** 74.8 / 100
**Nivel de Madurez:** L3
**Tendencia Global:** STABLE

---

## Scores

| Score | Valor | Tendencia | Threshold |
|-------|-------|-----------|-----------|
| Engineering Score | 78.3 | STABLE | — |
| Organizational Score | 82.1 | DECLINING ⚠️ | 70 |
| Knowledge Score | 65.4 | IMPROVING ↑ | — |
| Evolution Score | 71.2 | STABLE | — |

---

## Indicadores por Nivel

### Observación (13/13 calculados)

| Indicador | Valor | Confianza | Estado |
|-----------|-------|-----------|--------|
| readme_exists | ✅ | 0.99 | OK |
| architecture_exists | ❌ | 1.00 | ⚠️ CRITICAL |
| changelog_exists | ✅ | 0.99 | OK |
| tests_detected | ✅ | 0.95 | OK |
| memory_exists | ✅ | 0.99 | OK |

### Salud (6/6 calculados)

| Indicador | Valor | Tendencia | Estado |
|-----------|-------|-----------|--------|
| documentation_health | 68.2 | DECLINING ↓ | ⚠️ WARNING |
| architecture_health | 82.5 | STABLE | OK |
| security_health | 91.0 | IMPROVING ↑ | OK |
| deployment_health | 75.0 | STABLE | OK |
| knowledge_health | 58.0 | DECLINING ↓ | ⚠️ WARNING |
| federation_health | 85.0 | STABLE | OK |

### Riesgo (5/5 calculados)

| Indicador | Valor | Clasificación | Tendencia |
|-----------|-------|---------------|-----------|
| maintenance_risk | 0.35 | MEDIO | STABLE |
| deployment_risk | 0.20 | BAJO | STABLE |
| hallucination_risk | 0.08 | MUY BAJO | IMPROVING |
| tech_debt_risk | 0.55 | ALTO | DECLINING ↑ |
| fragmentation_risk | 0.12 | BAJO | STABLE |

### Predictivo (4/4 calculados)

| Indicador | Probabilidad | Confianza |
|-----------|--------------|-----------|
| production_failure_probability | 12% | 0.65 |
| tech_debt_30d_probability | 64% | 0.55 |
| domain_conflict_probability | 4% | 0.70 |
| hallucination_30d_probability | 2% | 0.75 |

---

## Eventos Activos

| Evento | Desde | Severidad |
|--------|-------|-----------|
| ARCHITECTURE_MISSING | 2026-07-10 | CRITICAL |
| DOCUMENTATION_WARNING | 2026-07-12 | WARNING |
| KNOWLEDGE_WARNING | 2026-07-14 | WARNING |
| TECH_DEBT_HIGH | 2026-07-08 | WARNING |

---

## Acciones Sugeridas

1. **Crear ARCHITECTURE.md** — Impacto crítico en gobernanza
2. **Actualizar memory/index.md** — La documentación health está en warning
3. **Revisar deuda técnica** — Tendencia creciente en tech_debt_risk
4. **Agregar patrones a knowledge/** — Knowledge health declining
```

---

## 2. SRIE_DIGITAL_TWIN.json

**Ruta:** `SDOS/SRIE_DIGITAL_TWIN.json`
**Propósito:** Representación completa del estado del proyecto en formato JSON, consumible por todos los Engines.

### Formato

```json
{
  "twin_version": 12,
  "ultima_actualizacion": "2026-07-15T10:00:00Z",
  "proyecto": {
    "id": "PROJECT-001",
    "nombre": "mi-proyecto",
    "estado": "ACTIVE",
    "nivel_madurez": "L3",
    "srie_score": 74.8
  },
  "identidad": {
    "domain_id": "DOM-001",
    "tipo": "standalone",
    "cadena_valida": true
  },
  "estructura": {
    "lenguajes": ["Python 3.12"],
    "frameworks": ["FastAPI"],
    "archivos_totales": 128,
    "capabilities": ["calendar"]
  },
  "indicadores": {
    "engineering_score": 78.3,
    "organizational_score": 82.1,
    "knowledge_score": 65.4,
    "evolution_score": 71.2,
    "global_score": 74.8,
    "ultima_medicion": "2026-07-15T10:00:00Z",
    "indicadores_detalle": {
      "IND-020": {
        "valor": 68.2,
        "confianza": 0.92,
        "tendencia": "DECLINING"
      }
    }
  },
  "scores_historicos": {
    "engineering_score": [75.0, 76.2, 77.1, 78.3],
    "global_score": [72.0, 73.1, 74.0, 74.8]
  },
  "eventos_activos": [
    {
      "tipo": "DOCUMENTATION_WARNING",
      "desde": "2026-07-12",
      "severidad": "warning"
    }
  ],
  "metadata": {
    "engine": "Indicators Engine",
    "ejecucion_id": "exec_ind_015",
    "modo": "full"
  }
}
```

---

## 3. Indicator Evidence

**Ruta:** `SDOS/evidence/indicators/<fecha>/<indicador_id>.json`
**Propósito:** Medición individual de un indicador, con toda su metadata.

### Formato

(Ver INDICATOR_MODEL.md → "La estructura de un indicador")

---

## 4. Trend History

**Ruta:** `SDOS/evidence/indicators/trends/<indicador_id>.json`
**Propósito:** Historial completo de mediciones de un indicador para análisis de tendencias.

### Formato

```json
{
  "indicador_id": "IND-020",
  "nombre": "documentation_health",
  "mediciones": [
    {"fecha": "2026-06-15", "valor": 82.0, "confianza": 0.90},
    {"fecha": "2026-06-22", "valor": 80.0, "confianza": 0.91},
    {"fecha": "2026-06-29", "valor": 78.0, "confianza": 0.90},
    {"fecha": "2026-07-06", "valor": 75.0, "confianza": 0.91},
    {"fecha": "2026-07-13", "valor": 73.5, "confianza": 0.92},
    {"fecha": "2026-07-15", "valor": 68.2, "confianza": 0.92}
  ],
  "tendencia_actual": {
    "direccion": "DECLINING",
    "pendiente": -2.3,
    "periodo_dias": 30
  },
  "eventos_asociados": [
    {"fecha": "2026-07-12", "evento": "DOCUMENTATION_WARNING"},
    {"fecha": "2026-07-15", "evento": "DOCUMENTATION_DEGRADING"}
  ]
}
```

---

## 5. INDICATORS_CACHE.yaml

**Ruta:** `SDOS/evidence/indicators/INDICATORS_CACHE.yaml`
**Propósito:** Estado de la última ejecución del pipeline.

### Formato

```yaml
# Indicators Cache
ultima_ejecucion: "2026-07-15T10:00:00Z"
modo: "full"
twin_version: 12
indicadores_calculados: 25
indicadores_fallidos: 0
indicadores_parciales:
  - id: "IND-041"
    razon: "historial_insuficiente_para_predictivo"
scores:
  engineering_score: 78.3
  organizational_score: 82.1
  knowledge_score: 65.4
  evolution_score: 71.2
  global_score: 74.8
eventos_emitidos: 4
ultimo_evento: "DOCUMENTATION_DEGRADING"
```

---

## Resumen de artefactos por modo

| Artefacto | FULL | INCREMENTAL | WATCH |
|-----------|------|-------------|-------|
| SRIE_REPORT.md | ✓ Generar completo | ✓ Solo secciones afectadas | Solo si cambios |
| DIGITAL_TWIN.json | ✓ Generar completo | ✓ Actualizar | ✓ Actualizar |
| Indicator Evidence | ✓ Todos | ✓ Solo recalculados | ✓ Solo recalculados |
| Trend History | ✓ Actualizar | ✓ Actualizar | ✓ Actualizar |
| INDICATORS_CACHE.yaml | ✓ Actualizar | ✓ Actualizar | ✓ Actualizar |
