# FEDERATION — Cómo cooperan múltiples SRIEs

---

## ¿Qué es este documento?

FEDERATION.md define **el modelo de federación** del sistema SRIE. Describe cómo múltiples instancias de SRIE (Domains) cooperan entre sí formando una red descentralizada, autónoma y gobernada por contratos.

Una federación SRIE no es un monolito. Es una **red de sistemas operativos de ingeniería** que operan independientemente pero se coordinan mediante eventos, contratos y un grafo de conocimiento compartido.

---

## ¿Por qué federación?

Un solo SRIE puede gobernar un proyecto de software. Pero una organización completa tiene múltiples dominios que necesitan coordinarse:

```
Marketing  ────  Ventas  ────  Producto  ────  Ingeniería
   │                                                  │
   ▼                                                  ▼
Customer  ────  Legal  ────  Finanzas  ────  Operaciones
```

Cada dominio tiene su propia realidad, su propia memoria y sus propias capacidades. Pero comparten objetivos, datos y procesos. Una federación permite que cada dominio opere con autonomía mientras se mantiene la coherencia del conjunto.

---

## Principios de la federación

### 1. Autonomía

Todo SRIE Domain debe ser capaz de ejecutarse completamente solo:

- Arrancar sin dependencias externas
- Operar su ciclo SDOS completo
- Generar y mantener su propia memoria
- Evolucionar su nivel de madurez
- Producir y consumir sus propios artefactos

### 2. Federación

Cuando un Domain se conecta a la red, debe ser capaz de:

- Descubrir otros Domains en la federación
- Intercambiar eventos con ellos
- Solicitar capacidades que no posee
- Compartir evidencia mediante contratos definidos
- Sincronizar su grafo con el grafo global

### 3. Fractalidad

El mismo patrón funciona a cualquier escala:

```
SRIE Empresa
  ├── SRIE Tecnología
  │     ├── SRIE Backend
  │     │     ├── SRIE Auth
  │     │     └── SRIE API
  │     ├── SRIE Frontend
  │     └── SRIE Data
  ├── SRIE Marketing
  ├── SRIE Ventas
  └── SRIE Finanzas
```

Cada nodo es un SRIE completo. No hay una versión "Enterprise" diferente. La escalabilidad vertical se logra conectando más nodos que hablan el mismo protocolo.

### 4. Aislamiento de memoria

Los Domains **nunca comparten memoria**. Cada Domain tiene:

```
SU_MEMORIA (ADR, context, evolution)
SU_REGISTRY (entidades locales)
SUS_CAPABILITIES (capacidades locales)
SUS_INDICATORS (métricas locales)
```

Lo único compartido es:

```
KNOWLEDGE_GRAPH (grafo global de la federación)
FEDERATION_REGISTRY (entidades federadas)
NETWORK (bus de eventos)
```

Esto evita contaminación de memoria entre dominios.

### 5. Comunicación por eventos

Los Domains no se llaman directamente. Se comunican mediante eventos a través del BUS (ver BUS.md). No hay APIs directas entre dominios. No hay dependencias de ejecución síncrona.

---

## Arquitectura de la federación

```
┌─────────────────────────────────────────────────────────────────────┐
│                        SRIE FEDERATION                              │
│                                                                      │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                 │
│  │ SRIE CORE   │  │ SRIE MARKET │  │  SRIE SALES │  ...             │
│  │ (kernel)    │  │ (domain)    │  │  (domain)   │                 │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘                 │
│         │                │                │                         │
│         └────────────────┼────────────────┘                         │
│                          │                                          │
│                    ┌─────▼──────┐                                   │
│                    │    BUS     │  (Event / Command / Query)         │
│                    └─────┬──────┘                                   │
│                          │                                          │
│          ┌───────────────┼───────────────┐                          │
│          │               │               │                          │
│   ┌──────▼──────┐ ┌──────▼──────┐ ┌──────▼──────┐                  │
│   │ FEDERATION   │ │   GLOBAL    │ │ FEDERATION  │                  │
│   │  REGISTRY    │ │ KNOWLEDGE   │ │  NETWORK    │                  │
│   │              │ │   GRAPH     │ │  DISCOVERY  │                  │
│   └─────────────┘ └─────────────┘ └─────────────┘                  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Componentes de la federación

### SRIE Core
El kernel de la federación. Proporciona:
- Ciclo SDOS base
- Discovery local
- Indicators locales
- Registry local
- Ontología

### SRIE Domain
Una instancia especializada de SRIE que opera sobre un dominio de negocio específico (Marketing, Ventas, Producto, etc.). Cada Domain:
- Tiene su propio SDOS
- Tiene sus propios Engines (puede especializarlos)
- Tiene su propia memoria
- Tiene sus propias capabilities
- Se conecta a la federación mediante un adaptador de red

### BUS
Canal de comunicación entre Domains (ver BUS.md). Soportes:
- Eventos (notificaciones asíncronas)
- Comandos (solicitudes con respuesta)
- Queries (consultas de estado)

### Federation Registry
Registro global de todos los Domains en la federación:

```yaml
federation_registry:
  domains:
    - id: "DOM-MARKETING"
      nombre: "Marketing SRIE"
      version: "1.0.0"
      estado: "ACTIVE"
      nivel_madurez: "L3"
      capabilities_publicadas:
        - "lead_capture"
        - "campaign_analytics"
      eventos_que_consume:
        - "NEW_PRODUCT_LAUNCHED"
        - "SALES_TARGET_CHANGED"
      eventos_que_produce:
        - "NEW_LEAD_CAPTURED"
        - "CAMPAIGN_COMPLETED"
      ultimo_heartbeat: "2026-07-15T10:00:00Z"

    - id: "DOM-SALES"
      nombre: "Sales SRIE"
      version: "1.2.0"
      estado: "ACTIVE"
      nivel_madurez: "L2"
      capabilities_publicadas:
        - "quote_generation"
        - "contract_management"
      eventos_que_consume:
        - "NEW_LEAD_CAPTURED"
        - "PRODUCT_UPDATED"
      eventos_que_produce:
        - "DEAL_WON"
        - "DEAL_LOST"
        - "CONTRACT_SIGNED"
```

### Global Knowledge Graph
Grafo que unifica los Knowledge Graphs de todos los Domains. No contiene la memoria interna de cada Domain, solo las entidades y relaciones que son relevantes para la federación (ver NETWORK.md).

### Federation Network Discovery
Mecanismo por el cual los Domains se descubren entre sí al unirse a la federación (ver DISCOVERY_FEDERATION.md).

---

## Ciclo de vida de un Domain en la federación

```
STANDALONE ────── Domain operando de forma independiente
    │
    ▼
DISCOVERED ────── Domain detectado por la federación
    │
    ▼
REGISTERED ────── Domain registrado en Federation Registry
    │
    ▼
CONNECTED ─────── Domain conectado al BUS
    │
    ├──► ACTIVE ──── Intercambiando eventos activamente
    │
    ├──► IDLE ────── Conectado pero sin actividad reciente
    │
    └──► DEGRADED ── Conectado con confianza baja
    │
    ▼
DISCONNECTED ──── Domain desconectado de la federación
    │
    ▼
ARCHIVED ──────── Domain eliminado del registro federado
```

---

## Niveles de integración federada

| Nivel | Nombre | Descripción |
|-------|--------|-------------|
| L0 | Aislado | Domain standalone. No se comunica con nadie. |
| L1 | Conectado | Domain registrado en la federación. Publica eventos. |
| L2 | Cooperativo | Domain consume eventos de otros Domains y reacciona. |
| L3 | Sincronizado | Domain sincroniza su grafo con el Global Knowledge Graph. |
| L4 | Contratado | Domain establece Inter-SRIE Contracts con otros Domains. |
| L5 | Autónomo | Domain puede delegar capacidades y recibir delegaciones automáticamente. |

---

## Reglas de la federación

1. **Un Domain es autónomo o no es un Domain.** Si depende de otro para operar, no es un Domain. Es un submódulo.

2. **La memoria es privada del Domain.** Ningún otro Domain puede leer la memoria interna de otro sin un contrato explícito.

3. **Los eventos son públicos.** Cualquier Domain suscrito puede recibir eventos de cualquier otro Domain.

4. **No hay llamadas directas.** Toda comunicación pasa por el BUS.

5. **El Global Knowledge Graph es de solo lectura** para los Domains individuales. Solo la federación lo escribe.

6. **Un Domain puede abandonar la federación en cualquier momento** y conservar toda su memoria y capacidades.

7. **La federación no tiene un punto único de fallo.** Si un Domain cae, el resto sigue operando.
