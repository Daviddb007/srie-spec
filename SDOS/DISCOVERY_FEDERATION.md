# DISCOVERY FEDERATION — Discovery entre Domains Federados

---

## ¿Qué es este documento?

DISCOVERY_FEDERATION.md define **cómo funciona Discovery en una federación de SRIEs**. No reemplaza a DISCOVERY_ENGINE.md ni a DISCOVERY_PROTOCOL.md. Los extiende para el contexto federado: un Domain descubre no solo su propio proyecto, sino también **la red, los otros Domains y sus capacidades**.

---

## Los tres niveles de Discovery en una federación

```
DISCOVERY FEDERADO
├── Nivel 1: Auto-descubrimiento
│     (Discovery local del Domain)
│
├── Nivel 2: Descubrimiento de red
│     (Descubrir otros Domains en la federación)
│
└── Nivel 3: Descubrimiento de capacidades
│     (Descubrir qué capacidades ofrece cada Domain)
```

### Nivel 1: Auto-descubrimiento

Es el Discovery estándar definido en DISCOVERY_ENGINE.md. Cada Domain ejecuta su propio Discovery para conocer su estado interno.

**No cambia nada.** Cada Domain sigue siendo un SRIE completo con su propio Discovery.

### Nivel 2: Descubrimiento de red

Cuando un Domain se conecta a la federación, descubre:

- Qué otros Domains existen
- Su estado (ACTIVE, IDLE, DEGRADED, OFFLINE)
- Su nivel de madurez
- Su versión
- Su última actividad

```yaml
network_discovery:
  mecanismos:
    - "Broadcast local (ver NETWORK.md)"
    - "Federation Registry"
    - "Eventos DOMAIN_JOINED / DOMAIN_LEFT"

  salida: "FEDERATION_MAP.md"

  contenido:
    - domain_id: "DOM-SALES"
      nombre: "Sales SRIE"
      estado: "ACTIVE"
      nivel_madurez: "L3"
      version: "1.2.0"
      ultimo_heartbeat: "2026-07-15T10:00:00Z"
      ultimo_contrato: "ISC-001"
      confianza_promedio: 0.92

    - domain_id: "DOM-PRODUCT"
      nombre: "Product SRIE"
      estado: "IDLE"
      nivel_madurez: "L2"
      version: "1.0.0"
      ultimo_heartbeat: "2026-07-15T09:30:00Z"
      ultimo_contrato: null
      confianza_promedio: null
```

### Nivel 3: Descubrimiento de capacidades

Cuando un Domain necesita una capacidad que no posee, descubre qué otros Domains la ofrecen.

```yaml
capability_discovery:
  disparador: "Un Engine necesita una capacidad que no existe localmente"
  proceso:
    1. "Domain A consulta el Federation Registry"
    2. "Federation Registry devuelve lista de Domains con la capacidad solicitada"
    3. "Domain A evalúa:"
       - "¿El Domain B tiene nivel de madurez suficiente?"
       - "¿La confianza de B es aceptable?"
       - "¿Hay contratos previos entre A y B?"
    4. "Domain A solicita un Inter-SRIE Contract"
    5. "Si se acepta, Domain A puede usar la capacidad de B"

  ejemplo:
    domain_a: "DOM-MARKETING"
    necesita: "quote_generation"
    resultados:
      - domain: "DOM-SALES"
        nivel: "L3"
        confianza: 0.92
        contrato_previo: "ISC-001"
        compatible: true
      - domain: "DOM-FINANCE"
        nivel: "L2"
        confianza: 0.70
        contrato_previo: null
        compatible: false (nivel insuficiente)
```

---

## El Domain Discovery Protocol

Cuando dos Domains se encuentran por primera vez, ejecutan este protocolo:

```yaml
protocolo:
  paso_1: SALUDO
    domain_a → BUS: "SRIE_DISCOVERY {domain_id, version, capabilities}"
    domain_b → BUS: "SRIE_DISCOVERY_ACK {domain_id, version, capabilities}"

  paso_2: INTERCAMBIO DE IDENTIDAD
    domain_a → domain_b: "PUBLIC_KEY {clave_pública_a}"
    domain_b → domain_a: "PUBLIC_KEY {clave_pública_b}"

  paso_3: INTERCAMBIO DE CONTRATOS
    domain_a → domain_b: "PROPOSE_CONTRACTS [lista de contratos que A ofrece a B]"
    domain_b → domain_a: "PROPOSE_CONTRACTS [lista de contratos que B ofrece a A]"

  paso_4: NEGOCIACIÓN
    (Cada Domain decide qué contratos acepta)

  paso_5: CONEXIÓN
    domain_a → BUS: "DOMAIN_CONNECTED {domain_a, domain_b}"
    domain_b → BUS: "DOMAIN_CONNECTED {domain_b, domain_a}"
```

---

## Federation Map

Cada Domain mantiene un mapa de la federación en `SDOS/federation/FEDERATION_MAP.md`:

```yaml
# Federation Map
ultima_actualizacion: "2026-07-15T10:00:00Z"

domains:
  - id: "DOM-CORE"
    nombre: "SRIE Core"
    tipo: "kernel"
    estado: "ACTIVE"
    nivel: "L5"
    conexion: "directa"
    contratos_activos: 3
    ultimo_heartbeat: "2026-07-15T10:00:00Z"

  - id: "DOM-MARKETING"
    nombre: "Marketing SRIE"
    tipo: "domain"
    estado: "ACTIVE"
    nivel: "L3"
    conexion: "directa"
    contratos_activos: 2
    ultimo_heartbeat: "2026-07-15T10:00:00Z"

  - id: "DOM-SALES"
    nombre: "Sales SRIE"
    tipo: "domain"
    estado: "ACTIVE"
    nivel: "L3"
    conexion: "enrutada (via DOM-CORE)"
    contratos_activos: 1
    ultimo_heartbeat: "2026-07-15T09:59:00Z"

  - id: "DOM-PRODUCT"
    nombre: "Product SRIE"
    tipo: "domain"
    estado: "IDLE"
    nivel: "L2"
    conexion: "directa"
    contratos_activos: 0
    ultimo_heartbeat: "2026-07-15T09:30:00Z"

metricas_federacion:
  total_domains: 4
  activos: 3
  inactivos: 1
  contratos_totales: 6
  confianza_promedio_red: 0.88
```

---

## Discovery federado incremental

Al igual que el Discovery local, el Discovery federado es incremental:

```yaml
incremental_federado:
  evento: "DOMAIN_JOINED"
  accion: "Ejecutar Nivel 2 y Nivel 3 solo para el nuevo Domain"

  evento: "DOMAIN_UPDATED"
  accion: "Actualizar FEDERATION_MAP.md con nueva información"

  evento: "DOMAIN_LEFT"
  accion: "Marcar como DISCONNECTED en FEDERATION_MAP.md"

  evento: "CONTRACT_ACTIVATED"
  accion: "Agregar nuevo contrato al mapa de capacidades"

  evento: "CONTRACT_TERMINATED"
  accion: "Remover contrato del mapa de capacidades"
```

---

## Artefactos del Discovery federado

| Artefacto | Ruta | Descripción |
|-----------|------|-------------|
| FEDERATION_MAP.md | `SDOS/federation/FEDERATION_MAP.md` | Mapa de todos los Domains conocidos |
| CAPABILITY_CATALOG.md | `SDOS/federation/CAPABILITY_CATALOG.md` | Catálogo de capacidades disponibles en la red |
| ACTIVE_CONTRACTS.md | `SDOS/federation/ACTIVE_CONTRACTS.md` | Lista de contratos activos |
| FEDERATION_GRAPH.json | `SDOS/federation/FEDERATION_GRAPH.json` | Grafo federado en formato JSON |

### CAPABILITY_CATALOG.md

```yaml
# Capability Catalog
ultima_actualizacion: "2026-07-15T10:00:00Z"

capacidades_disponibles:
  - nombre: "lead_capture"
    description: "Capturar leads desde múltiples fuentes"
    provider: "DOM-MARKETING"
    nivel_minimo: "L2"
    contrato_requerido: true
    confianza_provider: 0.95

  - nombre: "quote_generation"
    description: "Generar cotizaciones para leads calificados"
    provider: "DOM-SALES"
    nivel_minimo: "L2"
    contrato_requerido: true
    confianza_provider: 0.92

  - nombre: "contract_management"
    description: "Gestión de contratos comerciales"
    provider: "DOM-SALES"
    nivel_minimo: "L3"
    contrato_requerido: true
    confianza_provider: 0.88
```

---

## Reglas del Discovery federado

1. **Cada Domain es responsable de su propio Discovery.** No existe un Discovery global que escanee todos los Domains.

2. **El Discovery federado solo descubre lo público.** Un Domain solo publica: su ID, su nombre, su estado, su nivel, su versión, sus capacidades públicas y su confianza. Nunca su memoria interna.

3. **El mapa federado es asíncrono.** No hay garantía de que todos los Domains tengan el mismo mapa en el mismo instante.

4. **Un Domain puede ocultarse.** Un Domain en estado STANDALONE no es visible para la red.

5. **La confianza es contextual.** La confianza que Domain A tiene de Domain B puede ser diferente de la que Domain C tiene de B.

6. **El Discovery federado no es un Engine separado.** Es una extensión del Discovery Engine que se activa cuando el Domain está en modo CONNECTED.
