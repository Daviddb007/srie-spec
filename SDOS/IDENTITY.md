# IDENTITY — Sistema de Identidad del Ecosistema SRIE

---

## ¿Qué es este documento?

IDENTITY.md define el **sistema de identidad** de SRIE. Responde a la pregunta más fundamental que cualquier nodo del ecosistema debe responder antes de actuar:

> **¿Quién soy?**

En una federación de múltiples SRIEs — Empresa, Marketing, Ventas, Clientes, Laboratorios — la identidad es lo que permite que cada nodo sepa quién es, qué puede hacer, con quién puede hablar y qué reglas debe seguir.

---

## Las 5 preguntas de identidad

Antes de que cualquier Engine ejecute cualquier acción, el nodo debe responder:

```
1. ¿QUIÉN SOY?
   → domain_id, nombre, tipo, versión

2. ¿DÓNDE ESTOY?
   → red, federación, organización, posición en el grafo

3. ¿QUIÉN ME GOBIERNA?
   → autoridad superior, constitución, charter

4. ¿QUÉ PERMISOS TENGO?
   → qué puedo crear, modificar, registrar, eliminar

5. ¿QUÉ MEMORIA USO?
   → registry local/federado, graph local/global, memoria propia
```

---

## Identity Manifest

Cada Domain tiene un archivo `SDOS/IDENTITY.yaml` que es su documento de identidad:

```yaml
# SDOS/IDENTITY.yaml
identity:
  domain_id: "DOM-EMP-MARKETING-001"
  nombre: "Marketing SRIE"
  tipo: "domain"                    # core | federation | organization | domain | capability
  version: "1.0.0"
  estado: "ACTIVE"                  # STANDALONE | CONNECTED | FEDERATED

  organization:
    id: "ORG-001"
    nombre: "Empresa SRIE"

  parent: "DOM-EMP-001"            # Domain padre
  hijos: []                         # Sub-Domains

  authority:
    nivel: "Domain"
    gobernado_por: "DOM-EMP-001"
    puede_crear_domains: false
    puede_crear_capabilities: true
    puede_modificar_registry: "local"    # local | federation | none
    puede_modificar_ontology: false

  federation:
    network_id: "NET-EMP-001"
    registry: "federation"          # local | federation | global
    graph: "federado"               # local | federado | global
    memory: "local"                 # local (nunca compartida)
    knowledge: "compartido"         # local | compartido

  mission: "Gestionar campañas de marketing, captura de leads y analítica de mercado"
  charter: "SDOS/charter/MARKETING_CHARTER.md"

  contracts:
    activos: ["ISC-001", "ISC-003"]
    historico: ["ISC-000"]

  created: "2026-07-15T10:00:00Z"
  updated: "2026-07-15T10:00:00Z"
  fingerprint: "sha256:abc123..."   # Hash de la identidad completa
```

---

## Identity Chain

La identidad forma una cadena de confianza desde el Core hasta la Capability más pequeña:

```
SRIE CORE (identidad raíz)
  │
  ├── Firma: clave raíz del sistema
  │
  ▼
FEDERATION (identidad de red)
  │
  ├── Firma: clave de la federación
  │
  ▼
ORGANIZATION (identidad de organización)
  │
  ├── Firma: clave de la organización
  │
  ▼
DOMAIN (identidad de dominio)
  │
  ├── Firma: clave del domain
  │
  ▼
CAPABILITY (identidad de capacidad)
      │
      └── Firma: clave del domain propietario
```

Cada nivel firma al siguiente. Esto crea una cadena de confianza verificable:

```yaml
identity_chain:
  - nivel: "CORE"
    id: "SRIE-CORE-001"
    firma: "auto-firmada (raíz de confianza)"

  - nivel: "FEDERATION"
    id: "NET-EMP-001"
    firmado_por: "SRIE-CORE-001"
    firma: "..."

  - nivel: "ORGANIZATION"
    id: "ORG-001"
    firmado_por: "NET-EMP-001"
    firma: "..."

  - nivel: "DOMAIN"
    id: "DOM-EMP-MARKETING-001"
    firmado_por: "ORG-001"
    firma: "..."

  - nivel: "CAPABILITY"
    id: "CAP-lead-capture-001"
    firmado_por: "DOM-EMP-MARKETING-001"
    firma: "..."
```

---

## Ciclo de vida de la identidad

```yaml
identity_lifecycle:
  estados:
    - "PENDING"       # Identidad solicitada, no aprobada
    - "ACTIVE"        # Identidad vigente
    - "SUSPENDED"     # Identidad suspendida temporalmente
    - "REVOKED"       # Identidad revocada (no puede volver a ACTIVO)
    - "EXPIRED"       # Identidad expirada

  transiciones:
    PENDING → ACTIVE:   "Aprobación por autoridad superior"
    ACTIVE → SUSPENDED: "Violación de gobernanza o confianza baja"
    SUSPENDED → ACTIVE: "Resolución de la causa de suspensión"
    ACTIVE → REVOKED:   "Violación grave o decisión de autoridad superior"
    ACTIVE → EXPIRED:   "Tiempo de vida superado"
```

---

## Identity Engine

No es uno de los 11 Engines del ciclo SDOS. Es un **servicio del kernel** que todo Engine consulta antes de actuar.

```yaml
identity_engine:
  tipo: "servicio_kernel"
  disponible_para: "todos los Engines"

  metodos:
    - nombre: "whoami"
      descripcion: "Devuelve la identidad completa del Domain actual"
      salida: "IDENTITY.yaml completo"

    - nombre: "get_authority"
      descripcion: "Devuelve el nivel de autoridad y permisos"
      salida: "nivel, permisos"

    - nombre: "validate_identity"
      descripcion: "Verifica que la cadena de firmas sea válida"
      salida: "valido: boolean, cadena: []"

    - nombre: "get_parent"
      descripcion: "Devuelve la identidad del Domain padre"
      salida: "domain_id, nombre, nivel"

    - nombre: "get_children"
      descripcion: "Devuelve las identidades de los sub-Domains"
      salida: "[domain_id, nombre, estado]"

    - nombre: "get_peers"
      descripcion: "Devuelve las identidades de los Domains pares"
      salida: "[domain_id, nombre, estado, contratos]"
```

### Uso típico por un Engine

```
1. Engine necesita ejecutar una acción
2. Engine consulta Identity Engine: whoami()
3. Identity Engine devuelve: {domain_id, tipo, permisos}
4. Engine verifica: ¿tengo permiso para esta acción?
5. Si sí → ejecutar
6. Si no → registrar evento PERMISSION_DENIED, abortar
```

---

## Identity en INIT

El proceso INIT ahora comienza con identidad:

```
IDENTITY
  │
  ├── ¿Existe SDOS/IDENTITY.yaml?
  │     │
  │     ├── NO → ¿Proyecto nuevo?
  │     │         ├── SÍ → Crear identidad, firmar, registrar
  │     │         └── NO → Solicitar identidad al federation registry
  │     │
  │     └── SÍ → Validar cadena de firmas
  │                 │
  │                 ├── VÁLIDA → Continuar a inspect
  │                 │
  │                 └── INVÁLIDA → Solicitar renovación
  │
  ▼
INSPECT → ANALYZE → BOOTSTRAP
```

```yaml
identity_en_init:
  nivel_1: "identity"
  comando: "srie identity"
  accion: |
    - Si no existe IDENTITY.yaml:
      - Generar domain_id único
      - Crear cadena de identidad
      - Firmar con clave local
      - Si hay federación, registrar en federation registry
    - Si existe IDENTITY.yaml:
      - Validar cadena de firmas
      - Verificar que no haya expirado
      - Verificar que el domain_id no esté revocado
  salida: "SDOS/IDENTITY.yaml"
  modifica: "Solo crea/actualiza SDOS/IDENTITY.yaml"
```

---

## Identity Registry

El Registry se extiende para incluir identidades:

```yaml
# SDOS/REGISTRY.yaml (extensión)
identity:
  local:
    domain_id: "DOM-EMP-MARKETING-001"
    fingerprint: "sha256:abc123..."
    ultima_validacion: "2026-07-15T10:00:00Z"
    cadena_valida: true

  federation:
    - domain_id: "DOM-EMP-SALES-001"
      fingerprint: "sha256:def456..."
      confianza: 0.92
    - domain_id: "DOM-EMP-PRODUCT-001"
      fingerprint: "sha256:ghi789..."
      confianza: 0.88
```

---

## Reglas de identidad

1. **Todo nodo tiene una identidad.** No existe un nodo anónimo en la federación.

2. **La identidad es inmutable en sus campos clave.** `domain_id`, `tipo` y `parent` no pueden cambiar después de la creación. Para cambiar de identidad, se crea una nueva.

3. **La cadena de firmas es obligatoria.** Un Domain sin cadena de firmas válida no puede unirse a la federación.

4. **La identidad expira.** El tiempo máximo de vida de una identidad es 1 año. Después debe renovarse.

5. **Un Domain solo tiene una identidad.** No puede tener múltiples identidades simultáneamente.

6. **La identidad de un Domain es verificable por cualquier otro Domain** mediante la cadena de firmas.
