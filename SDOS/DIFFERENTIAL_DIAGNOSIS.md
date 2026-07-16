# DIFFERENTIAL DIAGNOSIS — Diagnóstico Diferencial Basado en Evidencia

---

## ¿Qué es este documento?

DIFFERENTIAL_DIAGNOSIS.md define el **proceso de diagnóstico diferencial** de SRIE. Tomado de la medicina, el diagnóstico diferencial no ofrece una única respuesta. Presenta **múltiples explicaciones compatibles con la evidencia**, cada una con su nivel de confianza, y selecciona la más probable sin descartar las alternativas.

Un diagnóstico diferencial nunca dice:

> "Este es el problema."

Siempre dice:

> "Estas son las explicaciones compatibles con la evidencia, ordenadas por probabilidad."

---

## El proceso de diagnóstico diferencial

```
HYPOTHESIS ENGINE (hipótesis validadas)
    │
    ▼
EVIDENCE MATCHER (buscar evidencia para cada hipótesis)
    │
    ▼
CONFIDENCE CALCULATOR (calcular confianza de cada hipótesis)
    │
    ▼
DIFFERENTIAL ANALYZER (comparar, refutar, seleccionar)
    │
    ├── Hipótesis A: 72%
    ├── Hipótesis B: 18%
    ├── Hipótesis C: 7%
    └── Hipótesis D: 3%
    │
    ▼
DIAGNOSIS (diagnóstico principal + alternativas)
    │
    ▼
PUBLISHER
```

---

## 1. Evidence Matcher

Para cada hipótesis validada, busca toda la evidencia disponible que la respalda o la contradice.

```yaml
evidence_matcher:
  entrada: "Hipótesis validadas"
  proceso:
    por_cada_hipotesis:
      1. "Buscar en Digital Twin evidencia que respalde la hipótesis"
      2. "Buscar en Digital Twin evidencia que contradiga la hipótesis"
      3. "Buscar en DIAGNOSIS_PATTERNS.md patrones coincidentes"
      4. "Buscar en CASE_REGISTRY.md casos similares"
      5. "Calcular match_score (0.0 - 1.0) para cada fuente"

  salida:
    hipotesis: "HYP-001"
    evidencia_favorable:
      - fuente: "documentation_health"
        valor: 68.2
        tendencia: "DECLINING"
        peso: 0.30
        match: 0.95
      - fuente: "architecture_exists"
        valor: false
        peso: 0.25
        match: 1.00
    evidencia_contradictoria:
      - fuente: "recent_commits"
        valor: "docs/"
        peso: 0.20
        match: 0.60
        nota: "Hay commits recientes en docs/, pero no en ARCHITECTURE.md"
    evidencia_patrones:
      - patron: "DOCUMENTATION_DECAY"
        match: 0.85
    evidencia_casos:
      - caso: "CASE-003"
        similitud: 0.78
```

---

## 2. Confidence Calculator

Calcula la confianza de cada hipótesis a partir de toda la evidencia recolectada.

```yaml
confidence_calculator:
  formula: |
    Confianza_hipotesis = (promedio_matches_evidencia × 0.35)
                        + (match_patrones × 0.25)
                        + (match_casos × 0.20)
                        - (penalizacion_contradicciones × 0.20)

  donde:
    promedio_matches_evidencia: "Promedio de match de todas las fuentes de evidencia"
    match_patrones: "Match del mejor patrón encontrado (0 si no hay)"
    match_casos: "Similitud del mejor caso encontrado (0 si no hay)"
    penalizacion_contradicciones: |
      Cantidad de evidencia contradictoria normalizada
      (0 contradicciones = 0, 1 = 0.2, 2+ = 0.5)

  reglas:
    - "Si hay evidencia contradictoria con match > 0.8, la hipótesis se descarta automáticamente"
    - "Si no hay evidencia favorable (match_promedio < 0.3), la hipótesis se descarta"
```

---

## 3. Differential Analyzer

Compara todas las hipótesis, las ordena por confianza, identifica la mejor explicación y documenta las alternativas descartadas.

```yaml
differential_analyzer:
  entrada: "Todas las hipótesis con su confianza calculada"

  proceso:
    1. "Ordenar hipótesis por confianza descendente"
    2. "Seleccionar la de mayor confianza como diagnóstico PRINCIPAL"
    3. "Las siguientes como ALTERNATIVAS"
    4. "Si confianza_principal - confianza_alternativa < 0.10:
        marcar diagnóstico como INCIERTO (necesita más evidencia)"
    5. "Si confianza_principal < 0.50:
        marcar diagnóstico como ESPECULATIVO"
    6. "Documentar por qué se descartó cada alternativa"

  salida:
    diagnostico:
      principal:
        hipotesis: "HYP-001"
        confianza: 0.72
        clasificacion: "MODERADA"
      alternativas:
        - hipotesis: "HYP-002"
          confianza: 0.45
          razon_descarte: "No se encontró evidencia de falta de memoria reciente"
        - hipotesis: "HYP-003"
          confianza: 0.31
          razon_descarte: "La deuda técnica no muestra correlación con falta de refactorización"
      estado: "CONFIRMADO"
```

---

## Formato del diagnóstico

```yaml
diagnostico:
  id: "DX-001"
  timestamp: "2026-07-15T10:00:00Z"

  resumen: |
    El proyecto muestra deterioro en su documentación y gobernanza.
    El indicador documentation_health ha caído de 82 a 68 en 30 días,
    ARCHITECTURE.md no existe y la memoria no se actualiza.
    La explicación más probable es que el equipo está priorizando
    features sobre documentación (DOCUMENTATION_DECAY).

  nivel: "salud"
  dominio: "gobernanza"

  principal:
    hipotesis: "HYP-001"
    titulo: "La documentación del proyecto se está deteriorando"
    confianza: 0.72
    clasificacion: "MODERADA"

  alternativas:
    - hipotesis: "HYP-002"
      titulo: "Riesgo de alucinación por falta de memoria"
      confianza: 0.45
      razon_descarte: "La memoria existe y tiene ADR recientes. No hay evidencia de alucinación."

    - hipotesis: "HYP-004"
      titulo: "El equipo está en fase de entrega y pospuso documentación"
      confianza: 0.38
      razon_descarte: "No hay evidencia de fechas de entrega próximas en el roadmap."

  evidencia:
    fuentes_favorables: 4
    fuentes_contradictorias: 1
    patrones_coincidentes: 1
    casos_similares: 1

  intervencion_sugerida:
    acciones:
      - "Crear ARCHITECTURE.md (crítico)"
      - "Actualizar memory/index.md (alto)"
      - "Agregar documentación al definition of done del equipo (medio)"
    impacto_esperado:
      documentation_health: "+15 puntos en 30 días"
    riesgo_intervencion: "BAJO"

  metadata:
    engine: "DX Engine (Differential Diagnosis)"
    ejecucion_id: "dx_001"
    hipotesis_evaluadas: 4
    tiempo_diagnostico_ms: 1200
```

---

## Clasificación de diagnósticos

| Clasificación | Rango de confianza | Significado | Acción |
|---------------|-------------------|-------------|--------|
| CONFIRMADO | ≥ 0.85 | Alta certeza | Actuar inmediatamente |
| MODERADA | 0.60 — 0.84 | Certeza suficiente | Actuar con seguimiento |
| INCIERTO | 0.40 — 0.59 | Múltiples explicaciones posibles | Buscar más evidencia |
| ESPECULATIVO | < 0.40 | Sin suficiente evidencia | No actuar. Marcar para revisión. |

---

## Registro del diagnóstico

Cada diagnóstico se registra en `SDOS/evidence/dx/<fecha>/<dx_id>.json`:

```json
{
  "diagnostico_id": "DX-001",
  "resumen": "...",
  "principal": { "hipotesis": "HYP-001", "confianza": 0.72 },
  "alternativas": [{"hipotesis": "HYP-002", "confianza": 0.45, "razon_descarte": "..."}],
  "intervencion_sugerida": { "acciones": [...], "impacto_esperado": {...} },
  "estado": "CONFIRMADO",
  "timestamp": "2026-07-15T10:00:00Z"
}
```

Además, el diagnóstico se incorpora al Case Registry como un nuevo caso (ver CASE_REGISTRY.md).
