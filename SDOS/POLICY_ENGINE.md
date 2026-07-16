# POLICY ENGINE — Políticas Ejecutables del Sistema

---

## ¿Qué es este documento?

POLICY_ENGINE.md define el **motor de políticas** de SRIE. No es documentación de reglas. Son **políticas ejecutables** que el sistema verifica antes de permitir acciones. Mientras que CONSTRAINTS.md define restricciones estratégicas, el Policy Engine las hace cumplir en tiempo de ejecución.

---

## ¿Qué es una política?

Una política es una **regla ejecutable** que el sistema verifica antes, durante o después de una acción.

```yaml
policy:
  id: "POL-001"
  nombre: "No desplegar viernes"
  tipo: "pre"                    # pre (antes de ejecutar) | post (después) | continuous (siempre)
  severidad: "warning"           # warning | error | critical

  condicion: |
    IF day_of_week = "Friday"
    THEN deny("No se permite desplegar en viernes")

  accion_si_viola:
    - "Rechazar la acción con mensaje: 'No se permite desplegar en viernes'"
    - "Emitir evento POLICY_VIOLATED"
    - "Registrar en evidencia"

  puede_excepcionarse: true
  excepcion_requiere: "Aprobación del Domain padre"
```

---

## Catálogo de políticas

### Pre-ejecución (se verifican antes de ejecutar)

```yaml
policies_pre:
  - id: "POL-001"
    nombre: "No desplegar viernes"
    condicion: "day_of_week = 'Friday' AND action = 'deploy'"
    severidad: "error"

  - id: "POL-002"
    nombre: "No borrar producción"
    condicion: "environment = 'production' AND action = 'delete'"
    severidad: "critical"

  - id: "POL-003"
    nombre: "No usar modelo externo sin autorización"
    condicion: "resource.type = 'ai_model' AND resource.provider != 'local'"
    severidad: "warning"

  - id: "POL-004"
    nombre: "No romper APIs públicas"
    condicion: "capability.is_public AND change.is_breaking"
    severidad: "critical"

  - id: "POL-005"
    nombre: "Coverage mínimo"
    condicion: "test_coverage < 70 AND action = 'deploy'"
    severidad: "error"

  - id: "POL-006"
    nombre: "Memory antes de deploy"
    condicion: "memory_freshness < 30 AND action = 'deploy'"
    severidad: "warning"
```

### Post-ejecución (se verifican después de ejecutar)

```yaml
policies_post:
  - id: "POL-010"
    nombre: "Verificar deploy exitoso"
    condicion: "deploy_status != 'SUCCESS'"
    severidad: "critical"
    accion: "Iniciar rollback automático"

  - id: "POL-011"
    nombre: "Verificar health después de cambio"
    condicion: "health_check_status != 'HEALTHY'"
    severidad: "critical"
    accion: "Notificar a Governance Engine"
```

### Continuas (siempre activas)

```yaml
policies_continuous:
  - id: "POL-020"
    nombre: "Secretos en código"
    condicion: "secret_detected_in_code = true"
    severidad: "critical"
    accion: "Emitir SECRET_LEAK_DETECTED, bloquear commit"

  - id: "POL-021"
    nombre: "Dependencias vulnerables"
    condicion: "vulnerable_dependencies > 0"
    severidad: "warning"
    accion: "Emitir VULNERABLE_DEPENDENCY_DETECTED"

  - id: "POL-022"
    nombre: "Confianza mínima"
    condicion: "confidence_score < 0.40 AND action = 'automatic'"
    severidad: "error"
    accion: "Requiere revisión humana antes de continuar"
```

---

## Formato de condición

```yaml
condition_format:
  sintaxis: "IF <condición> THEN <acción>"
  operadores:
    - "AND, OR, NOT"
    - "=, !=, >, <, >=, <=, contains, matches"
    - "day_of_week, hour_of_day"
  variables:
    - "action: string (deploy | create | delete | update | execute)"
    - "environment: string (production | staging | development)"
    - "resource.type: string"
    - "resource.provider: string"
    - "capability.is_public: boolean"
    - "change.is_breaking: boolean"
    - "test_coverage: float"
    - "confidence_score: float"
    - "memory_freshness: integer (días)"
```

---

## Evaluación de políticas

```yaml
policy_evaluation:
  flujo:
    1. "Acción solicitada (ej: deploy, create capability)"
    2. "Policy Engine selecciona políticas relevantes"
    3. "Evalúa cada política (pre-ejecución)"
    4. "Si alguna política CRITICAL falla → bloquear acción"
    5. "Si alguna política ERROR falla → bloquear acción"
    6. "Si alguna política WARNING falla → registrar advertencia, permitir"
    7. "Ejecutar acción"
    8. "Evaluar políticas post-ejecución"
    9. "Registrar resultado"

  respuesta:
    permitida: true | false | con_advertencias
    politicas_violadas:
      - id: "POL-001"
        severidad: "error"
        mensaje: "No se permite desplegar en viernes"
    advertencias:
      - id: "POL-006"
        severidad: "warning"
        mensaje: "Memory freshness baja (20 días). Recomendar actualizar."
```

---

## Policy Registry

Las políticas se almacenan en `SDOS/policies/`:

```yaml
# SDOS/policies/POL-001.yaml
policy:
  id: "POL-001"
  nombre: "No desplegar viernes"
  tipo: "pre"
  severidad: "error"
  condicion: "day_of_week = 'Friday' AND action = 'deploy'"
  creada: "2026-07-15"
  ultima_revision: "2026-07-15"
  revisada_por: "SRIE Core"
  violaciones: 3
  excepciones_concedidas: 1
```
