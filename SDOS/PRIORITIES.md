# PRIORITIES — Priorización del Sistema

---

## ¿Qué es este documento?

PRIORITIES.md define **cómo se priorizan** los objetivos, diagnósticos e intervenciones dentro de SRIE. No define qué es importante (eso es GOALS.md). Define **cómo decidir qué es más importante cuando todo parece urgente**.

---

## El problema de la priorización

Cuando múltiples diagnósticos compiten por atención, el sistema necesita decidir:

```
Diagnóstico A: Documentación deteriorada (salud: 68, declining)
Diagnóstico B: Deuda técnica alta (riesgo: 0.55, increasing)
Diagnóstico C: Seguridad vulnerable (salud: 45, critical)

¿Cuál atender primero?
```

La respuesta depende de: objetivos activos, restricciones, urgencia, impacto, esfuerzo y dependencias.

---

## Fórmula de priorización

```yaml
priorizacion:
  formula: |
    Prioridad = (impacto × 0.30)
              + (urgencia × 0.25)
              + (alineacion_objetivos × 0.20)
              - (esfuerzo_normalizado × 0.15)
              + (dependencias × 0.10)

  donde:
    impacto: "¿Cuánto mejora este diagnóstico al proyecto?"
             "0.0 - 1.0 (calculado desde la diferencia entre valor actual y threshold)"

    urgencia: "¿Qué tan rápido empeora si no se atiende?"
              "0.0 - 1.0 (basado en tendencia DECLINING y pendiente)"

    alineacion_objetivos: "¿Qué tanto se alinea con los objetivos activos?"
                          "0.0 - 1.0 (match con GOALS.md ponderado por peso)"

    esfuerzo_normalizado: "¿Cuánto esfuerzo requiere la intervención?"
                          "0.0 - 1.0 (estimado desde casos similares)"

    dependencias: "¿Este diagnóstico bloquea otros?"
                  "0.0 = no bloquea, 0.5 = bloquea algunos, 1.0 = bloquea muchos"

  clasificacion:
    CRITICA: "Prioridad ≥ 0.80 — Actuar inmediatamente"
    ALTA: "Prioridad ≥ 0.60 — Planificar para este ciclo"
    MEDIA: "Prioridad ≥ 0.40 — Planificar para próximo ciclo"
    BAJA: "Prioridad < 0.40 — Monitorear"
```

---

## Ejemplo de priorización

```yaml
diagnosticos:
  - id: "DX-001"
    titulo: "Documentación deteriorada"
    impacto: 0.70
    urgencia: 0.60
    alineacion_objetivos: 0.80 (GOAL-003: Gobernanza)
    esfuerzo_normalizado: 0.30
    dependencias: 0.20
    prioridad: (0.70×0.30)+(0.60×0.25)+(0.80×0.20)-(0.30×0.15)+(0.20×0.10)
             = 0.21 + 0.15 + 0.16 - 0.045 + 0.02
             = 0.495
    clasificacion: "MEDIA"

  - id: "DX-003"
    titulo: "Vulnerabilidad de seguridad"
    impacto: 0.95
    urgencia: 0.90
    alineacion_objetivos: 0.90 (GOAL-004: Seguridad)
    esfuerzo_normalizado: 0.40
    dependencias: 0.50
    prioridad: 0.285 + 0.225 + 0.18 - 0.06 + 0.05 = 0.68
    clasificacion: "ALTA"
```

---

## Límite de trabajo simultáneo

```yaml
limite_trabajo:
  maximo_intervenciones_simultaneas: 2
  regla: "No iniciar una intervención de prioridad MEDIA o BAJA si hay 2+ intervenciones activas"
  excepcion: "Intervenciones CRÍTICAS siempre pueden iniciarse (desplazan a las de menor prioridad)"
```

---

## Re-priorización

```yaml
repriorizacion:
  disparadores:
    - "Nuevo diagnóstico CRÍTICO"
    - "Cambio en objetivos (GOALS.md)"
    - "Cambio en restricciones (CONSTRAINTS.md)"
    - "Fallo de intervención en curso"
    - "Cada 7 días (re-priorización programada)"

  proceso:
    1. "Recalcular prioridad de todos los diagnósticos activos"
    2. "Si hay cambios significativos (> 0.10):"
       a. "Notificar al Planner"
       b. "Si una intervención en curso fue desplazada: evaluar cancelación"
       c. "Registrar evento PRIORITY_CHANGED"
```

---

## Priorización por nivel de madurez

| Nivel | Criterio de priorización | Enfoque |
|-------|--------------------------|---------|
| L0 | Impacto > Urgencia | Establecer bases |
| L1 | Urgencia > Impacto | Tapar huecos críticos |
| L2 | Alineación con objetivos | Construir sobre lo que importa |
| L3 | Esfuerzo < Impacto | Optimizar retorno |
| L4 | Dependencias > Todo | No dejar bloqueos |
| L5 | Balance automático | El sistema prioriza solo |
