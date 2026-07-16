# ECOSYSTEM VERSION — Versionado Semántico del Ecosistema SRIE

---

## ¿Qué es este documento?

ECOSYSTEM_VERSION.md define **la política de versionado** de todo el ecosistema SRIE. No es solo semver para un proyecto. Es **versionado coordinado** entre Foundation, Domains, Capabilities, Resources, Ontología y Registry.

---

## ¿Qué se versiona?

```yaml
ecosystem_components:
  foundation:     "1.0.0"   # La Foundation completa
  ontology:       "3"       # La ontología (número entero)
  runtime:        "1.0.0"   # El Runtime de SRIE
  engine_contract:"1.0.0"   # El contrato universal de Engines
  resource_contract:"1.0.0" # El contrato universal de recursos

  cada_domain:    "x.y.z"   # Cada Domain tiene su propia versión
  cada_capability:"x.y.z"   # Cada Capability tiene su propia versión
  cada_recurso:   "x.y.z"   # Cada Recurso tiene su propia versión
  cada_engine:    "x.y.z"   # Cada Engine implementado tiene su versión
```

---

## Reglas de versionado

### Foundation

```yaml
foundation_versioning:
  formato: "MAJOR.MINOR.PATCH"
  MAJOR: "Cambio incompatible (proceso de votación)"
  MINOR: "Adición compatible hacia atrás"
  PATCH: "Corrección, aclaración, error"

  compatibilidad:
    "MAJOR igual": "Compatible"
    "MAJOR diferente": "Incompatible (requiere migración)"
    "MINOR mayor o igual": "Compatible"
    "PATCH diferente": "Compatible (si MAJOR.MINOR igual)"
```

### Ontología

```yaml
ontology_versioning:
  formato: "ENTERO incremental"
  cambio: "Nueva entidad, nueva relación, cambio semántico"
  compatibilidad:
    "Misma versión": "100% compatible"
    "+1": "Adiciones compatibles"
    "+2+": "Puede haber cambios incompatibles"
```

### Domains y Capabilities

```yaml
domain_capability_versioning:
  formato: "MAJOR.MINOR.PATCH (semver estándar)"
  MAJOR: "Breaking change en API o contrato"
  MINOR: "Nueva funcionalidad compatible"
  PATCH: "Bug fix, mejora interna"

  compatibilidad_inter_domain:
    "Domain A (v1.2.0) consume Capability de Domain B (v2.0.0)":
      - "MAJOR diferente → verificar contrato"
      - "Si el contrato sigue siendo compatible → OK"
      - "Si no → requiere actualización de contrato"
```

---

## Matriz de compatibilidad

```yaml
compatibilidad:
  foundation: "1.0"
  runtime: "1.0"
  ontology: "3"

  combinaciones_validas:
    - foundation: "1.0"
      runtime: "1.0"
      ontology: "3"
      estado: "VIGENTE"

    - foundation: "1.0"
      runtime: "1.0"
      ontology: "4"
      estado: "EN_TRANSICIÓN"

    - foundation: "1.0"
      runtime: "2.0"
      ontology: "3"
      estado: "NO_SOPORTADO"
```

---

## Ecosystem Manifest

Cada sistema SRIE tiene `SDOS/ECOSYSTEM_VERSION.yaml`:

```yaml
# SDOS/ECOSYSTEM_VERSION.yaml
ecosystem:
  foundation: "1.0.0"
  runtime: "1.0.0"
  ontology: "3"
  engine_contract: "1.0.0"
  resource_contract: "1.0.0"

  domains:
    - id: "DOM-BACKEND"
      version: "1.2.0"
      foundation_requerida: ">=1.0.0"
      runtime_requerido: ">=1.0.0"

    - id: "DOM-MARKETING"
      version: "1.0.0"
      foundation_requerida: ">=1.0.0"
      runtime_requerido: ">=1.0.0"

  capabilities:
    - id: "CAP-auth"
      version: "2.1.0"
      domain: "DOM-BACKEND"

  resources:
    - id: "RES-GH-001"
      version: "1.0.0"
      contract_version: "1.0.0"

  ultima_actualizacion: "2026-07-15T10:00:00Z"
```
