# SRIE CORE — El ADN del Sistema

---

## ¿Qué es este documento?

SRIE_CORE.md define los **conceptos fundamentales** del sistema. No explica cómo se implementan (eso es ENGINE_CONTRACT.md). No explica el ciclo operativo (eso es SDOS.md). Explica **qué significa cada cosa** en el universo SRIE.

Este documento es la fuente de verdad para toda terminología. Si hay ambigüedad entre dos documentos, SRIE_CORE.md prevalece.

---

## ¿Qué es un Engine?

Un **Engine** es una unidad autónoma de trabajo dentro del sistema SRIE.

Conceptualmente, un Engine:

- Tiene una **responsabilidad única** y bien definida
- Recibe **entradas** con formato y contrato conocidos
- Produce **salidas** con formato y contrato conocidos
- Deja **evidencia** de cada ejecución
- Actualiza la **memoria** del sistema
- Puede ejecutarse **múltiples veces** (es idempotente o registra versiones)
- **Nunca modifica información fuera de su dominio** sin autorización explícita
- Expone un **manifest** (`engine.yaml`) que describe su contrato completo

Un Engine no es:
- Un script suelto
- Una función dentro de un archivo más grande
- Un microservicio (aunque podría implementarse como tal)

Un Engine es la unidad atómica de procesamiento en SRIE. Todo lo que el sistema hace, lo hace a través de un Engine.

---

## ¿Qué es una Capability?

Una **Capability** es una capacidad funcional del sistema. No es código. Es una **entidad de primer orden** dentro del grafo del proyecto.

Una Capability puede tener:

| Elemento | Descripción |
|----------|-------------|
| Diseño | Especificación de qué hace y por qué |
| Arquitectura | Cómo se estructura internamente |
| Implementación | Código que la realiza |
| Pruebas | Tests que validan su comportamiento |
| Logs | Registro de su ejecución |
| Memoria | Decisiones tomadas durante su construcción |
| Versiones | Historial de cambios |
| Bugs | Problemas conocidos y resueltos |
| Métricas | Indicadores de rendimiento y calidad |
| Deploy | Cómo se integra al proyecto |
| Rollback | Cómo se revierte si falla |

Cada Capability vive en su propio directorio dentro de `capabilities/` y expone un **manifest** (`capability.manifest`) que describe su estructura completa.

Una Capability no es:
- Una función o clase
- Un módulo de Python/JS/Go
- Un archivo suelto

Una Capability es un **microproyecto autocontenido** que puede construirse, probarse, desplegarse y revertirse de forma independiente.

---

## ¿Qué significa Evidencia?

**Evidencia** es todo archivo generado por un Engine durante su ejecución que documenta:

- Qué se hizo
- Con qué entradas
- Qué reglas se aplicaron
- Qué resultado se obtuvo
- Qué decisiones se tomaron
- Qué métricas se registraron

Principios de la evidencia:

1. **Todo deja evidencia.** Nunca existe un "confía en mí". Si no hay archivo, no ocurrió.
2. **La evidencia es determinista.** Misma entrada + mismas reglas = misma evidencia.
3. **La evidencia es legible.** Debe poder ser leída por humanos y por IA sin pérdida de información.
4. **La evidencia es inmutable.** Una vez generada, no se modifica. Se agrega nueva evidencia.
5. **La evidencia es trazable.** Cada archivo de evidencia sabe qué Engine lo generó y en qué iteración.

---

## ¿Qué significa Memoria?

**Memoria** no es un resumen de conversación. Es **conocimiento reutilizable** estructurado en archivos con formato y contrato definidos.

La Memoria en SRIE incluye:

| Tipo | Contenido | Formato |
|------|-----------|---------|
| ADR | Decisiones de ingeniería con contexto, alternativas y justificación | `memory/ADR/<fecha>-<titulo>.md` |
| Contexto | Estado del proyecto en cada iteración | `memory/context/<iteracion>.md` |
| Evolución | Cambios significativos en la arquitectura o el diseño | `memory/evolution.md` |
| Índice | Mapa de toda la memoria disponible | `memory/index.md` |

Principios de la Memoria:

1. **No es narrativa.** No cuenta una historia. Registra hechos y decisiones.
2. **Es estructurada.** Sigue formatos predecibles que la IA puede parsear.
3. **Es acumulativa.** Nunca se borra memoria. Se agrega.
4. **Es referenciable.** Cada entrada tiene un identificador único.
5. **Es consultable.** La IA puede leerla sin necesidad de conversación.

---

## ¿Qué significa Gobernanza?

**Gobernanza** es el conjunto de reglas que determinan:

- **Quién** puede modificar **qué**
- **Quién** no puede modificar **qué**
- **Qué** evidencia se requiere antes de modificar algo
- **Qué** validaciones deben pasar las modificaciones
- **Cómo** se auditan las modificaciones

En SRIE, la gobernanza no es opcional ni gradual. Para cada nivel de madurez, la gobernanza tiene requisitos específicos:

| Nivel | Gobernanza mínima |
|-------|-------------------|
| L0 | No hay gobernanza. Solo descubrimiento. |
| L1 | Los archivos de documentación no se modifican sin registro de cambio. |
| L2 | Los archivos de contrato requieren validación post-cambio. |
| L3 | Las capabilities requieren autorización del Engine correspondiente. |
| L4 | Toda modificación requiere evidencia previa y validación posterior. |
| L5 | El sistema puede proponer cambios pero requiere validación humana. |

---

## ¿Qué significa Estado?

El **Estado** del proyecto es la combinación de:

1. **Nivel de madurez** (L0 a L5)
2. **Fase actual del ciclo SDOS** (qué se está ejecutando ahora)
3. **Iteración actual** (número de ciclo)
4. **Última evidencia generada** (timestamp y tipo)

Los estados posibles de un proyecto gobernado por SRIE son:

| Estado | Significado |
|--------|-------------|
| **Descubierto** | El proyecto ha pasado por INIT. Existe PROJECT_STATE.md. |
| **Gobernado** | El proyecto tiene SDOS activo y contratos definidos. |
| **Verificable** | El proyecto puede ejecutar tests y validaciones de forma reproducible. |
| **Desplegable** | El proyecto puede integrar cambios y desplegarse. |
| **Maduro** | El proyecto ha completado al menos un ciclo SDOS completo. |

El estado se almacena en `SDOS/STATE.md` y es la fuente de verdad para que cualquier Engine sepa desde dónde continuar.

---

## ¿Qué es el Knowledge Graph?

El **Knowledge Graph** no es un archivo markdown. Es un **mapa de nodos y relaciones** que representa la estructura del proyecto.

Nodos del grafo:

```
PROJECT
├── Business
├── Architecture
├── Capabilities (n nodos, uno por capability)
├── Memory
├── Indicators
├── Deployment
├── Design
├── Security
└── SDOS (estado del sistema)
```

Cada nodo sabe:

- **Quién depende de él** (relaciones entrantes)
- **De quién depende** (relaciones salientes)
- **Qué archivos lo representan**
- **Cuándo fue actualizado por última vez**

Cuando un nodo cambia, todos los nodos relacionados reciben una **notificación de cambio**. Esto evita inconsistencias: si cambia Architecture, Capabilities sabe que debe revisar sus contratos.

El Knowledge Graph se materializa en `SDOS/GRAPH.md` y es la estructura que INIT construye en lugar de simplemente crear archivos sueltos.

---

## ¿Qué es un Manifest?

Un **Manifest** es un archivo YAML que describe el contrato completo de un componente del sistema SRIE.

Tipos de manifest:

| Manifest | Describe | Archivo |
|----------|----------|---------|
| Engine Manifest | Contrato de un Engine | `engine.yaml` dentro del directorio del Engine |
| Capability Manifest | Estructura de una Capability | `capability.manifest` dentro de la Capability |
| Project Manifest | Contrato del proyecto | `SDOS/PROJECT.yaml` |

Un manifest siempre responde estas preguntas:

1. **¿Qué es?** — Nombre, versión, responsabilidad
2. **¿Qué necesita?** — Entradas, archivos requeridos, dependencias
3. **¿Qué produce?** — Salidas, archivos generados
4. **¿Qué reglas sigue?** — Contrato de comportamiento
5. **¿Qué no debe hacer?** — Prohibiciones explícitas
6. **¿Cómo se valida?** — Criterios de éxito
7. **¿Cuánto tarda?** — Tiempo esperado de ejecución
8. **¿Qué nivel requiere?** — Nivel SRIE mínimo para ejecutarse

---

## Relaciones entre conceptos

```
SRIE CORE
    │
    ├── Engine (unidad de trabajo)
    │     ├── Tiene Manifest
    │     ├── Sigue ENGINE_CONTRACT
    │     ├── Produce Evidencia
    │     └── Actualiza Memoria
    │
    ├── Capability (unidad funcional)
    │     ├── Tiene Manifest
    │     ├── Puede ser construida por un Engine
    │     ├── Tiene su propia Memoria
    │     └── Tiene sus propias Métricas
    │
    ├── Evidencia (registro de ejecución)
    │     ├── Es generada por Engines
    │     ├── Es inmutable
    │     └── Alimenta la Memoria
    │
    ├── Memoria (conocimiento estructurado)
    │     ├── Incluye ADR, Contexto, Evolución
    │     ├── Es consultable por IA
    │     └── Crece con cada iteración
    │
    ├── Gobernanza (reglas de control)
    │     ├── Define permisos
    │     ├── Define validaciones
    │     └── Escala con el nivel de madurez
    │
    ├── Estado (posición actual del proyecto)
    │     ├── Se almacena en STATE.md
    │     └── Determina desde dónde continuar
    │
    └── Knowledge Graph (mapa de relaciones)
          ├── Conecta todos los nodos
          ├── Propaga cambios
          └── Es la estructura del proyecto
```
