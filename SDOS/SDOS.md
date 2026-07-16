# SDOS — Stone Development Operating System

---

## ¿Qué es SDOS?

SDOS (Stone Development Operating System) es el **kernel del sistema SRIE**. Define el ciclo operativo global que gobierna toda iteración de ingeniería dentro de un proyecto gobernado.

SDOS no es:

- Una metodología ágil (Scrum, Kanban)
- Un pipeline de CI/CD
- Un flujo de trabajo de git
- Un conjunto de scripts

SDOS es el **sistema operativo del proyecto**. Como un kernel de sistema operativo, coordina recursos (Engines), gestiona estado, ejecuta ciclos y garantiza que cada fase produzca la evidencia necesaria para la siguiente.

---

## El ciclo operativo SDOS

```
┌─────────────────────────────────────────────────────────────┐
│                        SDOS CYCLE                            │
│                                                              │
│   BOOT                                                        │
│     │                                                         │
│     ▼                                                         │
│   DISCOVERY                                                   │
│     │                                                         │
│     ▼                                                         │
│   INDICATORS                                                  │
│     │                                                         │
│     ▼                                                         │
│   DX ───────────────────┐                                     │
│     │                    │                                     │
│     ▼                    │                                     │
│   PLAN                   │   Loop de mejora                    │
│     │                    │   (iteraciones internas             │
│     ▼                    │    sin volver a BOOT)               │
│   BUILD                                                       │
│     │                                                         │
│     ▼                                                         │
│   VERIFY                                                      │
│     │                                                         │
│     ▼                                                         │
│   DEPLOY                                                      │
│     │                                                         │
│     ▼                                                         │
│   LEARN                                                       │
│     │                                                         │
│     ▼                                                         │
│   REPEAT ────────────────────────────────────────────────────┘│
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Fases del ciclo

### FASE 0: BOOT

**Objetivo:** Inicializar el sistema y determinar el punto de partida.

| Elemento | Descripción |
|----------|-------------|
| Entrada | Comando `srie init` o `srie boot` |
| Salida | Estado del sistema: proyecto nuevo o existente |
| Reglas | No modifica archivos. Solo descubre. Si no existe SDOS, ejecuta INIT. |
| Engine responsable | INIT |

**Diagrama de decisión:**

```
¿Existe SDOS/ en el proyecto?
│
├── NO → Ejecutar INIT (crear estructura SDOS)
│
└── SÍ → ¿Está completo el ciclo anterior?
          │
          ├── NO → Repair Engine (SIDC)
          │
          └── SÍ → Discovery Engine
```

---

### FASE 1: DISCOVERY

**Objetivo:** Escanear el proyecto y generar un inventario completo de su estado actual.

| Elemento | Descripción |
|----------|-------------|
| Entrada | Proyecto (sistema de archivos) |
| Salida | `PROJECT_STATE.md` — inventario completo |
| Reglas | No modifica archivos. Solo lee. Debe ser ejecutable en cualquier momento. |
| Engine responsable | Discovery Engine |

**Qué descubre:**

- Lenguaje(s) y versión(es)
- Framework(s) y dependencias principales
- Arquitectura detectada (capas, módulos, patrones)
- Documentación existente (README, Architecture, Memory, etc.)
- Estructura de ramas git
- Tests (framework, cobertura si detectable)
- Configuración de CI/CD
- Docker y contenedores
- Variables de entorno y secretos detectables
- Diseño y modelos de datos
- Backlog o issues

---

### FASE 2: INDICATORS

**Objetivo:** Medir la madurez del proyecto en todos los dominios definidos.

| Elemento | Descripción |
|----------|-------------|
| Entrada | `PROJECT_STATE.md` |
| Salida | `SRIE_REPORT.md` — puntuaciones por dominio y nivel de madurez |
| Reglas | Todas las métricas deben ser deterministas. Misma entrada = misma salida. |
| Engine responsable | Indicators Engine |

**Salida típica:**

```json
{
  "srie_score": 74.3,
  "nivel_madurez": "L2",
  "dominios": {
    "gobernanza": 95,
    "desarrollo": 82,
    "arquitectura": 71,
    "memoria": 45,
    "seguridad": 60,
    "operacion": 100,
    "ux": 55,
    "ia": 67
  },
  "fecha": "2026-07-15T10:00:00Z"
}
```

---

### FASE 3: DX

**Objetivo:** Evaluar la experiencia de desarrollo e identificar mejoras prioritarias.

| Elemento | Descripción |
|----------|-------------|
| Entrada | `PROJECT_STATE.md`, `SRIE_REPORT.md` |
| Salida | `DX_REPORT.md` — 5 categorías de diagnóstico |
| Reglas | Solo identifica. No ejecuta cambios. Las 5 preguntas deben responderse siempre. |
| Engine responsable | DX Engine |

**Las 5 preguntas del DX:**

| Pregunta | Categoría | Significado |
|----------|-----------|-------------|
| ¿Qué sirve? | KEEP | Prácticas que funcionan y deben preservarse |
| ¿Qué puede simplificarse? | SIMPLIFY | Procesos o herramientas que son más complejas de lo necesario |
| ¿Qué sobra? | REMOVE | Elementos que no aportan valor |
| ¿Qué falta? | CREATE | Capacidades necesarias que no existen |
| ¿Qué impide escalar? | BLOCKERS | Cuellos de botella que limitan el crecimiento |

---

### FASE 4: PLAN

**Objetivo:** Generar un plan de evolución del proyecto alineado con el diagnóstico DX y los objetivos de negocio.

| Elemento | Descripción |
|----------|-------------|
| Entrada | `DX_REPORT.md`, `SRIE_REPORT.md`, Roadmap (si existe) |
| Salida | `PLAN.md` — plan de evolución priorizado |
| Reglas | No genera tareas de código. Genera objetivos de evolución. Cada objetivo debe tener un indicador de éxito medible. |
| Engine responsable | Planner Engine |

**Estructura de PLAN.md:**

1. Objetivos de la iteración (máximo 3)
2. Capacidades a construir o modificar
3. Priorización (impacto × esfuerzo)
4. Indicadores de éxito por objetivo
5. Dependencias entre capacidades
6. Riesgos identificados

---

### FASE 5: BUILD

**Objetivo:** Construir las capacidades definidas en el plan.

| Elemento | Descripción |
|----------|-------------|
| Entrada | `PLAN.md`, capacidades existentes |
| Salida | Capacidades implementadas o modificadas |
| Reglas | Cada capability debe generarse como un microproyecto autocontenido. No se mezclan capacidades en el mismo cambio. |
| Engine responsable | Capability Engine |

**Estructura de una capability:**

```
capabilities/<nombre>/
├── <nombre>.md              — Especificación
├── <nombre>_design.md       — Diseño técnico
├── <nombre>_api.md          — Contrato de API
├── <nombre>_tests.md        — Tests
├── <nombre>_metrics.md      — Métricas
├── <nombre>_memory.md       — Decisiones de implementación
└── <nombre>_release.md      — Notas de release
```

---

### FASE 6: VERIFY

**Objetivo:** Validar que las capacidades construidas cumplen los requisitos y no introducen regresiones.

| Elemento | Descripción |
|----------|-------------|
| Entrada | Capacidad empaquetada, tests, proyecto actual |
| Salida | `VERIFICATION_REPORT.md` — resultados de validación |
| Reglas | La verificación ocurre en un Sandbox. Nunca en el proyecto principal. |
| Engine responsable | Sandbox Engine |

**Validaciones del Sandbox:**

1. Tests unitarios y de integración
2. Benchmarks de rendimiento
3. Análisis de seguridad
4. Pruebas de estrés
5. Impacto en métricas existentes
6. Cobertura de código

**Resultado:**

```
READY   → La capability puede desplegarse
REJECTED→ La capability requiere cambios
```

---

### FASE 7: DEPLOY

**Objetivo:** Integrar la capability validada al proyecto principal.

| Elemento | Descripción |
|----------|-------------|
| Entrada | Capacidad validada, `VERIFICATION_REPORT.md` |
| Salida | Proyecto actualizado, registro de deploy |
| Reglas | Solo se despliega si VERIFY = READY. Cada deploy debe ser reversible. |
| Engine responsable | Deployment Engine |

---

### FASE 8: LEARN

**Objetivo:** Extraer lecciones de la iteración y consolidar conocimiento.

| Elemento | Descripción |
|----------|-------------|
| Entrada | Toda la evidencia generada en la iteración |
| Salida | Patrones/antipatrones actualizados, memoria actualizada |
| Reglas | Cada iteración debe dejar al menos un patrón o antipatrón nuevo. Si no hay aprendizaje, la iteración no está completa. |
| Engine responsable | Memory Engine, Knowledge Engine |

**Qué se consolida:**

- Decisiones de ingeniería → ADR en Memory
- Patrones exitosos → Knowledge/patterns/
- Problemas recurrentes → Knowledge/anti_patterns/
- Métricas de rendimiento → Knowledge/performance_patterns/
- Vulnerabilidades detectadas → Knowledge/security_patterns/

---

### FASE 9: REPEAT

**Objetivo:** Iniciar un nuevo ciclo con el estado actualizado del proyecto.

| Elemento | Descripción |
|----------|-------------|
| Entrada | Proyecto actualizado + memoria consolidada |
| Salida | Nuevo ciclo SDOS |
| Reglas | REPEAT no es un reinicio. Es una iteración que comienza desde un nivel de madurez superior. |

---

## Contrato del ciclo SDOS

Cada fase del ciclo SDOS debe cumplir este contrato mínimo:

```yaml
fase:
  nombre: string                     # Nombre único de la fase
  entrada: path | string             # Archivo(s) o estado que recibe
  salida: path | string              # Archivo(s) o estado que produce
  engine: string                     # Engine responsable
  reglas:
    - string                         # Lista de reglas inviolables
  validacion:
    - string                         # Cómo se valida que la fase se completó
  evidencia_generada:
    - path                           # Archivos de evidencia que produce
```

---

## Gestión de estado

SDOS mantiene el estado del sistema en `SDOS/STATE.md`:

```yaml
ciclo_actual: 3                      # Número de iteración
fase_actual: BUILD                   # Fase en ejecución
ultima_fase_completada: PLAN         # Última fase finalizada
srie_score: 74.3                     # Última medición
nivel_madurez: L2                    # Nivel actual
proyecto_descubierto: true           # ¿Se ejecutó Discovery?
ultima_iteracion: "2026-07-15T10:00:00Z"
```

Este archivo es la fuente de verdad del estado operativo del sistema. Cualquier Engine puede leerlo para determinar desde dónde continuar.
