# DISCOVERY EVIDENCE — Evidencia Válida y Cálculo de Confianza

---

## ¿Qué es este documento?

DISCOVERY_EVIDENCE.md define **qué constituye evidencia válida** para el Discovery Engine y **cómo se calcula la confianza** de cada afirmación. Este documento es la base para que SRIE sea auditable: ninguna afirmación existe sin evidencia que la respalde y una puntuación explícita que mida su certeza.

---

## ¿Qué es evidencia válida?

En SRIE, la evidencia no es cualquier archivo. La evidencia debe cumplir estos criterios:

1. **Observable** — Debe existir en el sistema de archivos del proyecto.
2. **Legible** — Debe ser parseable por el Engine (texto, JSON, YAML, TOML, etc.).
3. **Trazable** — Debe tener una ruta y un hash que permitan verificarla.
4. **Falsable** — Debe ser posible encontrar evidencia que la contradiga.
5. **Independiente** — Dos observadores diferentes deben encontrar la misma evidencia.

### Tipos de evidencia

| Tipo | Descripción | Ejemplo | Peso base |
|------|-------------|---------|-----------|
| `dependencia_explicita` | Declaración directa en archivo de configuración | `fastapi` en `pyproject.toml` | 0.40 |
| `import_directo` | Import o require en código fuente | `from fastapi import FastAPI` | 0.35 |
| `patron_arquitectura` | Estructura de directorios o convención | `app/routers/`, `app/models/` | 0.25 |
| `documentacion_explicita` | Mención directa en documentación | `README.md` menciona "FastAPI" | 0.30 |
| `configuracion_herramienta` | Configuración específica de herramienta | `Dockerfile` expone puerto 8000 | 0.35 |
| `historial_git` | Evidencia en commits, ramas, tags | Commit "feat: add FastAPI routes" | 0.20 |
| `convencion_lenguaje` | Patrón reconocido del lenguaje | `__init__.py` indica paquete Python | 0.15 |
| `relacion_estructural` | Relación entre archivos | `calendar.manifest` declara dependencia | 0.30 |

### Evidencia NO válida

No constituye evidencia:

- Inferencias no respaldadas ("esto parece...")
- Patrones adivinados sin archivos que los confirmen
- Información del contexto de conversación que no esté en archivos
- Suposiciones sobre la intención del desarrollador
- Documentación externa al proyecto (documentación de librerías, tutoriales)

---

## Cálculo de Confidence Score

### Fórmula principal

```
Confidence = (source_quality × 0.30)
           + (evidence_diversity × 0.25)
           + (consistency_score × 0.25)
           + (specificity_score × 0.20)
```

### Componentes

#### 1. Source Quality (0.0 — 1.0)

Mide la calidad promedio de las fuentes de evidencia.

```yaml
source_quality:
  formula: "promedio(peso_cada_fuente)"
  factores:
    - "Peso base del tipo de evidencia (ver tabla arriba)"
    - "+0.10 si la fuente está en la raíz del proyecto"
    - "+0.05 si la fuente es un archivo de configuración estándar"
    - "-0.10 si la fuente es documentación no estructurada"
    - "-0.20 si la fuente es un archivo generado automáticamente"
```

#### 2. Evidence Diversity (0.0 — 1.0)

Mide cuántos tipos diferentes de evidencia respaldan la misma afirmación.

```yaml
evidence_diversity:
  formula: "min(tipos_distintos / 3, 1.0)"
  ejemplo: |
    Si una afirmación tiene:
    - dependencia_explicita (pyproject.toml)
    - import_directo (app/main.py)
    - documentacion_explicita (README.md)
    → 3 tipos distintos → diversity = 1.0
    → Solo 1 tipo → diversity = 0.33
```

#### 3. Consistency Score (0.0 — 1.0)

Mide si la evidencia es consistente entre sí y no hay contradicciones.

```yaml
consistency_score:
  formula: "1.0 - (contradicciones_encontradas / max_contradictions)"
  max_contradictions: 5
  reglas:
    - "Si 0 contradicciones → consistency = 1.0"
    - "Si 2+ contradicciones → consistency = max(0, 1.0 - count/5)"
    - "Una contradicción sola reduce a 0.80"
    - "Ejemplo: pyproject.toml dice Flask, imports muestran FastAPI → contradicción → consistency baja"
```

#### 4. Specificity Score (0.0 — 1.0)

Mide qué tan específica es la evidencia para la afirmación.

```yaml
specificity_score:
  formula: |
    1.0 = evidencia DIRECTA (archivo específico, import explícito)
    0.7 = evidencia FUERTE (patrón reconocible, estructura)
    0.4 = evidencia DÉBIL (convención, inferencia)
    0.1 = evidencia ESPECULATIVA (sin archivos confirmatorios)
  ejemplos:
    - "El archivo pyproject.toml contiene 'fastapi'" → 1.0
    - "Hay un directorio app/domain/" → 0.7
    - "Los imports usan snake_case (convención Python)" → 0.4
    - "El proyecto 'parece' usar DDD" → 0.1
```

### Ejemplo completo

```yaml
afirmacion: "El proyecto usa FastAPI como framework web"

evidencia:
  - fuente: "pyproject.toml"
    tipo: dependencia_explicita
    peso_base: 0.40
    especificidad: 1.0
  - fuente: "app/main.py"
    tipo: import_directo
    peso_base: 0.35
    especificidad: 1.0
  - fuente: "app/routers/"
    tipo: patron_arquitectura
    peso_base: 0.25
    especificidad: 0.7

calculo:
  source_quality:    (0.40 + 0.35 + 0.25) / 3 = 0.333
  evidence_diversity:   3 tipos / 3 = 1.000
  consistency_score:    0 contradicciones = 1.000
  specificity_score:    (1.0 + 1.0 + 0.7) / 3 = 0.900

  confidence: (0.333 × 0.30) + (1.000 × 0.25) + (1.000 × 0.25) + (0.900 × 0.20)
            = 0.100 + 0.250 + 0.250 + 0.180
            = 0.780

  hallucination_risk: 1.0 - 0.780 = 0.220
  clasificacion: "MEDIA" (>= 0.60)
```

---

## Clasificación de confianza

| Rango | Clasificación | Significado | Acción del Engine |
|-------|---------------|-------------|-------------------|
| 0.85 — 1.00 | ALTA | Afirmación bien respaldada | Actuar con normalidad |
| 0.60 — 0.84 | MEDIA | Afirmación respaldada pero incompleta | Registrar como media, puede actuar |
| 0.30 — 0.59 | BAJA | Afirmación con evidencia insuficiente | Registrar advertencia, buscar más evidencia |
| 0.00 — 0.29 | ESPECULATIVA | Afirmación sin respaldo suficiente | No actuar. Marcar como especulativa. Requiere intervención. |

---

## Manejo de contradicciones

Cuando dos piezas de evidencia se contradicen, el sistema debe:

```yaml
contradiccion:
  tipo: version_framework
  evidencia_a:
    fuente: "pyproject.toml"
    valor: "flask==3.0.0"
  evidencia_b:
    fuente: "app/main.py"
    valor: "from fastapi import FastAPI"
  resolucion:
    - "Verificar si es un proyecto híbrido (Flask + FastAPI)"
    - "Buscar más evidencia (imports en otros archivos)"
    - "Si no se resuelve, registrar como contradicción y bajar confidence"
    - "Confianza resultante: 0.40 (clasificación BAJA)"
```

---

## Evidencia por dominio

Cada dominio tiene tipos de evidencia específicos que se consideran válidos:

| Dominio | Evidencia válida | Peso |
|---------|------------------|------|
| business | README.md, ROADMAP.md, BUSINESS.md | 0.30 |
| functional | manifests, APIs, implementations, tests | 0.40 |
| technical | ARCHITECTURE.md, ADR, design docs | 0.35 |
| infrastructure | Dockerfile, docker-compose, CI/CD config | 0.35 |
| documentation | README, CONTRIBUTING, CHANGELOG, docs/ | 0.25 |
| security | .env.example, SECURITY.md, auth config | 0.30 |
| deployment | deploy scripts, release notes, VERSION | 0.30 |
| observability | metrics, logs, health checks | 0.30 |
| ai | knowledge/patterns, prompt coverage | 0.35 |
| knowledge | patterns/, anti_patterns/, index | 0.30 |
| governance | SDOS/*, STATE.md, GRAPH.md, REGISTRY.yaml | 0.40 |

---

## Almacenamiento de evidencia

La evidencia de Discovery se almacena en `SDOS/discovery/DISCOVERY_EVIDENCE.json`:

```json
{
  "discovery_id": "disc_001",
  "timestamp": "2026-07-15T10:00:20Z",
  "modo": "full",
  "afirmaciones": [
    {
      "afirmacion": "El proyecto usa FastAPI como framework web",
      "confidence": 0.78,
      "hallucination_risk": 0.22,
      "clasificacion": "MEDIA",
      "evidencia": [...]
    }
  ],
  "metricas_globales": {
    "total_afirmaciones": 13,
    "confirmadas_alta": 8,
    "confirmadas_media": 3,
    "confirmadas_baja": 1,
    "especulativas": 1,
    "confidence_promedio": 0.84
  }
}
```

---

## Reglas de evidencia

1. **Toda afirmación debe tener al menos 1 pieza de evidencia.** Si no hay evidencia, la afirmación no existe.
2. **La evidencia debe ser del proyecto.** No se acepta evidencia externa (documentación de librerías, Stack Overflow, etc.).
3. **La evidencia debe ser falsable.** Debe ser posible encontrar evidencia que contradiga la afirmación.
4. **La confianza es por afirmación, no por Engine.** Un mismo Engine puede producir afirmaciones con diferentes niveles de confianza.
5. **Confianza baja ≠ afirmación incorrecta.** Una afirmación con confianza baja es simplemente no verificable. Puede ser correcta pero no demostrable.
6. **Las afirmaciones especulativas se registran pero no se usan** para tomar decisiones en Engines posteriores sin revisión explícita.
