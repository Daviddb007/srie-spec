# DIAGNOSIS PATTERNS — Patrones de Diagnóstico Reutilizables

---

## ¿Qué es este documento?

DIAGNOSIS_PATTERNS.md define el **sistema de patrones de diagnóstico** de SRIE. Cada vez que el DX Engine resuelve un problema, puede escribir un patrón. Cuando el mismo patrón vuelva a aparecer, el motor lo reconoce sin tener que razonar desde cero.

Los patrones de diagnóstico son el puente entre la experiencia acumulada (Knowledge Engine) y el razonamiento activo (DX Engine).

---

## ¿Qué es un patrón de diagnóstico?

Un patrón de diagnóstico es una **configuración recurrente de indicadores** que corresponde a un problema conocido. Es como un "síndrome" en medicina: un conjunto de síntomas que suelen aparecer juntos y que apuntan a una causa común.

```yaml
patron_diagnostico:
  id: "PAT-DOC-001"
  nombre: "DOCUMENTATION_DECAY"
  dominio: "gobernanza"
  severidad: "media"

  descripcion: |
    El proyecto muestra deterioro progresivo en su documentación.
    Los indicadores de documentación caen consistentemente mientras
    los indicadores de desarrollo se mantienen o mejoran.
    Esto generalmente indica que el equipo prioriza features
    sobre documentación.

  sintomas:
    - indicador: "documentation_health"
      condicion: "valor < 70 AND tendencia = DECLINING"
      peso: 0.35
    - indicador: "architecture_exists"
      condicion: "valor = false"
      peso: 0.25
    - indicador: "memory_freshness"
      condicion: "valor < 50"
      peso: 0.20
    - indicador: "changelog_exists"
      condicion: "valor = false OR ultima_actualizacion > 30d"
      peso: 0.20

  causas_probables:
    - nombre: "Feature pressure"
      probabilidad: 0.60
      evidencia: "Indicadores de desarrollo saludables, documentación decayendo"
    - nombre: "Equipo nuevo"
      probabilidad: 0.25
      evidencia: " commits recientes de nuevos autores, sin ADR"
    - nombre: "Falta de estándares"
      probabilidad: 0.15
      evidencia: "Documentación existe pero inconsistentemente"

  intervencion:
    acciones:
      - "Crear ARCHITECTURE.md (crítico)"
      - "Actualizar memory/index.md (alto)"
      - "Agregar documentación al definition of done (medio)"
      - "Programar sesión de documentación semanal (bajo)"
    impacto_esperado:
      documentation_health: "+15 puntos en 30 días"
    tiempo_estimado: "3-5 días"

  casos_ejemplo:
    - "CASE-003: Proyecto con documentation_health de 82 a 68 en 30 días"
    - "CASE-012: Proyecto sin ARCHITECTURE.md durante 6 meses"

  eventos_relacionados:
    - "DOCUMENTATION_WARNING"
    - "DOCUMENTATION_DEGRADING"
    - "MEMORY_WARNING"
```

---

## Catálogo de patrones

Los patrones se almacenan en `knowledge/diagnosis/<dominio>/<patron>.md`. Esta es la estructura del catálogo inicial:

### Gobernanza

```yaml
patrones_gobernanza:
  - id: "PAT-GOV-001"
    nombre: "GOVERNANCE_GAP"
    descripcion: "Faltan documentos de gobierno fundamentales"
    sintomas:
      - "readme_exists = false"
      - "architecture_exists = false"
      - "memory_exists = false"

  - id: "PAT-GOV-002"
    nombre: "GOVERNANCE_AGING"
    descripcion: "La documentación de gobierno existe pero está desactualizada"
    sintomas:
      - "documentation_health < 70"
      - "memory_freshness < 50"
      - "adr_freshness < 50"

  - id: "PAT-GOV-003"
    nombre: "IDENTITY_EXPIRING"
    descripcion: "La identidad del Domain está por expirar"
    sintomas:
      - "identity_expiration < 30d"
```

### Arquitectura

```yaml
patrones_arquitectura:
  - id: "PAT-ARC-001"
    nombre: "TECH_DEBT_ACCUMULATION"
    descripcion: "La deuda técnica está creciendo sin control"
    sintomas:
      - "tech_debt_risk > 0.50 AND tendencia = INCREASING"
      - "maintenance_risk > 0.40"

  - id: "PAT-ARC-002"
    nombre: "ARCHITECTURE_EROSION"
    descripcion: "La arquitectura documentada no coincide con la implementada"
    sintomas:
      - "architecture_health < 60"
      - "graph_consistency < 0.70"

  - id: "PAT-ARC-003"
    nombre: "SILOED_CAPABILITIES"
    descripcion: "Capabilities sin relaciones en el grafo"
    sintomas:
      - "capability_count > 3"
      - "graph_edges_per_capability < 2"
```

### Seguridad

```yaml
patrones_seguridad:
  - id: "PAT-SEC-001"
    nombre: "SECURITY_GAP"
    descripcion: "Prácticas de seguridad básicas no implementadas"
    sintomas:
      - "env_example_exists = false"
      - "security_health < 50"

  - id: "PAT-SEC-002"
    nombre: "DEPENDENCY_VULNERABILITY"
    descripcion: "Dependencias con vulnerabilidades conocidas"
    sintomas:
      - "dependencies_scanned = true"
      - "vulnerable_dependencies > 0"
```

### Operación

```yaml
patrones_operacion:
  - id: "PAT-OPS-001"
    nombre: "DEPLOY_FRAGILITY"
    descripcion: "El deploy es frágil y propenso a fallos"
    sintomas:
      - "deployment_health < 60"
      - "deployment_risk > 0.50"

  - id: "PAT-OPS-002"
    nombre: "NO_ROLLBACK"
    descripcion: "No existe estrategia de rollback"
    sintomas:
      - "rollback_strategy = false"
      - "deployment_risk > 0.30"
```

### IA

```yaml
patrones_ia:
  - id: "PAT-IA-001"
    nombre: "MEMORY_STAGNATION"
    descripcion: "La memoria del sistema no se está actualizando"
    sintomas:
      - "memory_freshness < 30"
      - "adr_count < 3 en últimos 30 días"
      - "hallucination_risk > 0.20"

  - id: "PAT-IA-002"
    nombre: "KNOWLEDGE_FRAGMENTATION"
    descripcion: "El conocimiento está fragmentado sin patrones"
    sintomas:
      - "knowledge_health < 50"
      - "pattern_count < 5"
      - "knowledge_growth = STABLE"
```

### Organizacional

```yaml
patrones_organizacional:
  - id: "PAT-ORG-001"
    nombre: "DOMAIN_ISOLATION"
    descripcion: "Un Domain en la federación está operando aislado"
    sintomas:
      - "federation_health < 50"
      - "domain_contracts = 0"
      - "domain_heartbeat = IRREGULAR"

  - id: "PAT-ORG-002"
    nombre: "CAPABILITY_DUPLICATION"
    descripcion: "Múltiples Domains tienen la misma capability"
    sintomas:
      - "duplicate_capabilities > 0"
      - "fragmentation_risk > 0.30"
```

---

## Ciclo de vida de un patrón

```yaml
patron_lifecycle:
  IDENTIFIED: "Patrón detectado por primera vez en un diagnóstico"
    │
    ▼
  DRAFT: "Patrón documentado pero no validado"
    │
    ▼
  VALIDATED: "Patrón confirmado por 3+ casos"
    │
    ▼
  STABLE: "Patrón estable y reutilizable"
    │
    ├── DEPRECATED: "Ya no es relevante"
    │
    └── ARCHIVED: "Histórico, no se usa"
```

### Transiciones

```yaml
transiciones:
  IDENTIFIED → DRAFT: "Knowledge Engine documenta el patrón"
  DRAFT → VALIDATED: "3+ diagnósticos coinciden con el patrón"
  VALIDATED → STABLE: "Patrón revisado y aprobado"
  STABLE → DEPRECATED: "El patrón dejó de ser relevante"
  DEPRECATED → ARCHIVED: "6 meses sin uso"
```

---

## Match de patrones

Cuando el Hypothesis Engine busca patrones, calcula un **match score** para cada patrón candidato:

```yaml
match_score:
  formula: |
    Match = Σ(sintoma_coincide × peso_del_sintoma) / Σ(pesos)

  donde:
    sintoma_coincide: 1.0 si la condición se cumple, 0.0 si no
    peso_del_sintoma: peso definido en el patrón

  umbral_minimo: 0.60
  accion_si_supera: "Generar hipótesis por patrón"
  accion_si_no: "No usar el patrón"

  ejemplo:
    patrón: "DOCUMENTATION_DECAY"
    sintomas:
      - documentation_health < 70? → true (peso 0.35)
      - architecture_exists = false? → true (peso 0.25)
      - memory_freshness < 50? → false (peso 0.20)
      - changelog desactualizado? → true (peso 0.20)
    match: (0.35 + 0.25 + 0.0 + 0.20) / 1.0 = 0.80 → SUPERA UMBRAL
```
