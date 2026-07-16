# SRIE FOUNDATION — Núcleo Estable del Sistema

---

## ¿Qué es este documento?

La **SRIE Foundation** es el núcleo estable del sistema. Comprende todo lo definido en las fases 0 a 2.8. A partir de ahora, estos documentos entran en **régimen de estabilidad**: no desaparecen, no cambian de forma incompatible sin un proceso explícito.

Este documento define qué pertenece a la Foundation, las reglas de compatibilidad y el proceso para modificarla.

---

## Alcance de la Foundation

```yaml
foundation:
  fase_0:
    - SRIE_ENGINE.md         # Constitución (principios, definiciones)
    - SDOS.md                # Kernel y ciclo operativo
    - ENGINES.md             # Catálogo de motores
    - SRIE_INDICATORS.md     # Modelo de madurez L0-L5
    - INIT.md                # Boot del sistema (4 niveles)

  fase_1:
    - SRIE_CORE.md           # ADN del sistema
    - ENGINE_CONTRACT.md     # Ciclo universal de ejecución
    - CAPABILITY_MANIFEST.md # Contrato de capacidades
    - PROJECT_CONTRACT.md    # Contrato de proyecto

  fase_1.5:
    - GRAPH.md               # Knowledge Graph
    - DOMAINS.md             # Dominios
    - EVENTS.md              # Sistema de eventos
    - STATES.md              # Estados operativos
    - ARTIFACTS.md           # Tipos de artefactos

  fase_1.8:
    - REGISTRY.md            # Catálogo central
    - ONTOLOGY.md            # Ontología del sistema

  fase_2:
    - DISCOVERY_ENGINE.md    # Motor de descubrimiento
    - DISCOVERY_PROTOCOL.md  # Protocolo Observe→Classify→Correlate→Validate→Publish
    - DISCOVERY_EVIDENCE.md  # Evidencia y confianza
    - DISCOVERY_OUTPUTS.md   # Artefactos de Discovery
    - DISCOVERY_PIPELINE.md  # Pipeline incremental

  fase_2.5:
    - FEDERATION.md          # Modelo de federación
    - DOMAIN.md              # Qué es un Domain
    - NETWORK.md             # Red federada
    - INTER_SRIE_CONTRACT.md # Contratos entre Domains
    - BUS.md                 # Sistema de mensajería
    - DISCOVERY_FEDERATION.md# Discovery federado

  fase_2.8:
    - IDENTITY.md            # Sistema de identidad
    - GOVERNANCE.md          # Reglas de gobierno
    - AUTHORITY.md           # Niveles de autoridad
```

---

## Reglas de la Foundation

### Regla 1: Inmutabilidad semántica

Los conceptos definidos en la Foundation no pueden cambiar de significado. Si `SDOS.md` define que el ciclo tiene 9 fases, ninguna modificación puede reducir ese número sin un proceso de revisión de la Foundation.

### Regla 2: Compatibilidad hacia atrás

Todo Engine, Capability o Domain construido sobre la Foundation debe seguir funcionando después de una actualización de la Foundation. Las adiciones son libres. Las modificaciones deben preservar compatibilidad.

### Regla 3: Deprecación, no eliminación

Si un concepto deja de ser necesario, no se elimina. Se depreca con una nota clara y un período de transición mínimo de 3 iteraciones del ciclo SDOS.

### Regla 4: Versionado de la Foundation

```yaml
versionado:
  formato: "MAJOR.MINOR.PATCH"
  MAJOR: "Cambio incompatible en la Foundation"
  MINOR: "Adición compatible hacia atrás"
  PATCH: "Corrección, aclaración, error tipográfico"

  frecuencia_esperada:
    MAJOR: "Cada 12+ meses (evento extraordinario)"
    MINOR: "Cada 1-3 meses"
    PATCH: "Cuando sea necesario"
```

### Regla 5: Proceso de cambio MAJOR

```yaml
cambio_major:
  pasos:
    1. "Propuesta formal de cambio (documento RFC)"
    2. "Revisión por SRIE Core"
    3. "Período de comentarios (14 días)"
    4. "Votación de los Domains de la federación (si existe)"
    5. "Aprobación (≥ 67% de los Domains activos)"
    6. "Plan de migración (mínimo 3 ciclos SDOS)"
    7. "Nueva versión de la Foundation"
    8. "Los Domains existentes eligen cuándo migrar"
```

### Regla 6: Congelamiento

Durante la implementación de un Engine (FASE 3 en adelante), la Foundation no puede modificarse excepto para correcciones PATCH. Cualquier cambio MINOR o MAJOR debe esperar a que el Engine en construcción esté completo.

---

## Foundation Registry

La Foundation se materializa en `SDOS/FOUNDATION.yaml`:

```yaml
# SDOS/FOUNDATION.yaml
foundation:
  version: "1.0.0"
  estado: "STABLE"
  ultima_revision: "2026-07-15"
  documentos: 30
  compatibilidad:
    hacia_atras: true
    motores_implementados: 0
    domains_activos: 0
  changelog:
    - version: "1.0.0"
      fecha: "2026-07-15"
      cambios: ["Versión inicial de la Foundation"]
```
