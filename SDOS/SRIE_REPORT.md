# SRIE Report — Stonelytics Studio

**Fecha:** 2026-07-15
**Proyecto:** SRIE_Engine (Stonelytics Studio)
**SRIE Foundation:** v1.0.0 | **Nivel de Madurez:** L1
**Dominio:** DOM-STONELYTICS-001 | **Propósito:** Plataforma de ingeniería empresarial asistida por IA

---

## Scores

| Score | Valor | Tendencia | Clasificación |
|-------|-------|-----------|---------------|
| **Global SRIE Score** | **46.2** | STABLE | L1 — Proyecto documentado parcialmente |
| Engineering Score | 38.5 | STABLE | ⚠️ Requiere atención |
| Organizational Score | 25.0 | STABLE | ❌ Sin README, sin ARCHITECTURE |
| Knowledge Score | 82.0 | STABLE | ✅ Fortaleza del proyecto |
| Evolution Score | 15.0 | STABLE | ❌ Solo 5 commits, 1 autor |

---

## Indicadores por Nivel

### Observación (13/13)

| Indicador | Valor | Confianza | Estado |
|-----------|-------|-----------|--------|
| readme_exists | ❌ | 1.00 | 🚨 CRITICAL |
| architecture_exists | ❌ | 1.00 | 🚨 CRITICAL |
| changelog_exists | ❌ | 1.00 | ⚠️ WARNING |
| tests_detected | ✅ | 0.95 | ✅ OK (92 tests) |
| docker_detected | ❌ | 1.00 | ⚠️ WARNING |
| cicd_detected | ❌ | 1.00 | ⚠️ WARNING |
| git_detected | ✅ | 0.99 | ✅ OK |
| memory_exists | ✅ | 0.99 | ✅ OK (67 SDOS docs) |
| adr_exists | ✅ | 0.95 | ✅ OK |
| env_example_exists | ✅ | 1.00 | ✅ OK |
| srie_graph_exists | ✅ | 1.00 | ✅ OK |
| capability_detected | ✅ | 0.95 | ✅ OK (12 capabilities) |
| identity_valid | ✅ | 1.00 | ✅ OK |

### Salud (5/5)

| Indicador | Valor | Tendencia | Threshold | Estado |
|-----------|-------|-----------|-----------|--------|
| documentation_health | **25.0** | STABLE | ⚠️ 70 / 🚨 40 | 🚨 **CRITICAL** |
| architecture_health | **57.0** | STABLE | ⚠️ 65 / 🚨 35 | ⚠️ WARNING |
| security_health | **45.0** | STABLE | ⚠️ 70 / 🚨 40 | ⚠️ WARNING |
| deployment_health | **15.0** | STABLE | ⚠️ 70 / 🚨 40 | 🚨 **CRITICAL** |
| knowledge_health | **72.0** | STABLE | ⚠️ 50 / 🚨 30 | ✅ OK |

### Riesgo (4/4)

| Indicador | Valor | Clasificación | Estado |
|-----------|-------|---------------|--------|
| maintenance_risk | 0.58 | **MEDIO** | ⚠️ WARNING |
| deployment_risk | 0.80 | **CRÍTICO** | 🚨 CRITICAL |
| hallucination_risk | 0.12 | **MUY BAJO** | ✅ OK |
| tech_debt_risk | 0.45 | MEDIO | ⚠️ WARNING |

---

## Diagnóstico Diferencial

### Principal: HYP-001 — Documentación de gobierno ausente (confianza: 0.85)

El proyecto no tiene README.md ni ARCHITECTURE.md. La documentation_health está en 25/100 (CRÍTICO). Sin embargo, existe un activo excepcional: 67 documentos SDOS que funcionan como especificación del sistema. La intervención recomendada es crear README.md y ARCHITECTURE.md para cerrar la brecha de gobierno.

### Alternativa 1: HYP-002 — Deploy frágil (confianza: 0.82)
Deployment_health en 15/100. Sin Docker, sin CI/CD, deploy manual vía SSH. Riesgo alto pero no bloqueante.

### Alternativa 2: HYP-003 — Riesgo de seguridad (confianza: 0.78)
Secretos hardcodeados en deploy.bat y config.py. CSRF exempt en studio blueprint. Requiere atención pero no es crítica inmediata.

### Activo identificado: HYP-005 — Fortaleza SDOS (confianza: 0.88)
67 documentos de especificación SRIE. Industry packs, AI prompts, solution mapper. Este es el activo más valioso del proyecto y debe preservarse.

---

## Eventos Activos

| Evento | Desde | Severidad |
|--------|-------|-----------|
| DOCUMENTATION_CRITICAL | 2026-07-15 | 🔴 CRITICAL |
| DEPLOYMENT_CRITICAL | 2026-07-15 | 🔴 CRITICAL |
| ARCHITECTURE_WARNING | 2026-07-15 | 🟡 WARNING |
| SECURITY_WARNING | 2026-07-15 | 🟡 WARNING |
| MAINTENANCE_RISK_MEDIUM | 2026-07-15 | 🟡 WARNING |
| DEPLOYMENT_RISK_CRITICAL | 2026-07-15 | 🔴 CRITICAL |

---

## Acciones Sugeridas

| Prioridad | Acción | Dominio | Esfuerzo | Impacto Esperado |
|-----------|--------|---------|----------|------------------|
| 🚨 | Crear README.md | gobernanza | 1 día | doc_health: +15 |
| 🚨 | Crear ARCHITECTURE.md | arquitectura | 1 día | doc_health: +15, arch_health: +20 |
| 🔴 | Eliminar secretos hardcodeados | seguridad | 0.5 días | sec_health: +15 |
| 🔴 | Re-stringir CSRF exempt en studio | seguridad | 0.5 días | sec_health: +10 |
| 🟡 | Configurar CI/CD (GitHub Actions) | operación | 1 día | dep_health: +30 |
| 🟡 | Unificar pdf_service/pdf_generator | desarrollo | 0.5 días | tech_debt: -0.10 |
| 🟡 | Limpiar test DBs en instance/ | desarrollo | 0.2 días | tech_debt: -0.05 |

---

## Perfil del Proyecto

**Fortalezas:**
- 67 documentos SDOS de especificación (activo de gobernanza único)
- 92 tests con pytest (buena cobertura de funcionalidades)
- 12 capabilities funcionales bien modularizadas
- Industry packs, AI prompts y solution mapper (conocimiento de dominio estructurado)
- CSRF habilitado globalmente (excepto studio)
- ORM parameterizado (sin SQL injection)

**Debilidades:**
- Sin README.md ni ARCHITECTURE.md (documentación de gobierno ausente)
- Deploy manual sin CI/CD ni Docker
- Secretos hardcodeados en scripts de deploy
- CSRF exempt en studio blueprint (superficie de ataque)
- Duplicación de código (pdf_service/pdf_generator)
- Sin herramienta de cobertura de tests
- Sin linter ni formatter configurados
- 11 bases de datos de prueba en el working directory
- 1 solo autor, 5 commits (proyecto joven)

---

## Metadata

```
Engine:             SRIE Engine v1.0.0
Foundation:         v1.0.0
Indicators Engine:  SPRINT 0 (manual)
DX Engine:          SPRINT 0 (manual)
SRIE_SCORE:         46.2 / 100
Nivel de Madurez:   L1 (Proyecto documentado parcialmente)
Dominios OK:        3/8 (Memoria, Conocimiento, Tests)
Dominios WARNING:   3/8 (Arquitectura, Seguridad, Desarrollo)
Dominios CRITICAL:  2/8 (Gobernanza, Operación)
```
