# GOALS — Objetivos Estratégicos del Proyecto

---

## ¿Qué es este documento?

GOALS.md define los **objetivos estratégicos** que un proyecto o Domain puede perseguir. Los objetivos son la entrada principal de la Intent Layer: determinan qué prioriza el Planner cuando recibe un diagnóstico.

Los objetivos no son tareas. Son **direcciones estratégicas** que guían todas las decisiones del sistema.

---

## ¿Qué es un objetivo?

```yaml
goal:
  id: "GOAL-001"
  nombre: "Time to market"
  tipo: "estrategico"           # estrategico | tactico | operativo
  dominio: "desarrollo"

  descripcion: "Entregar valor al cliente lo antes posible"

  peso: 0.30                    # Importancia relativa (0.0 - 1.0)

  diagnosticos_alineados:
    - "DOCUMENTATION_DECAY"
    - "TECH_DEBT_ACCUMULATION"

  metricas_asociadas:
    - "deployment_health"
    - "maintenance_risk"

  conflictos_con:
    - goal: "GOAL-002"          # Excelencia técnica
      descripcion: "Time to market y excelencia técnica pueden estar en conflicto"

  sinergias_con:
    - goal: "GOAL-005"          # Crecimiento orgánico
      descripcion: "Entregar rápido permite validar hipótesis de crecimiento"

  horizonte: "corto"            # corto | mediano | largo

  criterios_exito:
    - "Deploy semanal"
    - "Feedback de cliente en < 7 días"
    - "Feature lead time < 14 días"
```

---

## Catálogo de objetivos

### GOAL-001: Time to market

```yaml
id: "GOAL-001"
nombre: "Time to market"
peso: 0.30
dominio: "desarrollo"
horizonte: "corto"
descripcion: "Entregar valor al cliente lo antes posible"
```

### GOAL-002: Excelencia técnica

```yaml
id: "GOAL-002"
nombre: "Excelencia técnica"
peso: 0.25
dominio: "arquitectura"
horizonte: "largo"
descripcion: "Código mantenible, testeable y bien documentado"
```

### GOAL-003: Gobernanza sostenible

```yaml
id: "GOAL-003"
nombre: "Gobernanza sostenible"
peso: 0.20
dominio: "gobernanza"
horizonte: "mediano"
descripcion: "El proyecto debe ser gobernable y medible en todo momento"
```

### GOAL-004: Seguridad primero

```yaml
id: "GOAL-004"
nombre: "Seguridad primero"
peso: 0.15
dominio: "seguridad"
horizonte: "siempre"
descripcion: "No comprometer seguridad por velocidad"
```

### GOAL-005: Crecimiento orgánico

```yaml
id: "GOAL-005"
nombre: "Crecimiento orgánico"
peso: 0.10
dominio: "arquitectura"
horizonte: "largo"
descripcion: "Estructura que permita escalar sin reescribir"
```

### GOAL-006: Colaboración federada

```yaml
id: "GOAL-006"
nombre: "Colaboración federada"
peso: 0.20
dominio: "organizacional"
horizonte: "mediano"
entidad: "federation"
descripcion: "Los Domains de la federación colaboran eficientemente"
```

### GOAL-007: Soberanía de datos

```yaml
id: "GOAL-007"
nombre: "Soberanía de datos"
peso: 0.15
dominio: "seguridad"
horizonte: "siempre"
descripcion: "Cada Domain controla su propia memoria y datos"
```

---

## Matriz de conflictos entre objetivos

| Objetivo A | Objetivo B | Conflicto | Resolución |
|------------|------------|-----------|------------|
| Time to market | Excelencia técnica | ALTO | Priorizar según fase del proyecto (early = speed, late = quality) |
| Time to market | Gobernanza | MEDIO | Documentación mínima aceptable, no cero |
| Seguridad | Time to market | ALTO | Seguridad no negociable, pero puede ser progresiva |
| Excelencia técnica | Crecimiento orgánico | BAJO | Sinérgicos: buena arquitectura permite escalar |
| Colaboración federada | Soberanía de datos | MEDIO | Contratos explícitos definen qué se comparte |

---

## Ciclo de vida de un objetivo

```yaml
goal_lifecycle:
  DEFINIDO: "Objetivo creado en PROJECT_INTENT.md"
    │
    ▼
  ACTIVO: "Objetivo vigente, usado por Intent Layer"
    │
    ├── COMPLETADO: "Objetivo alcanzado (criterios de éxito cumplidos)"
    │
    ├── REPRIORIZADO: "Peso modificado por cambio estratégico"
    │
    └── ARCHIVADO: "Ya no es relevante"
```

---

## Objetivos por nivel de madurez

| Nivel SRIE | Objetivos recomendados |
|------------|------------------------|
| L0 | GOAL-001 (time to market) |
| L1 | GOAL-001, GOAL-003 (gobernanza) |
| L2 | GOAL-001, GOAL-003, GOAL-004 (seguridad) |
| L3 | GOAL-002 (excelencia técnica), GOAL-005 (crecimiento) |
| L4 | GOAL-006 (colaboración federada) |
| L5 | Todos los objetivos, ponderados estratégicamente |
