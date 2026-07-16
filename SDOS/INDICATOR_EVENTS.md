# INDICATOR EVENTS — Eventos Generados por el Sistema de Indicadores

---

## ¿Qué es este documento?

INDICATOR_EVENTS.md define **todos los eventos** que el Indicators Engine puede generar. Cada evento corresponde a un cambio en el estado de un indicador, un cruce de threshold, una tendencia detectada o un cambio en los scores del sistema.

---

## Principio fundamental

**Indicators no produce números. Produce eventos.** Cada indicador es un sensor que emite eventos cuando su valor cambia significativamente, cruza un umbral o cambia su tendencia.

---

## Eventos por nivel de indicador

### Eventos de Observación

| Evento | Significado | Severidad |
|--------|-------------|-----------|
| `README_CREATED` | README.md fue creado | info |
| `README_DELETED` | README.md fue eliminado | warning |
| `ARCHITECTURE_CREATED` | ARCHITECTURE.md fue creado | info |
| `ARCHITECTURE_DELETED` | ARCHITECTURE.md fue eliminado | critical |
| `MEMORY_INITIALIZED` | memory/index.md fue creado | info |
| `MEMORY_DELETED` | memory/index.md fue eliminado | critical |
| `TESTS_DETECTED` | Tests fueron detectados por primera vez | info |
| `TESTS_LOST` | Tests dejaron de ser detectados | warning |
| `DOCKER_DETECTED` | Docker fue detectado | info |
| `CICD_DETECTED` | CI/CD fue detectado | info |
| `CAPABILITY_DETECTED` | Nueva capability detectada | info |
| `IDENTITY_CHANGED` | La identidad del nodo cambió | warning |
| `IDENTITY_EXPIRING` | La identidad está por expirar (30 días) | warning |

### Eventos de Salud

| Evento | Significado | Severidad |
|--------|-------------|-----------|
| `DOCUMENTATION_HEALTH_OK` | Documentación health ≥ threshold | info |
| `DOCUMENTATION_WARNING` | Documentación health < warning | warning |
| `DOCUMENTATION_CRITICAL` | Documentación health < critical | critical |
| `DOCUMENTATION_IMPROVING` | Documentación health mejorando consistentemente | info |
| `DOCUMENTATION_DEGRADING` | Documentación health deteriorándose | warning |
| `ARCHITECTURE_HEALTH_OK` | Arquitectura health ≥ threshold | info |
| `ARCHITECTURE_WARNING` | Arquitectura health < warning | warning |
| `ARCHITECTURE_CRITICAL` | Arquitectura health < critical | critical |
| `ARCHITECTURE_DEGRADING` | Arquitectura health deteriorándose | warning |
| `SECURITY_HEALTH_OK` | Seguridad health ≥ threshold | info |
| `SECURITY_WARNING` | Seguridad health < warning | warning |
| `SECURITY_CRITICAL` | Seguridad health < critical | critical |
| `DEPLOYMENT_HEALTH_OK` | Deployment health ≥ threshold | info |
| `DEPLOYMENT_WARNING` | Deployment health < warning | warning |
| `DEPLOYMENT_CRITICAL` | Deployment health < critical | critical |
| `KNOWLEDGE_HEALTH_OK` | Knowledge health ≥ threshold | info |
| `KNOWLEDGE_WARNING` | Knowledge health < warning | warning |
| `KNOWLEDGE_CRITICAL` | Knowledge health < critical | critical |
| `FEDERATION_HEALTH_OK` | Federation health ≥ threshold | info |
| `FEDERATION_WARNING` | Federation health < warning | warning |
| `FEDERATION_CRITICAL` | Federation health < critical | critical |

### Eventos de Riesgo

| Evento | Significado | Severidad |
|--------|-------------|-----------|
| `MAINTENANCE_RISK_LOW` | Riesgo de mantenimiento bajo | info |
| `MAINTENANCE_RISK_MEDIUM` | Riesgo de mantenimiento medio | warning |
| `MAINTENANCE_RISK_HIGH` | Riesgo de mantenimiento alto | error |
| `MAINTENANCE_RISK_CRITICAL` | Riesgo de mantenimiento crítico | critical |
| `DEPLOYMENT_RISK_LOW` | Riesgo de despliegue bajo | info |
| `DEPLOYMENT_RISK_HIGH` | Riesgo de despliegue alto | error |
| `DEPLOYMENT_RISK_CRITICAL` | Riesgo de despliegue crítico | critical |
| `HALLUCINATION_RISK_LOW` | Riesgo de alucinación bajo | info |
| `HALLUCINATION_RISK_MEDIUM` | Riesgo de alucinación medio | warning |
| `HALLUCINATION_RISK_HIGH` | Riesgo de alucinación alto | error |
| `HALLUCINATION_RISK_CRITICAL` | Riesgo de alucinación crítico | critical |
| `TECH_DEBT_RISK_LOW` | Deuda técnica baja | info |
| `TECH_DEBT_RISK_HIGH` | Deuda técnica alta | warning |
| `TECH_DEBT_RISK_CRITICAL` | Deuda técnica crítica | critical |
| `FRAGMENTATION_RISK_DETECTED` | Riesgo de fragmentación en la federación | warning |

### Eventos Predictivos

| Evento | Significado | Severidad |
|--------|-------------|-----------|
| `PRODUCTION_FAILURE_PROBABLE` | Probabilidad de fallo en producción > 30% | warning |
| `PRODUCTION_FAILURE_IMMINENT` | Probabilidad de fallo en producción > 50% | critical |
| `TECH_DEBT_PROJECTED` | Se proyecta aumento de deuda técnica | warning |
| `DOMAIN_CONFLICT_PROBABLE` | Probabilidad de conflicto entre Domains > 20% | warning |
| `KNOWLEDGE_STAGNATION_DETECTED` | El conocimiento no está creciendo | warning |

### Eventos de Score

| Evento | Significado | Severidad |
|--------|-------------|-----------|
| `ENGINEERING_SCORE_CHANGED` | Engineering Score cambió significativamente | info |
| `ORGANIZATIONAL_SCORE_CHANGED` | Organizational Score cambió significativamente | info |
| `KNOWLEDGE_SCORE_CHANGED` | Knowledge Score cambió significativamente | info |
| `EVOLUTION_SCORE_CHANGED` | Evolution Score cambió significativamente | info |
| `GLOBAL_SCORE_CHANGED` | Global SRIE Score cambió significativamente | info |
| `MATURITY_LEVEL_INCREASED` | El nivel de madurez subió | info |
| `MATURITY_LEVEL_DECREASED` | El nivel de madurez bajó | critical |
| `SCORE_TREND_NEGATIVE` | Tendencias negativas en múltiples scores | warning |

---

## Formato del evento

```json
{
  "id": "evt_ind_001",
  "tipo": "DOCUMENTATION_WARNING",
  "timestamp": "2026-07-15T10:00:00Z",
  "origen": {
    "engine": "Indicators Engine",
    "ejecucion_id": "exec_ind_015"
  },
  "entidad": {
    "tipo": "project",
    "id": "PROJECT-001"
  },
  "indicador": {
    "id": "IND-020",
    "nombre": "documentation_health",
    "valor_actual": 68.2,
    "threshold": 70,
    "diferencia": -1.8
  },
  "tendencia": {
    "direccion": "DECLINING",
    "pendiente": -2.3,
    "historial": [82, 80, 78, 75, 73.5, 71.2, 68.2]
  },
  "severidad": "warning",
  "accion_sugerida": "Actualizar ARCHITECTURE.md y memory/index.md",
  "relacionados": ["evt_disc_003"]
}
```

---

## Suscripción a eventos de indicadores

Los eventos de indicadores son consumidos por:

| Engine/Componente | Eventos que consume | Acción |
|-------------------|---------------------|--------|
| DX Engine | `*_DEGRADING`, `*_CRITICAL`, `*_WARNING` | Priorizar en diagnóstico DX |
| Planner Engine | `*_CRITICAL`, `*_DEGRADING`, `MATURITY_*` | Generar tareas automáticas |
| Governance Engine | `MATURITY_*`, `IDENTITY_*`, `FRAGMENTATION_*` | Auditar gobierno |
| Knowledge Engine | `*_IMPROVING`, `*_DEGRADING`, `KNOWLEDGE_*` | Consolidar patrones/antipatrones |
| BUS | Todos | Reenviar a Domains suscritos |
| SRIE Core | `*_CRITICAL`, `MATURITY_LEVEL_DECREASED` | Alertas del sistema |

---

## Reglas de eventos

1. **No hay eventos duplicados.** Si el mismo evento se emite dos veces seguidas con el mismo valor, se ignora.

2. **Debounce de eventos de threshold.** Si un indicador cruza un threshold, no se emite otro evento del mismo tipo hasta que el indicador salga del rango y vuelva a entrar.

3. **Eventos de tendencia requieren 3+ mediciones.** No se emite `*_DEGRADING` sin al menos 3 mediciones consecutivas que muestren la tendencia.

4. **Los eventos predictivos son informativos.** No activan acciones automáticas. Solo notifican.

5. **Cada evento se registra en SDOS/events/** siguiendo el formato de EVENTS.md.
