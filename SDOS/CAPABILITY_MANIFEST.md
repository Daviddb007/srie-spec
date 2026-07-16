# CAPABILITY MANIFEST — Contrato de Capacidades

---

## ¿Qué es este documento?

CAPABILITY_MANIFEST.md define el **contrato que toda Capability debe cumplir** dentro del sistema SRIE. Describe la estructura, los archivos obligatorios y opcionales, el formato del manifest y las reglas de composición entre capacidades.

---

## ¿Qué es una Capability? (refuerzo conceptual)

Una Capability no es código. Es una **capacidad funcional del sistema** que:

- Resuelve una necesidad específica del proyecto
- Es autocontenida: puede entenderse sin conocer el resto del sistema
- Es independiente: puede construirse, probarse y desplegarse sola
- Tiene memoria propia: registra sus decisiones y evolución
- Tiene métricas propias: se puede medir su calidad y rendimiento
- Tiene versión propia: se puede rastrear su historia
- Tiene contrato explícito: define qué espera y qué ofrece

---

## Estructura de una Capability

Toda Capability vive en `capabilities/<nombre>/` y sigue esta estructura:

```
capabilities/<nombre>/
├── <nombre>.manifest             # Manifest de la Capability (OBLIGATORIO)
├── <nombre>.md                   # Especificación funcional (OBLIGATORIO)
├── <nombre>_design.md            # Diseño técnico (OBLIGATORIO)
├── <nombre>_api.md               # Contrato de API (RECOMENDADO)
├── <nombre>_tests.md             # Tests (OBLIGATORIO)
├── <nombre>_memory.md            # Memoria de decisiones (OBLIGATORIO)
├── <nombre>_metrics.md           # Métricas (OBLIGATORIO)
├── <nombre>_release.md           # Notas de release (RECOMENDADO)
├── src/                          # Código fuente (si aplica)
├── tests/                        # Tests automatizados (si aplica)
└── docs/                         # Documentación adicional (opcional)
```

---

## Capability Manifest (`<nombre>.manifest`)

### Formato

```yaml
# capabilities/calendar/calendar.manifest
capability:
  nombre: "Calendar"
  version: "1.2.0"
  estado: "stable"                # draft | stable | deprecated | archived

  descripcion:
    corta: "Gestión de calendario y eventos"
    larga: "Permite crear, modificar, eliminar y consultar eventos..."

  responsabilidad: "Gestionar eventos de calendario con soporte para zonas horarias"

  dependencias:
    capabilities:
      - nombre: "Auth"
        version: ">=2.0.0"
        tipo: "requerido"
    engines:
      - nombre: "Sandbox Engine"
        version: ">=1.0.0"
        tipo: "requerido"
    sistema:
      - "Base de datos SQL"
      - "Servicio de tiempo (NTP)"

  entradas:
    eventos:
      - nombre: "crear_evento"
        descripcion: "Crear un nuevo evento en el calendario"
        entrada:
          - "titulo: string"
          - "fecha_inicio: datetime"
          - "fecha_fin: datetime"
          - "zona_horaria: string"
          - "participantes: string[]"
        salida:
          - "evento_id: uuid"
          - "estado: string"

  contratos:
    api:
      protocolo: "REST"
      base_path: "/api/v1/calendar"
      endpoints:
        - metodo: "POST"
          path: "/events"
          descripcion: "Crear evento"
        - metodo: "GET"
          path: "/events/{id}"
          descripcion: "Obtener evento"
        - metodo: "PUT"
          path: "/events/{id}"
          descripcion: "Actualizar evento"
        - metodo: "DELETE"
          path: "/events/{id}"
          descripcion: "Eliminar evento"

  archivos:
    estructura:
      - path: "calendar.md"
        obligatorio: true
      - path: "calendar_design.md"
        obligatorio: true
      - path: "calendar_api.md"
        obligatorio: false
      - path: "calendar_tests.md"
        obligatorio: true
      - path: "calendar_memory.md"
        obligatorio: true
      - path: "calendar_metrics.md"
        obligatorio: true
      - path: "calendar_release.md"
        obligatorio: false
      - path: "src/"
        obligatorio: false
      - path: "tests/"
        obligatorio: false

  metricas:
    - nombre: "cobertura_tests"
      descripcion: "Porcentaje de cobertura de tests"
      esperado: ">= 80%"
    - nombre: "tiempo_respuesta_p95"
      descripcion: "Percentil 95 de tiempo de respuesta"
      esperado: "< 200ms"
    - nombre: "tasa_error"
      descripcion: "Porcentaje de errores sobre total de operaciones"
      esperado: "< 0.1%"

  nivel_srie_requerido: "L2"

  tags:
    - "calendario"
    - "eventos"
    - "core"

  mantenedores:
    - "Discovery Engine"

  changelog:
    - version: "1.2.0"
      fecha: "2026-07-15"
      cambios:
        - "Añadido soporte para zonas horarias"
        - "Corregido bug en recurrencia semanal"
```

---

## Archivos de una Capability (detalle)

### 1. `<nombre>.md` — Especificación funcional

```markdown
# Calendar — Especificación Funcional

## Propósito
[¿Qué problema resuelve esta capability?]

## Funcionalidades
- [Funcionalidad 1]
- [Funcionalidad 2]

## Usuarios
- [¿Quién usa esta capability?]

## Criterios de éxito
- [¿Cómo se sabe que funciona correctamente?]

## Dependencias
- [¿Qué otras capabilities necesita?]
```

### 2. `<nombre>_design.md` — Diseño técnico

```markdown
# Calendar — Diseño Técnico

## Arquitectura
[Descripción de la arquitectura interna]

## Modelo de datos
[Entidades, relaciones, campos]

## Flujo
[Diagrama de flujo o descripción]

## Decisiones técnicas
- [Decisión 1 y justificación]
- [Decisión 2 y justificación]
```

### 3. `<nombre>_api.md` — Contrato de API

```markdown
# Calendar — API Contract

## Endpoints
[Tabla de endpoints con método, path, descripción]

## Formatos
[Request/response examples en JSON]

## Errores
[Códigos de error y significados]

## Autenticación
[Cómo se autentican las peticiones]
```

### 4. `<nombre>_tests.md` — Tests

```markdown
# Calendar — Tests

## Estrategia
[Enfoque de testing]

## Tests unitarios
[Lista de tests unitarios]

## Tests de integración
[Lista de tests de integración]

## Escenarios cubiertos
[Tabla de escenarios]

## Cobertura objetivo
[Porcentaje y criterios]
```

### 5. `<nombre>_memory.md` — Memoria de decisiones

```markdown
# Calendar — Memoria de Decisiones

## ADR-001: Elección de librería de calendario
- **Contexto:** Necesitamos manejar zonas horarias
- **Decisión:** Usar lib-x en lugar de lib-y
- **Alternativas:** lib-y (menos madura), lib-z (sin soporte de zonas)
- **Justificación:** lib-x tiene soporte nativo de zonas horarias
- **Fecha:** 2026-07-15

## ADR-002: Formato de recurrencia
- **Contexto:** Eventos recurrentes (diarios, semanales, mensuales)
- **Decisión:** Usar RRULE de iCalendar
- **Alternativas:** Formato propio (más simple pero no estándar)
- **Justificación:** RRULE es estándar y permite migración futura
```

### 6. `<nombre>_metrics.md` — Métricas

```yaml
# Calendar — Métricas
metricas:
  cobertura_tests:
    actual: 85%
    objetivo: 90%
    historico:
      - fecha: "2026-06-01"
        valor: 72%
      - fecha: "2026-07-01"
        valor: 85%

  tiempo_respuesta_p95:
    actual: 180ms
    objetivo: 200ms
    historico:
      - fecha: "2026-06-01"
        valor: 250ms

  tasa_error:
    actual: 0.05%
    objetivo: 0.1%
```

### 7. `<nombre>_release.md` — Notas de release

```markdown
# Calendar — Release v1.2.0

## Fecha
2026-07-15

## Cambios
- [Feature] Soporte para zonas horarias
- [Fix] Recurrencia semanal incorrecta
- [Perf] Reducción de 250ms a 180ms en p95

## Breaking changes
- Ninguno

## Upgrade path
- Actualizar dependencia lib-x a >=2.0.0
```

---

## Ciclo de vida de una Capability

```
DRAFT ──────────── Propuesta, en diseño
    │
    ▼
STABLE ─────────── Implementada y verificada
    │
    ├── DEPRECATED ─── En desuso, no se recomienda nuevas implementaciones
    │
    └── ARCHIVED ──── Ya no se mantiene, solo lectura
```

### Reglas de transición

| Transición | Requisitos |
|------------|------------|
| DRAFT → STABLE | Todos los archivos obligatorios existen. Tests pasan. Métricas dentro del rango esperado. |
| STABLE → DEPRECATED | Existe una alternativa recomendada. Se notifica a dependientes. |
| DEPRECATED → ARCHIVED | Han pasado 2 ciclos SDOS desde la deprecación. No hay dependientes activos. |
| ARCHIVED → (ninguno) | No se puede reactivar. Debe crearse una nueva capability. |

---

## Composición de Capabilities

Una Capability puede depender de otra. Las dependencias se declaran en el manifest.

### Reglas de composición

1. **Sin dependencias circulares.** A no puede depender de B si B depende de A.
2. **Versión explícita.** Toda dependencia debe especificar versión mínima.
3. **Contrato estable.** Una Capability no puede cambiar su API sin incrementar versión major.
4. **Herencia de nivel.** Si A depende de B, y B requiere nivel L2, A debe ejecutarse en un proyecto de nivel ≥ L2.

### Ejemplo de cadena de dependencias

```
Calendar ──── depende de ──── Auth (>=2.0.0)
    │
    └───────── depende de ──── Notification (>=1.0.0)
                                   │
                                   └───────── depende de ──── Template (>=1.5.0)
```

---

## Migración de Capabilities

Las Capabilities están diseñadas para ser portables entre proyectos. Una capability que existe en el proyecto A puede migrarse al proyecto B si:

1. **El manifest es válido** y no depende de capacidades exclusivas de A
2. **El nivel SRIE de B** es ≥ al nivel requerido por la capability
3. **Las dependencias de sistema** existen en B (base de datos, servicios, etc.)
4. **Los archivos obligatorios** están completos

---

## Validación de una Capability

Cada Capability debe pasar esta validación para considerarse completa:

```
[✓] Existe <nombre>.manifest
[✓] El manifest tiene todos los campos obligatorios
[✓] Existe <nombre>.md
[✓] Existe <nombre>_design.md
[✓] Existe <nombre>_tests.md
[✓] Existe <nombre>_memory.md
[✓] Existe <nombre>_metrics.md
[✓] El nivel SRIE requerido es coherente con el proyecto
[✓] Las dependencias existen y tienen versiones compatibles
[✓] No hay dependencias circulares
[✓] La versión del manifest sigue semver
```
