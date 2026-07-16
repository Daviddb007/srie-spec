# Sprint 1: Documentos Fundacionales de SRIE

**Fecha:** 2026-07-15
**Estado:** Aprobado para implementación

---

## Resumen

Este sprint construye la especificación completa del sistema SRIE (Sistema de Recomendación e Ingeniería Empresarial) en 5 documentos fundacionales. Ninguno contiene código. Todos son especificación pura — la "Constitución" del sistema.

La premisa central: **reemplazar memoria por evidencia**. La IA no debe reconstruir contexto desde la conversación; debe leerlo desde archivos vivos.

---

## Documentos a generar

| Archivo | Rol | Audiencia |
|---------|-----|-----------|
| `SDOS/SRIE_ENGINE.md` | Constitución y filosofía del sistema | Cualquier persona que quiera entender SRIE |
| `SDOS/SDOS.md` | Ciclo operativo global (kernel) | Ingenieros implementando el ciclo |
| `SDOS/ENGINES.md` | Catálogo de motores, responsabilidades, contratos | Ingenieros implementando motores |
| `SDOS/SRIE_INDICATORS.md` | Modelo de madurez L0-L5 y métricas | Evaluadores del estado del proyecto |
| `SDOS/INIT.md` | Especificación del proceso `srie init` | Implementadores del INIT |

---

## Arquitectura conceptual

```
SRIE (meta-sistema)
│
├── SDOS (kernel — ciclo operativo)
│   ├── BOOT → DISCOVERY → INDICATORS → DX → PLAN
│   ├── BUILD → VERIFY → DEPLOY → LEARN → REPEAT
│   └── Contrato unificado de ciclo
│
├── ENGINES (motores especializados)
│   ├── Discovery Engine
│   ├── Indicators Engine
│   ├── DX Engine
│   ├── Planner Engine
│   ├── Capability Engine
│   ├── Sandbox Engine
│   ├── Repair Engine (SIDC)
│   ├── Memory Engine
│   ├── Knowledge Engine
│   ├── Governance Engine
│   └── Deployment Engine
│
├── INDICATORS (modelo de madurez)
│   ├── 8 dominios de indicadores
│   ├── Niveles L0 a L5 (CMMI adaptado)
│   └── SRIE_SCORE compuesto
│
└── INIT (proceso de boot)
    ├── Discovery del proyecto
    ├── Árbol de decisión nuevo/existente
    └── Generación de PROJECT_STATE.md
```

---

## Principios de diseño

1. **Especificación antes que implementación** — los .md son la fuente de verdad; el código es intérprete
2. **Contratos explícitos** — todo Engine sigue Input → Validation → Execution → Verification → Evidence → Memory → Exit
3. **Gobierno por evidencia** — ninguna decisión depende del contexto de conversación
4. **Separación de niveles** — SRIE es meta-sistema; Stonelytics Studio es una implementación concreta
5. **Madurez progresiva** — L0 a L5 como CMMI, no como versiones semánticas

---

## Estructura de cada documento

### SRIE_ENGINE.md
- Preámbulo y declaración fundacional
- La hipótesis: memoria vs evidencia
- Filosofía del sistema (3 pilares)
- 10 principios fundacionales
- Definiciones del universo SRIE (8+ términos)
- Mapa conceptual

### SDOS.md
- ¿Qué es SDOS?
- El ciclo operativo (9 fases)
- Cada fase: objetivo, entrada, salida, reglas
- Contrato del ciclo
- Gestión de estado del sistema
- Integración con Engines

### ENGINES.md
- Contrato unificado de Engine (7 pasos)
- 11 fichas de Engine (una por motor)
- Cada ficha: objetivo, entradas, salidas, reglas, archivos, indicadores, dependencias
- Mapa de dependencias entre Engines
- Glosario de contratos

### SRIE_INDICATORS.md
- Modelo CMMI adaptado: L0 a L5
- 8 dominios: Gobernanza, Desarrollo, Arquitectura, Memoria, Seguridad, Operación, UX, IA
- Métricas detalladas por dominio
- Fórmulas de puntuación (ponderación)
- Interpretación de niveles

### INIT.md
- Propósito y ámbito
- Fase 0: Discovery del proyecto
- Árbol de decisión: ¿proyecto nuevo o existente?
- Generación de PROJECT_STATE.md
- Contrato de salida
- Lo que INIT NUNCA hace

---

## Criterios de éxito

- [ ] Los 5 documentos existen en `SDOS/`
- [ ] No contienen código de implementación (solo especificación)
- [ ] Son autocontenidos: un lector nuevo puede entender SRIE sin contexto externo
- [ ] Son consistentes entre sí: ningún documento contradice a otro
- [ ] Incluyen definiciones explícitas de todos los términos clave
- [ ] Definen contratos lo suficientemente precisos para implementar código
