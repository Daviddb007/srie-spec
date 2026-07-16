# INDICATOR CATALOG — Catálogo Declarativo de Indicadores

---

## ¿Qué es este documento?

INDICATOR_CATALOG.md define **el catálogo completo de indicadores** que el Indicators Engine puede calcular. Es un archivo **declarativo**: los indicadores no están codificados en el Engine. Están definidos aquí. El Engine sabe cómo medir; el Catálogo define qué medir.

Cada indicador responde:

```
Nombre → ¿Qué mide?
Valor → ¿Cuánto vale ahora?
Evidencia → ¿En qué archivos se basa?
Confianza → ¿Qué certeza tenemos?
Última medición → ¿Cuándo se midió?
Responsable → ¿Qué Engine es dueño de este indicador?
Acción sugerida → ¿Qué hacer si está mal?
Tendencia → ¿Mejora, empeora o estable?
```

---

## Formato de un indicador

```yaml
indicador:
  id: "IND-001"
  nombre: "readme_exists"
  nivel: "observacion"              # observacion | salud | riesgo | predictivo
  dominio: "gobernanza"
  entidad: "project"                # project | domain | capability | engine | federation

  descripcion: "¿Existe README.md en la raíz del proyecto?"

  formula:
    tipo: "binario"                 # binario | continuo | porcentaje | score
    logica: "existe('README.md')"
    rango: [0, 1]
    unidad: "booleano"

  peso_global: 0.02

  fuente:
    tipo: "artifact"
    path: "SDOS/PROJECT_STATE.md"
    campo: "documentacion"

  frecuencia: "por_evento"          # continuo | por_evento | programado

  threshold:
    warning: 0
    critical: null

  eventos_asociados:
    - "DOCUMENTATION_IMPROVING"
    - "DOCUMENTATION_DEGRADING"

  accion_sugerida:
    si_warning: "Crear README.md con propósito y descripción del proyecto"
    si_critical: null

  responsable: "Discovery Engine"
```

---

## Catálogo completo

### Nivel 1: Observación

Indicadores binarios que constatan hechos. No interpretan.

```yaml
indicadores_observacion:
  - id: "IND-001"
    nombre: "readme_exists"
    dominio: "gobernanza"
    formula: "existe('README.md')"
    peso: 0.02

  - id: "IND-002"
    nombre: "architecture_exists"
    dominio: "gobernanza"
    formula: "existe('ARCHITECTURE.md')"
    peso: 0.02

  - id: "IND-003"
    nombre: "changelog_exists"
    dominio: "gobernanza"
    formula: "existe('CHANGELOG.md')"
    peso: 0.02

  - id: "IND-004"
    nombre: "tests_detected"
    dominio: "desarrollo"
    formula: "existe_directorio('tests/') OR existe('pyproject.toml[tool.pytest]')"
    peso: 0.03

  - id: "IND-005"
    nombre: "docker_detected"
    dominio: "infraestructura"
    formula: "existe('Dockerfile') AND existe('docker-compose.yml')"
    peso: 0.02

  - id: "IND-006"
    nombre: "cicd_detected"
    dominio: "infraestructura"
    formula: "existe('.github/') OR existe('.gitlab-ci.yml') OR existe('Jenkinsfile')"
    peso: 0.02

  - id: "IND-007"
    nombre: "git_detected"
    dominio: "desarrollo"
    formula: "existe('.git/')"
    peso: 0.01

  - id: "IND-008"
    nombre: "memory_exists"
    dominio: "memoria"
    formula: "existe('memory/index.md')"
    peso: 0.03

  - id: "IND-009"
    nombre: "adr_exists"
    dominio: "memoria"
    formula: "existe('memory/ADR/') AND count('memory/ADR/*.md') > 0"
    peso: 0.03

  - id: "IND-010"
    nombre: "env_example_exists"
    dominio: "seguridad"
    formula: "existe('.env.example')"
    peso: 0.02

  - id: "IND-011"
    nombre: "srie_graph_exists"
    dominio: "gobernanza"
    formula: "existe('SDOS/GRAPH.md')"
    peso: 0.02

  - id: "IND-012"
    nombre: "capability_detected"
    dominio: "desarrollo"
    formula: "existe('capabilities/') AND count('capabilities/*/*.manifest') > 0"
    peso: 0.03

  - id: "IND-013"
    nombre: "identity_valid"
    dominio: "gobernanza"
    formula: "identity_chain_valid('SDOS/IDENTITY.yaml')"
    peso: 0.04
```

### Nivel 2: Salud

Indicadores compuestos que evalúan calidad.

```yaml
indicadores_salud:
  - id: "IND-020"
    nombre: "documentation_health"
    dominio: "gobernanza"
    formula: |
      (readme_exists × 0.30)
      + (architecture_exists × 0.25)
      + (changelog_exists × 0.15)
      + (memory_exists × 0.30)
    rango: [0, 100]
    threshold:
      warning: 70
      critical: 40

  - id: "IND-021"
    nombre: "architecture_health"
    dominio: "arquitectura"
    formula: |
      (existe('ARCHITECTURE.md') × 0.30)
      + (adr_count / max_adr_expected × 0.25)
      + (design_exists × 0.20)
      + (graph_consistency × 0.25)
    rango: [0, 100]
    threshold:
      warning: 65
      critical: 35

  - id: "IND-022"
    nombre: "security_health"
    dominio: "seguridad"
    formula: |
      (env_example_exists × 0.25)
      + (no_hardcoded_secrets × 0.30)
      + (dependencies_scanned × 0.25)
      + (auth_implemented × 0.20)
    rango: [0, 100]

  - id: "IND-023"
    nombre: "deployment_health"
    dominio: "operacion"
    formula: |
      (docker_detected × 0.25)
      + (cicd_detected × 0.30)
      + (rollback_strategy × 0.25)
      + (health_checks × 0.20)
    rango: [0, 100]

  - id: "IND-024"
    nombre: "knowledge_health"
    dominio: "conocimiento"
    formula: |
      (existe('knowledge/index.md') × 0.30)
      + (pattern_count / max_patterns × 0.25)
      + (antipattern_count / max_antipatterns × 0.20)
      + (knowledge_freshness × 0.25)
    rango: [0, 100]

  - id: "IND-025"
    nombre: "federation_health"
    dominio: "organizacional"
    entidad: "federation"
    formula: |
      (domains_connected / total_domains × 0.30)
      + (avg_contract_confidence × 0.25)
      + (contracts_active / max_contracts × 0.20)
      + (heartbeat_success_rate × 0.25)
    rango: [0, 100]
```

### Nivel 3: Riesgo

Indicadores que diagnostican peligros potenciales.

```yaml
indicadores_riesgo:
  - id: "IND-030"
    nombre: "maintenance_risk"
    dominio: "arquitectura"
    formula: |
      (1.0 - architecture_health / 100) × 0.30
      + (tech_debt_ratio × 0.25)
      + (dependency_age / 365 × 0.20)
      + (coverage_gap × 0.25)
    rango: [0, 1]
    threshold:
      warning: 0.50
      critical: 0.75
    clasificacion:
      - "BAJO: < 0.30"
      - "MEDIO: 0.30 - 0.50"
      - "ALTO: 0.50 - 0.75"
      - "CRÍTICO: > 0.75"

  - id: "IND-031"
    nombre: "deployment_risk"
    dominio: "operacion"
    formula: |
      (1.0 - cicd_detected) × 0.30
      + (1.0 - rollback_strategy) × 0.25
      + (deploy_fail_rate) × 0.25
      + (1.0 - docker_detected) × 0.20
    rango: [0, 1]

  - id: "IND-032"
    nombre: "hallucination_risk"
    dominio: "ia"
    formula: |
      (1.0 - confidence_promedio) × 0.35
      + (speculative_claims / total_claims × 0.25)
      + (memory_gaps / total_memory × 0.20)
      + (1.0 - knowledge_health / 100) × 0.20
    rango: [0, 1]
    threshold:
      warning: 0.30
      critical: 0.50
    clasificacion:
      - "MUY BAJO: < 0.10"
      - "BAJO: 0.10 - 0.30"
      - "MEDIO: 0.30 - 0.50"
      - "ALTO: > 0.50"

  - id: "IND-033"
    nombre: "tech_debt_risk"
    dominio: "arquitectura"
    formula: "composite(dependency_age, coverage_gap, code_smells, architecture_violations)"
    rango: [0, 1]

  - id: "IND-034"
    nombre: "fragmentation_risk"
    dominio: "organizacional"
    entidad: "federation"
    formula: |
      (orphan_domains / total_domains × 0.30)
      + (duplicate_capabilities / total_capabilities × 0.25)
      + (disconnected_domains / total_domains × 0.25)
      + (avg_response_time / max_response_time × 0.20)
    rango: [0, 1]
```

### Nivel 4: Predictivo

Indicadores que estiman probabilidades futuras.

```yaml
indicadores_predictivos:
  - id: "IND-040"
    nombre: "production_failure_probability"
    dominio: "operacion"
    formula: "predictive(deployment_risk, maintenance_risk, test_coverage, deploy_frequency)"
    rango: [0, 1]
    descripcion: "Probabilidad estimada de romper producción en los próximos 30 días"

  - id: "IND-041"
    nombre: "tech_debt_30d_probability"
    dominio: "arquitectura"
    formula: "predictive(tech_debt_risk, commit_frequency, refactor_ratio)"
    rango: [0, 1]
    descripcion: "Probabilidad de aumentar la deuda técnica en los próximos 30 días"

  - id: "IND-042"
    nombre: "domain_conflict_probability"
    dominio: "organizacional"
    entidad: "federation"
    formula: "predictive(fragmentation_risk, contract_disputes, authority_violations)"
    rango: [0, 1]

  - id: "IND-043"
    nombre: "hallucination_30d_probability"
    dominio: "ia"
    formula: "predictive(hallucination_risk, memory_freshness, knowledge_growth)"
    rango: [0, 1]
```

---

## Scores compuestos

Los scores no están en el catálogo. Se calculan a partir de los indicadores:

```yaml
scores:
  engineering_score:
    formula: "(architecture_health × 0.25) + (deployment_health × 0.20) + (security_health × 0.20) + (maintenance_risk_inv × 0.20) + (hallucination_risk_inv × 0.15)"
    rango: [0, 100]

  organizational_score:
    formula: "(federation_health × 0.35) + (fragmentation_risk_inv × 0.30) + (governance_compliance × 0.20) + (identity_chain_valid × 0.15)"
    rango: [0, 100]
    entidad: "federation"

  knowledge_score:
    formula: "(knowledge_health × 0.40) + (memory_exists × 0.20) + (adr_quality × 0.20) + (pattern_reuse × 0.20)"
    rango: [0, 100]

  evolution_score:
    formula: "(score_trend × 0.30) + (capability_growth × 0.25) + (coverage_improvement × 0.25) + (knowledge_growth × 0.20)"
    rango: [0, 100]

  global_srie_score:
    formula: "(engineering_score × 0.35) + (organizational_score × 0.25) + (knowledge_score × 0.20) + (evolution_score × 0.20)"
    rango: [0, 100]
```

---

## Registro de indicadores

Cada indicador medido se registra en `SDOS/evidence/indicators/`:

```yaml
# SDOS/evidence/indicators/2026-07-15/IND-020.json
{
  "indicador_id": "IND-020",
  "nombre": "documentation_health",
  "valor": 73.5,
  "confianza": 0.92,
  "tendencia": "DECLINING",
  "tendencia_pendiente": -2.3,
  "ultima_medicion": "2026-07-15T10:00:00Z",
  "mediciones_historial": 12,
  "threshold_warning": 70,
  "threshold_critical": 40,
  "evento_emitido": true,
  "evento_tipo": "DOCUMENTATION_WARNING",
  "evidencia": {
    "fuentes": 4,
    "archivos": ["SDOS/PROJECT_STATE.md"]
  }
}
```
