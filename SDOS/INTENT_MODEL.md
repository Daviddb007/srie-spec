# INTENT MODEL — Modelo de Intención del Sistema

---

## ¿Qué es este documento?

INTENT_MODEL.md define **la capa de intención** de SRIE — el nivel que responde "¿qué estamos intentando lograr?" antes de que cualquier Engine decida "cómo hacerlo".

Esta capa es la **conciencia del sistema**. No ejecuta. No descubre. No mide. **Decide qué vale la pena perseguir.**

```
DIAGNOSIS → ¿Qué está mal?
    │
    ▼
INTENT → ¿Qué queremos conseguir?
    │
    ▼
PLANNER → ¿Cómo llegamos?
```

---

## ¿Por qué existe la Intent Layer?

Un mismo diagnóstico puede requerir estrategias completamente distintas según la intención:

| Diagnóstico | Intención A: "Salir rápido al mercado" | Intención B: "Máxima seguridad" |
|-------------|----------------------------------------|----------------------------------|
| Arquitectura frágil | Deuda técnica aceptable, priorizar features | Refactorizar antes de añadir features |
| Documentación baja | Documentación mínima, priorizar código | Documentar cada decisión |
| Tests insuficientes | Tests críticos solamente | Cobertura ≥ 90% |

El diagnóstico es idéntico. La estrategia cambia por la intención.

---

## El flujo de la Intent Layer

```
DX ENGINE (diagnóstico)
    │
    ▼
INTENT RESOLVER
    │
    ├── Leer PROJECT_INTENT.md (objetivos, restricciones, prioridades)
    ├── Leer CONSTRAINTS.md (lo que nunca puede romperse)
    ├── Leer GOALS.md (objetivos estratégicos)
    ├── Consultar DECISION_REGISTRY.md (decisiones previas)
    │
    ▼
INTENT DEFINITION
    │
    ├── Objetivo priorizado
    ├── Restricciones activas
    ├── Tradeoffs aceptables
    └── Criterios de éxito
    │
    ▼
PLANNER ENGINE (plan)
```

---

## ¿Qué es una intención?

Una intención es la **respuesta estructurada** a la pregunta "¿qué queremos lograr?" ante un diagnóstico concreto.

```yaml
intencion:
  id: "INT-001"
  timestamp: "2026-07-15T10:00:00Z"

  diagnostico_origen: "DX-001"
  resumen_diagnostico: "La documentación del proyecto se está deteriorando"

  objetivo_priorizado:
    id: "GOAL-003"
    nombre: "Gobernanza sostenible"
    peso: 0.35

  restricciones_activas:
    - id: "CON-001"
      nombre: "No romper compatibilidad hacia atrás"
    - id: "CON-003"
      nombre: "Presupuesto mensual fijo"

  tradeoffs_aceptables:
    - decision: "Aceptar 2 semanas sin nuevas features"
      a_cambio_de: "Recuperar documentation_health a 80+"
      riesgo: "MEDIO"

  criterios_exito:
    - "documentation_health >= 80 en 30 días"
    - "architecture_exists = true"
    - "memory_freshness >= 70"

  confianza_intencion: 0.85
  estado: "ACTIVA"
```

---

## Componentes de la Intent Layer

### 1. Intent Resolver

Toma un diagnóstico y lo cruza con los objetivos, restricciones y prioridades del proyecto para producir una intención.

```yaml
intent_resolver:
  entrada: "Diagnóstico + PROJECT_INTENT.md + CONSTRAINTS.md + GOALS.md"
  proceso:
    1. "Identificar dominio del diagnóstico"
    2. "Seleccionar objetivos relevantes para ese dominio"
    3. "Ponderar objetivos según prioridades"
    4. "Activar restricciones del dominio"
    5. "Calcular tradeoffs disponibles"
    6. "Generar intención priorizada"
  salida: "Intención con objetivo, restricciones, tradeoffs y criterios de éxito"
```

### 2. Goal Selector

Selecciona el objetivo más relevante para el diagnóstico actual.

```yaml
goal_selector:
  formula: |
    relevancia_objetivo = (match_diagnostico × peso_objetivo)
                        - (penalizacion_por_restricciones)
                        + (sinergia_con_otros_objetivos)

    match_diagnostico: ¿Qué tanto se alinea este objetivo con el diagnóstico?
    peso_objetivo: Peso definido en GOALS.md
    penalizacion_por_restricciones: Si el objetivo viola una restricción activa
    sinergia: Si lograr este objetivo también ayuda a otros
```

### 3. Constraint Checker

Verifica que la intención no viole ninguna restricción activa.

```yaml
constraint_checker:
  proceso:
    - "Para cada restricción activa: verificar compatibilidad con la intención"
    - "Si una restricción se viola: marcar intención como NO_VÁLIDA"
    - "Si ninguna restricción se viola: marcar intención como VÁLIDA"
  salida: "VÁLIDA | NO_VÁLIDA (con razón)"
```

### 4. Tradeoff Analyzer

Evalúa los tradeoffs disponibles para la intención.

```yaml
tradeoff_analyzer:
  proceso:
    - "Para cada tradeoff disponible: calcular impacto en el objetivo"
    - "Seleccionar tradeoff con mejor relación impacto/riesgo"
  salida: "Tradeoff seleccionado + justificación"
```

---

## PROJECT_INTENT.md

Cada proyecto (o Domain) tiene un archivo `PROJECT_INTENT.md` que define su constitución estratégica:

```yaml
# PROJECT_INTENT.md
project_intent:
  proposito: "Construir una plataforma de ingeniería empresarial asistida por IA"

  objetivos:
    - id: "GOAL-001"
      nombre: "Time to market"
      descripcion: "Entregar valor al cliente lo antes posible"
      peso: 0.30
      dominio: "desarrollo"

    - id: "GOAL-002"
      nombre: "Excelencia técnica"
      descripcion: "Código mantenible, testeable, bien documentado"
      peso: 0.25
      dominio: "arquitectura"

    - id: "GOAL-003"
      nombre: "Gobernanza sostenible"
      descripcion: "El proyecto debe ser gobernable y medible"
      peso: 0.20
      dominio: "gobernanza"

    - id: "GOAL-004"
      nombre: "Seguridad primero"
      descripcion: "No comprometer seguridad por velocidad"
      peso: 0.15
      dominio: "seguridad"

    - id: "GOAL-005"
      nombre: "Crecimiento orgánico"
      descripcion: "Estructura que permita escalar sin reescribir"
      peso: 0.10
      dominio: "arquitectura"

  restricciones:
    - "No romper compatibilidad hacia atrás"
    - "Presupuesto mensual fijo (no contratar más personas)"
    - "Soporte para Python 3.12+"
    - "Deploy en Linux (DigitalOcean)"

  no_negociables:
    - "Los secretos nunca se hardcodean"
    - "No se deploya sin tests pasando"
    - "No se elimina memoria del sistema"

  prioridades:
    criterio: "impacto × urgencia / esfuerzo"
    maximos_simultaneos: 2

  criterios_exito_generales:
    - "SRIE_SCORE >= 80"
    - "hallucination_risk < 0.15"
    - "Deployments exitosos > 95%"
```

---

## Intent Registry

Las intenciones se registran en `SDOS/evidence/intent/`:

```yaml
# SDOS/evidence/intent/INT-001.yaml
intencion:
  id: "INT-001"
  estado: "ACTIVA"         # ACTIVA | COMPLETADA | CANCELADA | REEMPLAZADA
  creada: "2026-07-15T10:00:00Z"
  completada: null

  diagnostico_origen: "DX-001"
  planner_resultante: "PLAN-001"

  objetivo_priorizado: "GOAL-003"
  tradeoff_seleccionado: "Aceptar 2 semanas sin features por documentación"

  resultado:
    documentation_health_post: 85
    mejoria: "+17 puntos"
    exito: true
    lecciones: "La intención clara permitió al Planner priorizar correctamente"
```
