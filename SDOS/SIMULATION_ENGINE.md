# SIMULATION ENGINE — Motor de Simulación

---

## ¿Qué es este documento?

SIMULATION_ENGINE.md define el **motor de simulación** de SRIE. No es el Sandbox (que valida cambios reales). Es un motor que responde **"¿qué pasaría si...?"** sin ejecutar nada real. Permite al Planner evaluar múltiples escenarios antes de decidir.

---

## ¿Qué es una simulación?

Una simulación proyecta el estado futuro del Digital Twin bajo un escenario hipotético:

```yaml
simulacion:
  id: "SIM-001"
  escenario: "Priorizar documentación 2 semanas"
  horizonte_dias: 30

  estado_inicial:
    documentation_health: 68.2
    tech_debt_risk: 0.55
    engineering_score: 78.3

  intervenciones:
    - accion: "Crear ARCHITECTURE.md"
      esfuerzo_dias: 2
      impacto_esperado: {documentation_health: +8}
      confianza_impacto: 0.85

    - accion: "Actualizar memory/index.md"
      esfuerzo_dias: 1
      impacto_esperado: {documentation_health: +5}
      confianza_impacto: 0.80

    - accion: "Agregar docs al definition of done"
      esfuerzo_dias: 1
      impacto_esperado: {documentation_health: +3, tech_debt_risk: -0.02}
      confianza_impacto: 0.60

  estado_proyectado:
    documentation_health: 82.0
    tech_debt_risk: 0.50
    engineering_score: 80.1
    halluncination_risk: 0.06

  riesgos:
    - riesgo: "2 semanas sin features pueden afectar time to market"
      probabilidad: 0.30
      impacto: "MEDIO"

  confianza_simulacion: 0.72
```

---

## Casos de uso

```yaml
use_cases:
  - nombre: "Evaluar intervención"
    pregunta: "¿Qué pasa si priorizamos documentación 2 semanas?"
    entrada: "Diagnóstico + plan de intervención"
    salida: "Estado futuro proyectado + riesgos"

  - nombre: "Comparar escenarios"
    pregunta: "¿Es mejor priorizar documentación o seguridad?"
    entrada: "Diagnóstico + múltiples planes"
    salida: "Comparación de estados futuros"

  - nombre: "Detectar efectos secundarios"
    pregunta: "¿Qué más se rompe si hacemos esto?"
    entrada: "Plan de intervención"
    salida: "Indicadores que empeorarían"

  - nombre: "Estimar esfuerzo"
    pregunta: "¿Cuánto tiempo toma llegar a SRIE_SCORE 85?"
    entrada: "Estado actual + objetivo"
    salida: "Plan estimado + duración + confianza"
```

---

## Motor de simulación

```yaml
simulation_engine:
  entrada:
    twin_actual: "Digital Twin actual"
    escenario:
      intervenciones: [{accion, params}]
      horizonte_dias: integer

  proceso:
    1. "Clonar Digital Twin (simulación no afecta al real)"
    2. "Aplicar cada intervención al Twin simulado"
    3. "Proyectar indicadores usando:"
       - "Tendencias históricas del indicador"
       - "Impacto conocido de intervenciones similares (de CASES)"
       - "Correlaciones entre indicadores (de PATTERNS)"
    4. "Detectar efectos secundarios (indicadores que empeoran)"
    5. "Calcular confianza de la proyección"
    6. "Generar reporte de simulación"

  salida:
    estado_proyectado: {}
    riesgos: [{riesgo, probabilidad, impacto}]
    efectos_secundarios: [{indicador, cambio_esperado}]
    confianza: float
```

---

## Proyección de indicadores

```yaml
proyeccion:
  formula: |
    Valor_futuro = Valor_actual
                  + (tendencia_natural × horizonte)
                  + Σ(impacto_intervenciones)
                  + ruido_aleatorio(confianza)

  donde:
    tendencia_natural: "Pendiente de regresión lineal del historial"
    impacto_intervenciones: "Suma de impactos conocidos de casos similares"
    ruido_aleatorio: "Variación basada en la confianza de la simulación"

  confianza_proyeccion:
    alta: "≥ 10 casos similares en CASE_REGISTRY"
    media: "3-9 casos similares"
    baja: "< 3 casos similares o escenario nuevo"
```

---

## Comparación de escenarios

```yaml
comparacion:
  entrada: "Múltiples simulaciones"
  salida:
    - escenario: "Priorizar documentación"
      engineering_score_final: 80.1
      riesgo: 0.30
      duracion_dias: 14
      confianza: 0.72

    - escenario: "Priorizar seguridad"
      engineering_score_final: 76.5
      riesgo: 0.15
      duracion_dias: 10
      confianza: 0.68

    - escenario: "No hacer nada"
      engineering_score_final: 74.2
      riesgo: 0.55
      duracion_dias: 0
      confianza: 0.85

  recomendacion: "Priorizar documentación (mejor score final a pesar del riesgo)"
```

---

## Simulación como servicio

```yaml
simulation_service:
  metodo: "simulate(escenario) → Simulación"

  ejemplos:
    - simulate({intervencion: "crear ARCHITECTURE.md", horizonte: 30})
    - simulate_all([escenario_A, escenario_B, escenario_C])
    - simulate_to_target({objetivo: "engineering_score >= 85"})

  rate_limit: "10 simulaciones por minuto"
  cache: "Resultados de simulación tienen TTL de 1 hora (los mismos inputs = mismo output)"
```

---

## Limitaciones conocidas

```yaml
limitaciones:
  - "La precisión de la simulación depende de la cantidad de casos en CASE_REGISTRY"
  - "Escenarios sin precedentes tienen confianza baja"
  - "Efectos secundarios no evidentes pueden no detectarse"
  - "La simulación no reemplaza al Sandbox: es orientativa, no confirmatoria"
```
