# CONSTRAINTS — Restricciones del Sistema

---

## ¿Qué es este documento?

CONSTRAINTS.md define **las restricciones del sistema** — aquello que nunca puede romperse, sin importar el objetivo o la urgencia del diagnóstico. Las restricciones son la **línea roja** del proyecto: si una intervención las viola, no puede ejecutarse, aunque resuelva el problema.

---

## Tipos de restricciones

```yaml
tipos_restriccion:
  - tipo: "técnica"
    descripcion: "Restricciones de tecnología, arquitectura, compatibilidad"
    ejemplo: "Soporte para Python 3.12+"

  - tipo: "operativa"
    descripcion: "Restricciones de proceso, deploy, operación"
    ejemplo: "No se deploya sin tests pasando"

  - tipo: "de gobierno"
    descripcion: "Restricciones de documentación, memoria, evidencia"
    ejemplo: "No se elimina memoria del sistema"

  - tipo: "de seguridad"
    descripcion: "Restricciones de secretos, auth, dependencias"
    ejemplo: "Los secretos nunca se hardcodean"

  - tipo: "de negocio"
    descripcion: "Restricciones de presupuesto, recurso, tiempo"
    ejemplo: "Presupuesto mensual fijo"

  - tipo: "de federación"
    descripcion: "Restricciones de contratos, memoria compartida, autoridad"
    ejemplo: "Un Domain no puede leer la memoria de otro sin contrato"
```

---

## Formato de una restricción

```yaml
restriccion:
  id: "CON-001"
  nombre: "No romper compatibilidad hacia atrás"
  tipo: "técnica"
  severidad: "critical"           # warning | critical

  descripcion: |
    Ningún cambio puede romper la compatibilidad con versiones anteriores.
    Esto incluye: APIs públicas, formatos de datos, contratos activos.

  dominio: "arquitectura"

  aplica_a:
    - "intervenciones del Planner"
    - "cambios en capabilities públicas"
    - "modificaciones de API"

  sancion_por_violacion: "La intervención se rechaza automáticamente"
  puede_excepcionarse: false     # true = con aprobación del nivel superior
  excepcion_requiere: null

  ultima_revision: "2026-07-15"
  revisada_por: "SRIE Core"
```

---

## Catálogo de restricciones

### CON-001: Compatibilidad hacia atrás

```yaml
id: "CON-001"
nombre: "No romper compatibilidad hacia atrás"
tipo: "técnica"
severidad: "critical"
puede_excepcionarse: false
```

### CON-002: Secretos hardcodeados

```yaml
id: "CON-002"
nombre: "No hardcodear secretos"
tipo: "seguridad"
severidad: "critical"
puede_excepcionarse: false
```

### CON-003: Presupuesto fijo

```yaml
id: "CON-003"
nombre: "Presupuesto mensual fijo"
tipo: "negocio"
severidad: "warning"
puede_excepcionarse: true
excepcion_requiere: "Aprobación del Domain padre"
```

### CON-004: Tests antes de deploy

```yaml
id: "CON-004"
nombre: "Tests antes de deploy"
tipo: "operativa"
severidad: "critical"
puede_excepcionarse: false
```

### CON-005: Memoria inmutable

```yaml
id: "CON-005"
nombre: "No eliminar memoria del sistema"
tipo: "gobierno"
severidad: "critical"
puede_excepcionarse: false
```

### CON-006: Memoria privada entre Domains

```yaml
id: "CON-006"
nombre: "Memoria privada entre Domains"
tipo: "federación"
severidad: "critical"
puede_excepcionarse: false
aplica_a: ["federation"]
```

### CON-007: Intervenciones simultáneas

```yaml
id: "CON-007"
nombre: "Máximo 2 intervenciones simultáneas"
tipo: "operativa"
severidad: "warning"
puede_excepcionarse: true
excepcion_requiere: "Diagnóstico CRÍTICO en otro dominio"
```

---

## Validación de restricciones

Cada vez que el Intent Layer genera una intención, el Constraint Checker valida contra todas las restricciones activas:

```yaml
validacion_restricciones:
  entrada: "Intención propuesta"
  proceso:
    1. "Cargar todas las restricciones activas del proyecto"
    2. "Para cada restricción:"
       a. "¿La intención viola esta restricción?"
       b. "Si sí: ¿la restricción puede excepcionarse?"
       c. "Si no puede excepcionarse: marcar intención como INVALIDADA"
       d. "Si puede excepcionarse: marcar excepción como REQUERIDA"
    3. "Si hay excepciones requeridas: solicitar aprobación"
  salida:
    valida: true | false
    restricciones_violadas: []
    excepciones_requeridas: []
```

---

## Ciclo de vida de una restricción

```yaml
constraint_lifecycle:
  DEFINIDA: "Restricción creada en PROJECT_INTENT.md"
    │
    ▼
  ACTIVA: "Restricción vigente"
    │
    ├── EXCEPTIONADA: "Excepción temporal aprobada"
    │       │
    │       ▼
    │   ACTIVA (expirada la excepción)
    │
    └── ARCHIVADA: "Restricción ya no aplica"
```

### Reglas de excepción

1. **Toda excepción debe tener fecha de expiración.** No existen excepciones permanentes.
2. **Toda excepción debe dejar evidencia.** Quién la aprobó, por qué, hasta cuándo.
3. **Una restricción con 3+ excepciones en 6 meses debe revisarse.** Probablemente es demasiado restrictiva.
