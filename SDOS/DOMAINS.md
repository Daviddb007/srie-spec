# DOMAINS — Dominios del Proyecto

---

## ¿Qué es este documento?

DOMAINS.md define **todos los dominios posibles** de un proyecto gobernado por SRIE. Cada archivo del proyecto pertenece a un dominio. Cada nodo del grafo pertenece a un dominio. Los dominios permiten clasificar, diagnosticar y gobernar el proyecto de forma estructurada.

---

## ¿Qué es un dominio?

Un dominio es una **categoría semántica** que agrupa archivos, capacidades y artefactos relacionados con un aspecto específico del proyecto.

Ejemplo: el archivo `Dockerfile` pertenece al dominio `infrastructure`. La capability `Calendar` pertenece al dominio `functional`.

---

## Catálogo de dominios

### 1. BUSINESS

| Campo | Valor |
|-------|-------|
| ID | `business` |
| Descripción | Propósito del proyecto, objetivos de negocio, stakeholders, métricas de éxito |
| Color | `#4A90D9` |
| Nivel SRIE mínimo | L0 |

**Archivos típicos:**

```
README.md
ROADMAP.md
BUSINESS.md (opcional)
```

**Indicadores asociados:**

- Existe README.md
- Existe ROADMAP.md
- El README explica el propósito del proyecto

---

### 2. FUNCTIONAL

| Campo | Valor |
|-------|-------|
| ID | `functional` |
| Descripción | Capacidades funcionales del proyecto: features, módulos, APIs |
| Color | `#7B68EE` |
| Nivel SRIE mínimo | L1 |

**Archivos típicos:**

```
capabilities/*/
capabilities/*/*.md
capabilities/*/*_design.md
capabilities/*/*_api.md
```

**Indicadores asociados:**

- Las capabilities tienen manifest
- Las capabilities tienen tests
- Las capabilities tienen diseño documentado

---

### 3. TECHNICAL

| Campo | Valor |
|-------|-------|
| ID | `technical` |
| Descripción | Decisiones técnicas, arquitectura, patrones, deuda técnica |
| Color | `#2E8B57` |
| Nivel SRIE mínimo | L1 |

**Archivos típicos:**

```
ARCHITECTURE.md
DESIGN.md
memory/ADR/
memory/evolution.md
```

**Indicadores asociados:**

- Existe ARCHITECTURE.md
- Existen ADR para decisiones recientes
- La arquitectura está documentada con diagramas

---

### 4. INFRASTRUCTURE

| Campo | Valor |
|-------|-------|
| ID | `infrastructure` |
| Descripción | Infraestructura técnica: contenedores, CI/CD, deploy, cloud |
| Color | `#CD5C5C` |
| Nivel SRIE mínimo | L2 |

**Archivos típicos:**

```
Dockerfile
docker-compose.yml
.github/
.gitlab-ci.yml
Jenkinsfile
.helm/
terraform/
```

**Indicadores asociados:**

- Existe Dockerfile
- Existe CI/CD configurado
- El deploy es automatizado

---

### 5. DOCUMENTATION

| Campo | Valor |
|-------|-------|
| ID | `documentation` |
| Descripción | Documentación general del proyecto, guías, contribución |
| Color | `#4682B4` |
| Nivel SRIE mínimo | L1 |

**Archivos típicos:**

```
README.md
CONTRIBUTING.md
CHANGELOG.md
docs/
wiki/
```

**Indicadores asociados:**

- Existe README.md
- Existe CHANGELOG.md
- La documentación está actualizada

---

### 6. SECURITY

| Campo | Valor |
|-------|-------|
| ID | `security` |
| Descripción | Seguridad: secretos, dependencias, headers, autenticación, autorización |
| Color | `#DC143C` |
| Nivel SRIE mínimo | L2 |

**Archivos típicos:**

```
.env.example
SECURITY.md (opcional)
.security/
```

**Indicadores asociados:**

- No hay secretos hardcodeados
- Las variables de entorno están documentadas
- No hay dependencias con CVSs conocidas

---

### 7. DEPLOYMENT

| Campo | Valor |
|-------|-------|
| ID | `deployment` |
| Descripción | Estado del despliegue, releases, versiones, rollbacks |
| Color | `#DAA520` |
| Nivel SRIE mínimo | L2 |

**Archivos típicos:**

```
SDOS/DEPLOY_REPORT.md
capabilities/*/*_release.md
VERSION (opcional)
```

**Indicadores asociados:**

- Existe estrategia de deploy
- Existe estrategia de rollback
- Los deploys son registrados

---

### 8. OBSERVABILITY

| Campo | Valor |
|-------|-------|
| ID | `observability` |
| Descripción | Monitoreo, logging, métricas, alertas, health checks |
| Color | `#20B2AA` |
| Nivel SRIE mínimo | L3 |

**Archivos típicos:**

```
capabilities/*/*_metrics.md
SDOS/evidence/
SDOS/SRIE_REPORT.md
```

**Indicadores asociados:**

- Existe logging estructurado
- Existen health checks
- Existen alertas configuradas

---

### 9. AI

| Campo | Valor |
|-------|-------|
| ID | `ai` |
| Descripción | Aspectos relacionados con IA: prompt coverage, capabilities, knowledge, riesgo de alucinación |
| Color | `#9370DB` |
| Nivel SRIE mínimo | L2 |

**Archivos típicos:**

```
knowledge/
knowledge/patterns/
knowledge/anti_patterns/
SDOS/SRIE_ENGINE.md
```

**Indicadores asociados:**

- Existe prompt coverage
- Las capabilities están definidas
- El knowledge base tiene patrones documentados
- Riesgo de alucinación bajo

---

### 10. KNOWLEDGE

| Campo | Valor |
|-------|-------|
| ID | `knowledge` |
| Descripción | Conocimiento acumulado: patrones, antipatrones, lecciones aprendidas |
| Color | `#696969` |
| Nivel SRIE mínimo | L3 |

**Archivos típicos:**

```
knowledge/
knowledge/index.md
knowledge/patterns/
knowledge/anti_patterns/
knowledge/performance_patterns/
knowledge/security_patterns/
```

**Indicadores asociados:**

- Existe knowledge/index.md
- Hay patrones documentados
- Hay antipatrones documentados
- El conocimiento se actualiza por iteración

---

### 11. GOVERNANCE

| Campo | Valor |
|-------|-------|
| ID | `governance` |
| Descripción | Gobierno del proyecto: reglas, contratos, madurez, estado SDOS |
| Color | `#8B0000` |
| Nivel SRIE mínimo | L1 |

**Archivos típicos:**

```
SDOS/STATE.md
SDOS/PROJECT.yaml
SDOS/PROJECT_STATE.md
SDOS/GRAPH.md
SDOS/SRIE_ENGINE.md
SDOS/SRIE_INDICATORS.md
SDOS/ENGINE_CONTRACT.md
SDOS/GOVERNANCE_REPORT.md
```

**Indicadores asociados:**

- SDOS/STATE.md existe y está actualizado
- SDOS/GRAPH.md existe
- El nivel de madurez está definido

---

## Jerarquía de dominios

```
                    ┌──────────────┐
                    │   BUSINESS    │
                    │  (propósito)  │
                    └──────┬───────┘
                           │
              ┌────────────┼────────────┐
              │            │            │
       ┌──────▼─────┐ ┌───▼────┐ ┌────▼──────┐
       │ GOVERNANCE │ │FUNCTION│ │ TECHNICAL │
       │  (reglas)  │ │  -AL   │ │ (arquitect)│
       └──────┬─────┘ └───┬────┘ └────┬──────┘
              │           │           │
              ▼           ▼           ▼
       ┌───────────┐ ┌────────┐ ┌──────────┐
       │ DOCUMENTA │ │ SECURI │ │ INFRASTR │
       │  -TION    │ │  -TY   │ │  -UCTURE │
       └───────────┘ └────────┘ └────┬─────┘
                                      │
                                      ▼
                               ┌──────────┐
                               │ DEPLOYME │
                               │  -NT     │
                               └────┬─────┘
                                    │
                                    ▼
                              ┌───────────┐
                              │ OBSERVABI │
                              │  -LITY    │
                              └─────┬─────┘
                                    │
                    ┌───────────────┼───────────────┐
                    │               │               │
              ┌─────▼──────┐ ┌──────▼─────┐ ┌──────▼──────┐
              │     AI     │ │ KNOWLEDGE  │ │   MEMORY    │
              │ (IA engine)│ │ (patrones)  │ │ (decisiones)│
              └────────────┘ └────────────┘ └─────────────┘
```

---

## Mapeo archivo → dominio

Cada archivo del proyecto se mapea a un dominio según su ruta y nombre:

```yaml
mapeo:
  - patron: "README.md"
    dominio: business
  - patron: "ROADMAP.md"
    dominio: business

  - patron: "ARCHITECTURE.md"
    dominio: technical
  - patron: "DESIGN.md"
    dominio: technical
  - patron: "memory/ADR/*"
    dominio: technical

  - patron: "capabilities/*/"
    dominio: functional
  - patron: "capabilities/*/*.manifest"
    dominio: functional
  - patron: "capabilities/*/*_api.md"
    dominio: functional

  - patron: "Dockerfile"
    dominio: infrastructure
  - patron: "docker-compose.yml"
    dominio: infrastructure
  - patron: ".github/**"
    dominio: infrastructure

  - patron: "CHANGELOG.md"
    dominio: documentation
  - patron: "CONTRIBUTING.md"
    dominio: documentation
  - patron: "docs/**"
    dominio: documentation

  - patron: ".env.example"
    dominio: security

  - patron: "SDOS/DEPLOY_REPORT.md"
    dominio: deployment
  - patron: "capabilities/*/*_release.md"
    dominio: deployment

  - patron: "capabilities/*/*_metrics.md"
    dominio: observability
  - patron: "SDOS/SRIE_REPORT.md"
    dominio: observability

  - patron: "knowledge/**"
    dominio: knowledge

  - patron: "memory/**"
    dominio: memory (hereda de technical)

  - patron: "SDOS/STATE.md"
    dominio: governance
  - patron: "SDOS/GRAPH.md"
    dominio: governance
  - patron: "SDOS/PROJECT.yaml"
    dominio: governance
  - patron: "SDOS/SRIE_ENGINE.md"
    dominio: ai
  - patron: "SDOS/ENGINE_CONTRACT.md"
    dominio: governance
  - patron: "SDOS/SRIE_INDICATORS.md"
    dominio: governance
```

---

## Reglas de dominio

### Regla 1: Pertenencia única

Cada archivo pertenece a un único dominio. No puede estar en dos dominios simultáneamente.

### Regla 2: Herencia

Los archivos dentro de un directorio heredan el dominio del directorio raíz a menos que se especifique un mapeo explícito.

### Regla 3: Dominio mínimo

Cada proyecto debe tener al menos los dominios `business`, `governance` y `technical` para alcanzar L1.

### Regla 4: Completitud progresiva

A medida que el proyecto sube de nivel de madurez, debe incorporar más dominios:

| Nivel | Dominios requeridos |
|-------|---------------------|
| L0 | Ninguno (solo descubrimiento) |
| L1 | business, governance, technical |
| L2 | + functional, documentation, infrastructure, security |
| L3 | + deployment, ai |
| L4 | + observability, knowledge |
| L5 | Todos los dominios |

### Regla 5: Diagnóstico por dominio

Los indicadores de madurez se calculan por dominio. Un dominio con baja puntuación arrastra el nivel general del proyecto.
