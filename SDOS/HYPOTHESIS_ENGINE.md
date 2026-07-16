# HYPOTHESIS ENGINE — Generación de Hipótesis desde Indicadores

---

## ¿Qué es este documento?

HYPOTHESIS_ENGINE.md define el motor de **generación de hipótesis** de SRIE. No es un Engine independiente del ciclo SDOS. Es el **primer submotor del DX Engine** — el paso que transforma indicadores en hipótesis explicativas antes de cualquier diagnóstico.

Un médico no ve síntomas y diagnostica. Primero formula hipótesis. SRIE hace lo mismo.

---

## El flujo completo de razonamiento

```
INDICATORS (síntomas)
    │
    ▼
HYPOTHESIS ENGINE (hipótesis)
    │
    ▼
EVIDENCE MATCHER (validación)
    │
    ├── Hipótesis A: 72% (confirmada)
    ├── Hipótesis B: 18% (posible)
    ├── Hipótesis C: 7%  (improbable)
    └── Hipótesis D: 3%  (descartada)
    │
    ▼
DIFFERENTIAL DIAGNOSIS (diagnóstico)
    │
    ▼
DIAGNOSIS PUBLISHER (publicación)
```

---

## ¿Qué es una hipótesis?

Una hipótesis es una **explicación plausible** del estado actual del sistema basada en los indicadores disponibles. No es un diagnóstico. Es una **candidata a diagnóstico** que debe ser validada.

```yaml
hipotesis:
  id: "HYP-001"
  titulo: "La documentación del proyecto se está deteriorando"
  nivel: "salud"                    # observacion | salud | riesgo
  dominio: "gobernanza"

  sintomas:
    - indicador: "documentation_health"
      valor: 68.2
      tendencia: "DECLINING"
    - indicador: "architecture_exists"
      valor: false
    - indicador: "memory_freshness"
      valor: 45

  explicacion: |
    El indicador documentation_health ha caído de 82 a 68 en 30 días.
    ARCHITECTURE.md no existe. La memoria no se ha actualizado en 45 días.
    Es probable que el equipo esté priorizando features sobre documentación.

  evidencia_actual:
    fuentes: 4
    confianza_parcial: 0.72

  patrones_relacionados:
    - "DOCUMENTATION_DECAY"
    - "FEATURE_OVER_DOCS"

  casos_relacionados:
    - "CASE-003"
```

---

## Generación de hipótesis

El Hypothesis Engine recibe los indicadores del Indicators Engine y genera hipótesis en cuatro niveles:

### Nivel 1: Hipótesis directas

Una hipótesis directa se genera cuando un indicador cruza un threshold conocido.

```yaml
generacion_directa:
  regla: "Si un indicador cruza un threshold, generar hipótesis directa"
  ejemplo:
    indicador: "documentation_health"
    valor: 68.2
    threshold_warning: 70
    hipotesis_generada:
      - "La documentación del proyecto está por debajo del nivel saludable"
      - "La falta de documentación afecta la gobernanza del proyecto"
```

### Nivel 2: Hipótesis compuestas

Una hipótesis compuesta se genera cuando múltiples indicadores relacionados muestran un patrón.

```yaml
generacion_compuesta:
  regla: "Si 2+ indicadores del mismo dominio muestran tendencia negativa, generar hipótesis compuesta"
  ejemplo:
    indicadores: ["documentation_health", "architecture_health", "memory_freshness"]
    dominio: "gobernanza"
    hipotesis_generada:
      - "El proyecto está acumulando deuda de documentación en múltiples dimensiones"
      - "La gobernanza del proyecto se está deteriorando de forma generalizada"
```

### Nivel 3: Hipótesis por patrón

Una hipótesis por patrón se genera cuando los indicadores coinciden con un patrón de diagnóstico conocido.

```yaml
generacion_por_patron:
  regla: "Si los indicadores coinciden con un patrón en DIAGNOSIS_PATTERNS.md, generar hipótesis"
  fuente: "knowledge/diagnosis/<dominio>/<patron>.md"
  ejemplo:
    patron: "DOCUMENTATION_DECAY"
    match: 0.85
    hipotesis_generada:
      - "Patrón DOCUMENTATION_DECAY detectado con 85% de coincidencia"
      - "Este patrón indica que el equipo prioriza features sobre documentación"
```

### Nivel 4: Hipótesis por caso

Una hipótesis por caso se genera cuando los indicadores son similares a un caso previo en el Case Registry.

```yaml
generacion_por_caso:
  regla: "Si los indicadores tienen alta similitud con un caso previo, generar hipótesis"
  fuente: "knowledge/cases/<id>.md"
  ejemplo:
    caso: "CASE-003"
    similitud: 0.78
    hipotesis_generada:
      - "Caso similar CASE-003 detectado con 78% de similitud"
      - "En aquel caso, la causa fue falta de recursos en el equipo"
```

---

## Validación de hipótesis

Cada hipótesis generada debe ser validada antes de pasar a diagnóstico diferencial.

```yaml
validacion_hipotesis:
  criterios:
    - nombre: "evidencia_suficiente"
      descripcion: "La hipótesis tiene al menos 2 piezas de evidencia que la respaldan"
      peso: 0.30

    - nombre: "sin_contradicciones"
      descripcion: "No existe evidencia que contradiga directamente la hipótesis"
      peso: 0.30

    - nombre: "coherencia_interna"
      descripcion: "La explicación de la hipótesis es internamente consistente"
      peso: 0.20

    - nombre: "plausibilidad"
      descripcion: "La hipótesis es plausible dado el contexto del proyecto"
      peso: 0.20

  umbral_minimo: 0.40
  accion_si_no_pasa: "Descartar hipótesis (no pasa a diagnóstico diferencial)"
```

---

## Ciclo de vida de una hipótesis

```yaml
hipotesis_lifecycle:
  GENERADA: "Hipótesis creada por el Hypothesis Engine"
    │
    ▼
  VALIDADA: "Pasó la validación inicial (confianza ≥ 0.40)"
    │
    ▼
  EN_EVALUACION: "En proceso de diagnóstico diferencial"
    │
    ├── CONFIRMADA: "Seleccionada como diagnóstico principal"
    │
    ├── ALTERNATIVA: "Descartada como principal pero sigue siendo posible"
    │
    └── DESCARTADA: "Refutada por evidencia o por diagnóstico diferencial"
```

---

## Salida del Hypothesis Engine

```yaml
# SDOS/evidence/hypothesis/<ejecucion_id>.json
{
  "ejecucion_id": "hyp_001",
  "timestamp": "2026-07-15T10:00:00Z",
  "indicadores_entrada": 25,
  "hipotesis_generadas": [
    {
      "id": "HYP-001",
      "titulo": "La documentación del proyecto se está deteriorando",
      "nivel": "salud",
      "dominio": "gobernanza",
      "confianza_inicial": 0.72,
      "origen": "compuesta",
      "estado": "VALIDADA"
    },
    {
      "id": "HYP-002",
      "titulo": "Riesgo de alucinación por falta de memoria actualizada",
      "nivel": "riesgo",
      "dominio": "ia",
      "confianza_inicial": 0.85,
      "origen": "patron",
      "patron_match": "MEMORY_STAGNATION",
      "estado": "VALIDADA"
    },
    {
      "id": "HYP-003",
      "titulo": "La deuda técnica está aumentando por falta de refactorización",
      "nivel": "riesgo",
      "dominio": "arquitectura",
      "confianza_inicial": 0.65,
      "origen": "caso",
      "caso_similar": "CASE-007",
      "similitud": 0.72,
      "estado": "VALIDADA"
    }
  ],
  "hipotesis_descartadas": 3,
  "tiempo_generacion_ms": 450
}
```
