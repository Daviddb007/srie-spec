# TIMELINE — Temporalidad del Sistema

---

## ¿Qué es este documento?

TIMELINE.md define la **dimensión temporal** de SRIE. No solo el historial. La capacidad del sistema de representar **pasado, presente y futuro** — y de navegar entre ellos.

Las organizaciones no viven en un instante. Evolucionan. SRIE debe poder responder: ¿cómo era el proyecto hace 3 meses? ¿cómo estará en 30 días si seguimos esta estrategia?

---

## Las tres líneas de tiempo

```yaml
timeline:
  pasado: "PROJECT_TIMELINE.md (historial desde git + memoria)"
  presente: "Digital Twin (estado actual, indicadores, diagnósticos)"
  futuro: "PROJECTION.md (escenarios, simulaciones, objetivos)"
```

---

## Pasado: PROJECT_TIMELINE.md

(Ver DISCOVERY_OUTPUTS.md para el formato completo)

El timeline pasado se construye desde:
- Git (commits, ramas, autores, fechas)
- Memoria (ADR, contextos de iteración)
- Evidencia (ejecuciones de Engines)
- Decisiones (DECISION_REGISTRY.md)

```yaml
timeline_pasado:
  fuentes:
    - "SDOS/PROJECT_TIMELINE.md"
    - "memory/context/"
    - "SDOS/decisions/"
    - "SDOS/evidence/"
    - "git log"
  
  consultas:
    - "¿Cuándo nació esta capability?"
    - "¿Cuándo apareció este bug?"
    - "¿Cuándo cambió la arquitectura?"
    - "¿Cuándo se rompió el deploy?"
    - "¿Cuál era el SRIE_SCORE hace 3 meses?"
```

---

## Presente: Digital Twin

(Ver INDICATOR_MODEL.md)

El timeline presente es el Digital Twin en su versión actual. Es el punto de partida para cualquier decisión.

```yaml
timeline_presente:
  twin_version: 12
  ultima_actualizacion: "2026-07-15T10:00:00Z"
  indicadores_actuales:
    engineering_score: 78.3
    organizational_score: 82.1
    knowledge_score: 65.4
    evolution_score: 71.2
    global_score: 74.8
  diagnosticos_activos:
    - "DOCUMENTATION_DECAY"
    - "TECH_DEBT_ACCUMULATION"
```

---

## Futuro: PROJECTION.md

El timeline futuro es el **escenario proyectado** — qué pasaría si no se interviene versus qué pasaría con cada intervención posible.

Se genera por el Simulation Engine (ver SIMULATION_ENGINE.md).

```yaml
timeline_futuro:
  escenario_base: "Sin intervención"
  proyeccion_30d:
    documentation_health: 62 (DECLINING)
    tech_debt_risk: 0.62 (INCREASING)
    hallucination_risk: 0.12 (INCREASING)
    engineering_score: 74.2

  escenario_intervencion: "Priorizar documentación"
  proyeccion_30d:
    documentation_health: 82 (IMPROVING)
    tech_debt_risk: 0.50 (STABLE)
    hallucination_risk: 0.06 (DECLINING)
    engineering_score: 80.1
```

---

## Timeline navigator

```yaml
timeline_navigator:
  metodos:
    - nombre: "snapshot(fecha)"
      descripcion: "Obtener el Digital Twin en una fecha específica"
      requiere: "Que exista evidencia de esa fecha"

    - nombre: "diff(fecha_a, fecha_b)"
      descripcion: "Comparar dos estados del sistema"
      salida: "Cambios en indicadores, estructura, diagnóstico"

    - nombre: "trend(indicador, periodo)"
      descripcion: "Tendencia de un indicador en un período"
      salida: "Valores, pendiente, clasificación"

    - nombre: "project(escenario, horizonte_dias)"
      descripcion: "Proyectar estado futuro bajo un escenario"
      salida: "Estado futuro estimado"
```

---

## Registro temporal

```yaml
temporal_registry:
  archivo: "SDOS/TIMELINE.yaml"
  contenido:
    pasado:
      primer_commit: "2026-01-15"
      primer_discovery: "2026-07-10"
      iteraciones_completadas: 2
    presente:
      iteracion_actual: 3
      twin_version: 12
      fase_sdos: "DX"
    futuro:
      proyecciones_disponibles: true
      escenarios: ["base", "intervencion_docs", "intervencion_seguridad"]
      horizonte_maximo_dias: 90
```
