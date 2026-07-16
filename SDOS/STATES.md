# STATES — Estados Operativos del Sistema SRIE

---

## ¿Qué es este documento?

STATES.md define **todos los estados operativos** del sistema SRIE. No confundir con los niveles de madurez (L0-L5). Los niveles de madurez miden qué tan gobernado está el proyecto. Los estados operativos definen **en qué punto del ciclo de vida** se encuentra una entidad.

Estados aplican a: proyectos, capacidades, engines, artefactos y nodos del grafo.

---

## Estados de un proyecto

```
UNKNOWN
    │
    ▼
DISCOVERED ──────────── El proyecto ha sido escaneado (srie inspect)
    │
    ▼
ANALYZED ────────────── El proyecto ha sido analizado (srie analyze)
    │
    ▼
BOOTSTRAPPED ────────── SDOS ha sido creado (srie bootstrap)
    │
    ▼
ACTIVE ──────────────── El ciclo SDOS se está ejecutando
    │
    ├──► IDLE ───────── Esperando nueva iteración
    │
    ├──► REPAIR ─────── En proceso de reparación
    │
    └──► EVOLVING ───── En proceso de evolución automática (L5)
    │
    ▼
STABLE ──────────────── Ciclo completado. Proyecto en estado consistente.
    │
    ▼
ARCHIVED ────────────── Proyecto cerrado. Solo lectura.
```

### Definiciones

| Estado | Descripción | ¿Permite modificaciones? |
|--------|-------------|--------------------------|
| `UNKNOWN` | El proyecto no ha sido inspeccionado. No existe SDOS. | Sí, pero no gobernadas |
| `DISCOVERED` | `srie inspect` ejecutado. Se conoce la estructura básica. | Sí, pero no gobernadas |
| `ANALYZED` | `srie analyze` ejecutado. Se conocen los indicadores. | Sí, pero no gobernadas |
| `BOOTSTRAPPED` | `srie bootstrap` ejecutado. SDOS creado. | Sí, gobernadas por SDOS |
| `ACTIVE` | Ciclo SDOS en ejecución. Una o más fases activas. | Solo por el Engine en curso |
| `IDLE` | SDOS presente. No hay ciclo en ejecución. | Sí, gobernadas por SDOS |
| `REPAIR` | Repair Engine trabajando en una desviación. | Solo por Repair Engine |
| `EVOLVING` | El sistema está evolucionando automáticamente (solo L5). | Solo por Evolution Engine |
| `STABLE` | Último ciclo completado sin errores. Evidencia consolidada. | Sí, gobernadas por SDOS |
| `ARCHIVED` | Proyecto cerrado. No se ejecutan ciclos. | Solo lectura |

### Diagrama de transiciones

```
UNKNOWN ──► DISCOVERED ──► ANALYZED ──► BOOTSTRAPPED ──► ACTIVE ──► STABLE
  │            │              │               │              │
  └────────────┴──────────────┴───────────────┴──────► IDLE ──┘
                                                         │
                                                         ▼
                                                      REPAIR
                                                         │
                                                         ▼
                                                      ACTIVE
                                                         │
                                                         ▼
                                                      STABLE
                                                         │
                                                    ARCHIVED
```

### Reglas de transición

| Transición | Requisitos |
|------------|------------|
| UNKNOWN → DISCOVERED | `srie inspect` ejecutado exitosamente |
| DISCOVERED → ANALYZED | `srie analyze` ejecutado exitosamente |
| ANALYZED → BOOTSTRAPPED | `srie bootstrap` ejecutado exitosamente |
| BOOTSTRAPPED → ACTIVE | `srie discover` (Discovery Engine) invocado |
| ACTIVE → STABLE | Ciclo SDOS completo + Governance Engine OK |
| STABLE → IDLE | Nueva iteración no iniciada |
| IDLE → ACTIVE | Nueva iteración iniciada por SDOS |
| ACTIVE → REPAIR | Fase del ciclo falla. Repair Engine invocado. |
| REPAIR → ACTIVE | Reparación completada. Ciclo reanudado. |
| STABLE → ARCHIVED | Decisión explícita del proyecto |

---

## Estados de una Capability

```
DRAFT ──────────── En diseño, no implementada
    │
    ▼
BUILDING ───────── En construcción por Capability Engine
    │
    ▼
TESTING ────────── En verificación por Sandbox Engine
    │
    ├──► REJECTED ─ No pasó verificación. Requiere cambios.
    │
    ▼
STABLE ─────────── Implementada y verificada
    │
    ├──► DEPLOYED ─ Integrada al proyecto principal
    │
    ├──► UPDATING ─ En proceso de modificación
    │
    ├──► DEPRECATED ─ En desuso
    │       │
    │       ▼
    │   ARCHIVED ─ Ya no se mantiene
    │
    └──► BROKEN ─── Rotura detectada
            │
            ▼
        REPAIR ──── En reparación
```

### Definiciones

| Estado | Descripción |
|--------|-------------|
| `DRAFT` | Capability propuesta, en diseño. Solo existe el manifest. |
| `BUILDING` | Capability Engine está construyendo los archivos. |
| `TESTING` | Sandbox Engine está ejecutando validaciones. |
| `REJECTED` | No pasó verificación. Requiere cambios. |
| `STABLE` | Implementada y verificada. Lista para deploy. |
| `DEPLOYED` | Integrada al proyecto principal. |
| `UPDATING` | En proceso de modificación por una nueva iteración. |
| `DEPRECATED` | En desuso. No se recomienda para nuevas implementaciones. |
| `ARCHIVED` | Ya no se mantiene. Solo lectura. |
| `BROKEN` | Se detectó una rotura (bug, incompatibilidad). |
| `REPAIR` | Repair Engine trabajando en la corrección. |

### Reglas de transición

| Transición | Requisitos |
|------------|------------|
| DRAFT → BUILDING | Plan la incluye. Capability Engine invocado. |
| BUILDING → TESTING | Todos los archivos obligatorios existen. |
| TESTING → STABLE | Todas las validaciones pasan. |
| TESTING → REJECTED | Alguna validación crítica falla. |
| REJECTED → BUILDING | Se corrigen los problemas identificados. |
| STABLE → DEPLOYED | Deployment Engine ejecutado exitosamente. |
| DEPLOYED → UPDATING | Nueva versión planificada. |
| UPDATING → TESTING | Cambios completados. |
| STABLE → DEPRECATED | Decisión explícita. Alternativa recomendada. |
| DEPRECATED → ARCHIVED | 2 ciclos sin uso. Sin dependientes activos. |
| DEPLOYED → BROKEN | Bug detectado en producción. |
| BROKEN → REPAIR | Repair Engine invocado. |
| REPAIR → STABLE | Bug corregido y verificado. |

---

## Estados de un Engine

```
REGISTERED ─────── Manifest registrado en SDOS
    │
    ▼
IDLE ───────────── Esperando ser invocado
    │
    ▼
RUNNING ────────── Ejecutando el ciclo universal
    │
    ├──► SUCCESS ── Ciclo completado exitosamente
    │
    ├──► FAILED ─── Ciclo falló (error recuperable)
    │
    └──► ERROR ──── Ciclo falló (error no recuperable)
    │
    ▼
IDLE ───────────── Listo para próxima invocación
```

### Definiciones

| Estado | Descripción |
|--------|-------------|
| `REGISTERED` | Engine registrado con su manifest. No ha sido invocado. |
| `IDLE` | Disponible para ser invocado. |
| `RUNNING` | Ejecutando alguna etapa del ciclo universal. |
| `SUCCESS` | Ciclo completado con código 0. |
| `FAILED` | Ciclo falló pero es recuperable (reintento). |
| `ERROR` | Ciclo falló de forma no recuperable. |

---

## Estados de un Artefacto

```
PENDING ────────── Pendiente de generación
    │
    ▼
GENERATED ──────── Generado por un Engine
    │
    ▼
VALIDATED ──────── Validado y consistente
    │
    ▼
CURRENT ────────── Versión actual del artefacto
    │
    ▼
SUPERSEDED ─────── Reemplazado por una versión más reciente
    │
    ▼
ARCHIVED ───────── Ya no es relevante
```

---

## Estados de un Nodo del Grafo

```
PENDING ────────── Nodo definido pero sin archivos asociados
    │
    ▼
SYNCED ─────────── Nodo actualizado. Archivos existen y son consistentes.
    │
    ▼
DESYNCED ───────── Nodo desactualizado. Archivos cambiaron pero el grafo no refleja el cambio.
    │
    ▼
STALE ──────────── Nodo sin actualizar por más de 30 días.
    │
    ▼
ORPHAN ─────────── Nodo sin conexiones entrantes ni salientes (excepto PROJECT).
```

---

## Estados compuestos

Un proyecto puede tener múltiples entidades con diferentes estados. El estado global del proyecto se calcula como:

```
Estado global = Mínimo estado de todas las entidades críticas

Donde:
- CRITICAL = PROJECT, SDOS, ARCHITECTURE, INDICATORS
- ORDEN: UNKNOWN < DISCOVERED < ANALYZED < BOOTSTRAPPED < IDLE < ACTIVE < STABLE < ARCHIVED

Ejemplo:
- PROJECT: STABLE
- SDOS: ACTIVE (ciclo en ejecución)
- ARCHITECTURE: SYNCED
- INDICATORS: SYNCED

Estado global = ACTIVE (porque SDOS está en ACTIVE y ACTIVE < STABLE)
```

---

## Registro de estados

Los estados se registran en:

1. `SDOS/STATE.md` — Estado global del proyecto
2. `capabilities/*/*.manifest` — Estado de cada capability
3. `engines/*/engine.yaml` — Estado de cada engine (cuando estén implementados)
4. `SDOS/GRAPH.md` — Estado de cada nodo del grafo
5. `SDOS/events/` — Eventos de cambio de estado

Toda transición de estado debe generar un evento (ver EVENTS.md).
