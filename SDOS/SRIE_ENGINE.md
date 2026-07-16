# SRIE ENGINE — Constitución y Filosofía del Sistema

---

## Preámbulo

SRIE (Sistema de Recomendación e Ingeniería Empresarial) es un **Sistema Operativo de Desarrollo** — un meta-sistema diseñado para gobernar, medir y evolucionar proyectos de software asistidos por inteligencia artificial.

SRIE no es:

- Un framework de prompts
- Un conjunto de scripts
- Un plugin para asistentes de código
- Una metodología ágil

SRIE es un **estándar de ingeniería asistida por IA** donde cada decisión se toma desde evidencia almacenada en archivos, no desde el contexto efímero de una conversación.

---

## La hipótesis central

El problema fundamental de la IA aplicada al desarrollo de software no es la alucinación.

El problema es que **la IA no posee un estado interno estable del proyecto**.

Cada vez que recibe un prompt, reconstruye el contexto desde cero. Como no hay una memoria persistente y estructurada, inventa respuestas inconsistentes con decisiones anteriores.

**SRIE reemplaza memoria por evidencia.**

Toda decisión debe originarse en archivos vivos — documentos, contratos, indicadores, patrones — no en el historial de una conversación. La IA no "recuerda": lee.

---

## Filosofía del sistema

Tres pilares sostienen SRIE:

### 1. Gobierno por archivos

El estado del proyecto no está en la mente del desarrollador ni en el contexto del LLM. Está en archivos con formato y ubicación definidos. Si un archivo no existe, esa dimensión del proyecto no está gobernada.

### 2. Madurez progresiva

Un proyecto no se construye de una vez. Atraviesa niveles de madurez (L0 a L5). Cada nivel añade capacidades de gobierno, medición y automatización. No se puede saltar de L0 a L3 sin pasar por L1 y L2.

### 3. Contratos explícitos

Cada componente del sistema (Engines, capacidades, ciclos) expone un contrato formal: qué recibe, qué produce, qué reglas sigue, qué evidencia genera. La interoperabilidad no depende de convenciones implícitas sino de contratos validables.

---

## Principios fundacionales

1. **Confianza trazable** — Ningún Engine puede afirmar algo que no pueda demostrar con evidencia registrada y una puntuación explícita de confianza. Toda afirmación debe incluir su confidence score, evidence count y hallucination risk.

2. **Evidencia sobre memoria** — Ninguna decisión de ingeniería se basa en lo que "recuerda" la conversación. Toda decisión se respalda con un archivo.

3. **Especificación antes que implementación** — Primero se define qué debe hacer el sistema (`.md`). Luego se implementa (código). El código es intérprete de la especificación, no la especificación misma.

4. **Ciclo cerrado** — Toda iteración del sistema termina con una fase de aprendizaje que alimenta la siguiente iteración. No hay iteraciones huérfanas.

5. **Contrato sobre convención** — La comunicación entre componentes sigue contratos explícitos y validables, no convenciones implícitas.

6. **Madurez no negociable** — Un proyecto no puede avanzar de nivel de madurez sin cumplir todos los indicadores del nivel actual.

7. **Cada capacidad es un microproyecto** — Una capability no es código suelto. Es un paquete con diseño, API, tests, logs, memoria y métricas.

8. **El boot es observación, no creación** — `srie init` no genera código. Descubre, diagnostica y documenta el estado actual.

9. **El sandbox es la puerta de producción** — Ningún cambio llega a producción sin pasar por Sandbox, que valida impacto, rendimiento y riesgo.

10. **La memoria del sistema crece con cada iteración** — Toda ejecución deja evidencia. Esa evidencia se consolida en patrones y antipatrones reutilizables.

11. **El sistema es autoevolutivo en L5** — En su nivel máximo de madurez, el sistema es capaz de proponer y ejecutar mejoras sobre sí mismo.

---

## Definiciones del universo SRIE

### Proyecto Gobernado
Un proyecto del que existe evidencia estructurada de su estado, decisiones, arquitectura y evolución. No es un proyecto "documentado" en sentido tradicional: es un proyecto cuyos archivos son ejecutables por el sistema.

### Capability
Un microproyecto autocontenido que resuelve una necesidad específica del sistema. Incluye diseño, implementación, tests, memoria y métricas. Una capability no es una función ni un módulo: es una entidad de primer orden dentro del grafo del proyecto.

### Evidence
Cualquier archivo generado por un Engine durante su ejecución que documenta una decisión, un resultado o una medición. La evidencia es el único insumo que el sistema acepta como fuente de verdad. Toda evidencia debe incluir un confidence score que mida la certeza de la afirmación basada en la cantidad y calidad de las fuentes que la respaldan.

### Confidence
Medida de certeza (0.0 — 1.0) que acompaña a toda afirmación generada por un Engine. Se calcula a partir de: cantidad de evidencia que respalda la afirmación, calidad de las fuentes, consistencia interna y ausencia de contradicciones. Un confidence score bajo no invalida la afirmación: la hace auditable. Cualquier Engine puede actuar sobre información con confianza baja, pero debe registrarlo explícitamente.

### Hallucination Risk
Métrica inversa a Confidence (1.0 — Confidence). Mide el riesgo de que una afirmación sea inventada o incorrecta. Un riesgo alto obliga al Engine a buscar más evidencia antes de actuar.

### Memory
No es un archivo de texto libre. Memory es un sistema estructurado de archivos que registra decisiones de ingeniería (ADR), patrones, antipatrones, contexto y evolución del proyecto. Memory tiene formato y contrato.

### Iteración
Una ejecución completa del ciclo SDOS: desde BOOT hasta LEARN. Cada iteración tiene un objetivo medible, produce evidencia y deja el proyecto en un estado de madurez igual o superior al inicial.

### Madurez (nivel SRIE)
Escala de 6 niveles (L0 a L5) adaptada del modelo CMMI que mide qué tan gobernado, reproducible y autoevolutivo es un proyecto. Cada nivel tiene indicadores específicos.

### Engine
Un procesador especializado dentro del sistema SRIE que recibe una entrada, ejecuta una transformación, verifica el resultado, genera evidencia y actualiza la memoria del sistema. Todos los Engines siguen el mismo contrato.

### SDOS
Stone Development Operating System — el kernel del sistema. Define el ciclo operativo que todos los Engines siguen. Es el orquestador del flujo de trabajo.

### SRIE_SCORE
Métrica compuesta (0-100) que agrega los indicadores de los 8 dominios de madurez. Es el "termómetro" del proyecto.

### Knowledge
Repositorio estructurado de patrones y antipatrones acumulados durante la historia del proyecto. No es documentación: es experiencia destilada en formato reutilizable por Engines.

---

## Mapa conceptual del sistema

```
                    ┌─────────────────────┐
                    │     SRIE ENGINE      │
                    │   (Meta-sistema)     │
                    └──────────┬──────────┘
                               │
                    ┌──────────▼──────────┐
                    │        SDOS          │
                    │   (Kernel — ciclo)   │
                    └──────────┬──────────┘
                               │
           ┌───────────────────┼───────────────────┐
           │                   │                   │
    ┌──────▼──────┐   ┌───────▼───────┐   ┌──────▼──────┐
    │   Engines   │   │  Indicators   │   │    INIT     │
    │ (11 motores)│   │  (L0 a L5)    │   │   (Boot)    │
    └──────┬──────┘   └───────────────┘   └──────┬──────┘
           │                                      │
           └──────────────────┬───────────────────┘
                              │
                    ┌─────────▼──────────┐
                    │   Stonelytics       │
                    │   (Implementación)  │
                    └────────────────────┘
```

---

## Relación con implementaciones concretas

SRIE es un meta-sistema. Define cómo se gobiernan los proyectos, no qué proyectos se construyen.

Stonelytics Studio es una **implementación concreta** de SRIE — una instancia del sistema operando sobre un dominio específico (ingeniería empresarial).

Cualquier otro proyecto que adopte SRIE como estándar de gobierno será también una implementación concreta. La especificación (estos documentos) es la fuente de verdad compartida.
