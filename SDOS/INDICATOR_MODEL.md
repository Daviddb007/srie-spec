# INDICATOR MODEL — Modelo de Medición y Digital Twin

---

## ¿Qué es este documento?

INDICATOR_MODEL.md define **el modelo conceptual de la medición** en SRIE. No describe el Engine (INDICATOR_ENGINE.md) ni el catálogo (INDICATOR_CATALOG.md). Describe cómo se mide, qué es un indicador, qué es un score, qué es una tendencia y, fundamentalmente, **qué es el Digital Twin** del proyecto.

---

## ¿Qué significa medir en SRIE?

Medir no es calcular números. Medir es **responder preguntas** sobre el estado del sistema, respaldar cada respuesta con evidencia y calcular la confianza de cada afirmación.

Toda medición en SRIE sigue este flujo:

```
Pregunta → Evidencia → Cálculo → Confianza → Tendencia → Evento
```

---

## Los 4 niveles de medición

### Nivel 1: Observación

Hechos binarios constatables.

```
¿Existe README? → Sí (confianza: 0.99)
¿Existe ARCHITECTURE? → No (confianza: 1.0)
```

No interpreta. Solo constata.

### Nivel 2: Salud

Indicadores compuestos que evalúan calidad.

```
Documentation Health → 73.5 / 100 (confianza: 0.92)
```

Interpreta desde múltiples fuentes.

### Nivel 3: Riesgo

Diagnósticos que identifican peligros potenciales.

```
Maintenance Risk → ALTO (confianza: 0.78)
Hallucination Risk → MUY BAJO (confianza: 0.85)
```

Ya no es una métrica. Es un diagnóstico.

### Nivel 4: Predictivo

Estimaciones de probabilidad futura.

```
Production Failure Probability → 12% (confianza: 0.65)
```

Se basa en tendencias históricas y correlación de indicadores.

---

## La estructura de un indicador

```yaml
indicador:
  # Identidad
  id: "IND-020"
  nombre: "documentation_health"
  nivel: "salud"
  dominio: "gobernanza"
  entidad: "project"

  # Valor actual
  valor: 73.5
  rango: [0, 100]
  unidad: "puntos"

  # Confianza
  confianza: 0.92
  evidencia:
    fuentes: 4
    archivos: ["SDOS/PROJECT_STATE.md", "SDOS/GRAPH.md"]

  # Tendencia (últimas N mediciones)
  tendencia:
    direccion: "DECLINING"        # IMPROVING | STABLE | DECLINING | VOLATILE
    pendiente: -2.3               # Cambio promedio por medición
    historial: [82, 80, 78, 75, 73.5]
    ultima_medicion: "2026-07-15T10:00:00Z"
    ventana_dias: 30

  # Thresholds
  threshold:
    warning: 70
    critical: 40

  # Eventos
  evento_emitido: true
  evento_tipo: "DOCUMENTATION_WARNING"

  # Metadata
  responsable: "Indicators Engine"
  accion_sugerida: "Actualizar ARCHITECTURE.md y memory/index.md"
```

---

## Tendencias

Cada indicador mantiene un historial de mediciones. La tendencia se calcula a partir de ese historial.

```yaml
calculo_tendencia:
  metodo: "regresión lineal simple sobre últimas N mediciones"
  ventana_default: 30_días
  minimo_mediciones: 3

  clasificacion:
    IMPROVING: "pendiente > 1.0"
    STABLE: "pendiente entre -1.0 y 1.0"
    DECLINING: "pendiente < -1.0"
    VOLATILE: "desviación estándar > 15% del valor promedio"

  eventos_por_tendencia:
    IMPROVING: "emitir <INDICADOR>_IMPROVING"
    DECLINING: "emitir <INDICADOR>_DEGRADING"
    CRITICAL_DECLINING: "emitir <INDICADOR>_CRITICAL (si el valor cruza threshold critical)"
```

---

## Scores compuestos

Los scores son agregaciones de indicadores. No están en el catálogo. Se calculan dinámicamente.

```yaml
score:
  id: "engineering_score"
  nombre: "Engineering Score"
  rango: [0, 100]

  componentes:
    - indicador: "architecture_health"
      peso: 0.25
    - indicador: "deployment_health"
      peso: 0.20
    - indicador: "security_health"
      peso: 0.20
    - indicador: "maintenance_risk"
      peso: 0.20
      transformacion: "invertir (1.0 - risk) × 100"
    - indicador: "hallucination_risk"
      peso: 0.15
      transformacion: "invertir (1.0 - risk) × 100"

  formula: "Σ(indicador × peso)"
  calculo: "automático tras cada medición de componentes"
```

---

## El Digital Twin

El **Digital Twin** (Gemelo Digital) es la representación viva del proyecto, Domain o Federación dentro de SRIE. No es un archivo. Es el **modelo completo** que SRIE mantiene del sistema físico.

### ¿Qué contiene el Digital Twin?

```yaml
digital_twin:
  identidad: "IDENTITY.yaml"
  estado: "SDOS/STATE.md"
  estructura: "SDOS/PROJECT_STATE.md"
  grafo: "SDOS/GRAPH.md"
  registry: "SDOS/REGISTRY.yaml"
  indicadores: "SDOS/evidence/indicators/"
  tendencias: "SDOS/evidence/indicators/trends/"
  eventos: "SDOS/events/"
  memoria: "memory/"
  conocimiento: "knowledge/"
  contratos: "SDOS/federation/contracts/"
```

### Principios del Digital Twin

1. **El Twin es la fuente de verdad operativa.** Todos los Engines operan sobre el Twin, no directamente sobre los archivos del proyecto físico.

2. **El Twin se actualiza continuamente.** Discovery Engine sincroniza el Twin con el proyecto físico. Indicators Engine actualiza los indicadores del Twin.

3. **El Twin es consultable.** Cualquier Engine puede consultar el estado del Twin sin escanear el proyecto físico.

4. **El Twin es versionado.** Cada cambio en el Twin incrementa su versión. El historial de cambios se registra en `memory/context/`.

5. **El Twin permite simulaciones.** El Sandbox Engine puede crear un Twin temporal para simular cambios sin afectar el Twin real.

### Relación Twin → Proyecto físico

```
Proyecto Físico (archivos, git, dependencias)
       │
       │ Discovery Engine (sincronización)
       ▼
Digital Twin (modelo, grafo, indicadores)
       │
       │ Engines operan aquí
       ▼
Decisiones (Planner, Capability, Deployment)
       │
       │ Changes
       ▼
Proyecto Físico actualizado
```

### Twin en federación

En una federación, cada Domain tiene su propio Digital Twin. El Global Knowledge Graph es la unificación de los Twins públicos:

```
Digital Twin Marketing → publica → Grafo Público Marketing
Digital Twin Sales    → publica → Grafo Público Sales
                                     │
                                     ▼
                              Global Knowledge Graph
                              (unificación de Twins públicos)
```
