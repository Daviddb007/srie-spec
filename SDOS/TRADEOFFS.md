# TRADEOFFS — Compensaciones entre Opciones

---

## ¿Qué es este documento?

TRADEOFFS.md define **cómo SRIE maneja las compensaciones** entre opciones cuando no existe una solución perfecta. En ingeniería, casi nunca hay una opción correcta y otra incorrecta. Hay opciones con diferentes perfiles de impacto, riesgo y esfuerzo. Este documento define cómo elegir entre ellas.

---

## ¿Qué es un tradeoff?

Un tradeoff es la **renuncia consciente** a un beneficio para obtener otro. SRIE no oculta los tradeoffs: los explicita, los documenta y los convierte en parte de la memoria del sistema.

```yaml
tradeoff:
  id: "TO-001"
  nombre: "Velocidad vs Documentación"
  dominio: "gobernanza"

  descripcion: |
    Aceptar 2 semanas sin nuevas features para dedicar
    el tiempo del equipo a recuperar la documentación.

  opciones:
    - opcion: "A"
      nombre: "Priorizar documentación"
      impacto_positivo:
        - "documentation_health: +20 puntos"
        - "architecture_exists: true"
      impacto_negativo:
        - "features_entregadas: 0 por 2 semanas"
        - "time_to_market: +14 días"
      riesgo: "MEDIO"
      esfuerzo_dias: 10

    - opcion: "B"
      nombre: "Priorizar features"
      impacto_positivo:
        - "features_entregadas: 3 en 2 semanas"
        - "time_to_market: -14 días"
      impacto_negativo:
        - "documentation_health: -5 puntos más"
        - "hallucination_risk: +0.05"
      riesgo: "ALTO"
      esfuerzo_dias: 0

  decision: "A"
  razon: "El diagnóstico DOCUMENTATION_DECAY es crítico para la gobernanza.
           La tendencia DECLINING continuará empeorando si no se interviene ahora."
  confianza_decision: 0.78
```

---

## Evaluación de tradeoffs

```yaml
evaluacion_tradeoff:
  formula: |
    Valor_opcion = (impacto_positivo_total × 0.35)
                 - (impacto_negativo_total × 0.25)
                 - (riesgo_normalizado × 0.20)
                 - (esfuerzo_normalizado × 0.10)
                 + (alineacion_objetivos × 0.10)

  donde:
    impacto_positivo_total: "Suma de impactos positivos ponderados por valor"
    impacto_negativo_total: "Suma de impactos negativos ponderados por valor"
    riesgo_normalizado: "0.0 = bajo, 1.0 = crítico"
    esfuerzo_normalizado: "0.0 = mínimo, 1.0 = máximo"
    alineacion_objetivos: "0.0 - 1.0 (alineación con GOALS.md)"
```

---

## Catálogo de tradeoffs comunes

### TO-001: Velocidad vs Documentación

```yaml
id: "TO-001"
nombre: "Velocidad vs Documentación"
dominio: "gobernanza"
cuando_aplica: "documentation_health < 70 Y tendencia DECLINING"
```

### TO-002: Deuda técnica vs Features

```yaml
id: "TO-002"
nombre: "Deuda técnica vs Features"
dominio: "arquitectura"
cuando_aplica: "tech_debt_risk > 0.50 Y maintenance_risk > 0.40"
```

### TO-003: Seguridad vs Velocidad

```yaml
id: "TO-003"
nombre: "Seguridad vs Velocidad"
dominio: "seguridad"
cuando_aplica: "security_health < 60"
nota: "La seguridad tiene prioridad sobre la velocidad según GOAL-004"
```

### TO-004: Refactor vs Nueva funcionalidad

```yaml
id: "TO-004"
nombre: "Refactor vs Nueva funcionalidad"
dominio: "arquitectura"
cuando_aplica: "architecture_health < 60 Y feature_pressure alta"
```

### TO-005: Estándar vs Propietario

```yaml
id: "TO-005"
nombre: "Estándar vs Propietario"
dominio: "arquitectura"
cuando_aplica: "Decisión de adopción tecnológica"
recomendacion: "Preferir estándares abiertos a menos que el propietario ofrezca ventaja competitiva clara"
```

### TO-006: Centralización vs Federación

```yaml
id: "TO-006"
nombre: "Centralización vs Federación"
dominio: "organizacional"
cuando_aplica: "federation_health < 50"
```

---

## Registro de tradeoffs

Cada tradeoff evaluado se registra en `SDOS/evidence/tradeoffs/`:

```yaml
# SDOS/evidence/tradeoffs/TO-001-2026-07-15.yaml
tradeoff:
  id: "TO-001"
  evaluacion: "2026-07-15"
  contexto: "DX-001 (Documentación deteriorada)"

  opciones:
    - opcion: "A"
      valor: 0.72
      seleccionada: true
    - opcion: "B"
      valor: -0.15
      seleccionada: false

  decision: "A"
  justificacion: "El diagnóstico es crítico. La tendencia DECLINING requiere intervención inmediata."
  confianza: 0.78

  seguimiento:
    resultado_real: "documentation_health mejoró de 68 a 85 en 35 días"
    desviacion: "+3 puntos sobre lo esperado"
    leccion: "El tradeoff fue correcto. El equipo respondió bien."
```

---

## Reglas de tradeoffs

1. **Todo tradeoff debe tener al menos 2 opciones.** Si solo hay una opción, no hay tradeoff.
2. **Cada opción debe explicitar impacto positivo y negativo.** No se ocultan las desventajas.
3. **El tradeoff seleccionado debe incluir justificación.** "Por qué A sobre B" debe ser explícito.
4. **Los tradeoffs se registran en el Decision Registry.** Son decisiones estratégicas.
5. **Si el resultado real se desvía > 30% del esperado, el tradeoff se marca para revisión** y el Knowledge Engine lo analiza.
