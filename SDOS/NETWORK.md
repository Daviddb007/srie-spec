# NETWORK — Topología, Bus y Discovery entre Domains

---

## ¿Qué es este documento?

NETWORK.md define la **red de federación** de SRIE. Describe cómo los Domains se descubren, se conectan y se comunican entre sí formando una topología descentralizada sin punto único de fallo.

---

## Principios de la red

1. **Descentralizada** — No hay un servidor central. Cada Domain es un nodo peer.
2. **Descubrimiento espontáneo** — Los Domains se descubren entre sí sin configuración manual.
3. **Tolerante a fallos** — Si un Domain cae, la red sigue operando.
4. **Segura por contrato** — La comunicación está gobernada por Inter-SRIE Contracts.
5. **Asíncrona** — Ningún Domain espera una respuesta síncrona de otro.

---

## Topología de la red

SRIE usa una **topología de malla parcial** (partial mesh):

```
                  ┌──────────────┐
                  │  MARKETING   │
                  └──────┬───────┘
                         │
           ┌─────────────┼─────────────┐
           │             │             │
    ┌──────▼──────┐ ┌───▼──────┐ ┌────▼──────┐
    │   SALES     │ │ PRODUCT  │ │ CUSTOMER  │
    └──────┬──────┘ └───┬──────┘ └────┬──────┘
           │            │             │
           └────────────┼─────────────┘
                        │
                  ┌─────▼──────┐
                  │ ENGINEERING │
                  └─────┬──────┘
                        │
           ┌────────────┼────────────┐
           │            │            │
    ┌──────▼──────┐ ┌──▼──────┐ ┌───▼───────┐
    │   BACKEND   │ │FRONTEND │ │   DATA    │
    └─────────────┘ └─────────┘ └───────────┘
```

Cada Domain se conecta **solo a los Domains con los que necesita comunicarse**. No hay una malla completa (cada Domain con cada Domain).

### Tipos de conexión

| Tipo | Descripción | Latencia | Persistencia |
|------|-------------|----------|--------------|
| `directa` | Conexión punto a punto entre dos Domains | Baja | Persistente |
| `enrutada` | A través de un Domain intermediario | Media | Persistente |
| `eventual` | A través del BUS (cola de eventos) | Alta | Durable |

---

## Discovery de Domains

### Discovery local (misma máquina/red local)

```yaml
discovery_local:
  mecanismo: "Broadcast en la red local o multicast DNS"
  puerto: 47123
  protocolo: "UDP"
  formato_mensaje:
    tipo: "SRIE_DISCOVERY"
    domain_id: "DOM-MARKETING"
    domain_name: "Marketing SRIE"
    version: "1.0.0"
    capabilities: ["lead_capture", "campaign_analytics"]
    puerto_bus: 47124
    timestamp: "2026-07-15T10:00:00Z"
    firma: "hash_firma"

  respuesta:
    tipo: "SRIE_DISCOVERY_ACK"
    domain_id: "DOM-SALES"
    domain_name: "Sales SRIE"
    version: "1.2.0"
    capabilities: ["quote_generation", "contract_management"]
    puerto_bus: 47125
```

### Discovery remoto (distintas redes)

```yaml
discovery_remoto:
  mecanismo: "Federation Registry público o DNS compartido"
  formato: "Lista de Domains conocidos con sus direcciones"
  actualizacion: "Cada Domain publica su dirección al conectarse"

  registry_remoto:
    tipo: "Archivo compartido (HTTP/S3) | DNS SRV | Blockchain ligero"
    contenido:
      - domain_id: "DOM-MARKETING"
        direccion: "srie://marketing.empresa.com:47123"
        public_key: "..."   # Para verificación de identidad
```

### Discovery por eventos

```yaml
discovery_eventos:
  mecanismo: "Un Domain emite un evento JOINED_FEDERATION y los demás responden"
  evento: "DOMAIN_JOINED"
  payload:
    domain_id: "DOM-NUEVO"
    domain_name: "Nuevo Domain"
    capabilities: ["cap1", "cap2"]
```

---

## Heartbeat y monitoreo

Cada Domain emite un heartbeat periódico para indicar que sigue vivo.

```yaml
heartbeat:
  intervalo_segundos: 30
  timeout_segundos: 90       # Si no se recibe en 90s, se marca como DEGRADADO
  timeout_total_segundos: 300 # Si no se recibe en 5min, se marca como OFFLINE

  mensaje:
    tipo: "HEARTBEAT"
    domain_id: "DOM-MARKETING"
    timestamp: "2026-07-15T10:00:00Z"
    estado: "ACTIVE"
    confianza_promedio: 0.92
    load:
      ciclos_sdos_activos: 1
      eventos_procesados_ultimo_minuto: 15
```

---

## Canal de comunicación (BUS)

La comunicación entre Domains ocurre a través del BUS (ver BUS.md). Cada Domain mantiene:

1. **Un canal de salida** para publicar eventos
2. **Uno o más canales de entrada** para recibir eventos de otros Domains
3. **Un canal de control** para heartbeats y discovery

### Formato del canal

```yaml
canal:
  tipo: "TCP | WebSocket | Message Queue"
  formato_mensaje: "JSON sobre el protocolo de BUS"
  autenticacion: "Firma HMAC con clave compartida o certificado"
  cifrado: "TLS 1.3+"
```

---

## Grafo global de la federación

El Global Knowledge Graph es la unificación de los Knowledge Graphs de todos los Domains federados.

```yaml
global_graph:
  contenido:
    - "Entidades compartidas (Domains, capabilities públicas)"
    - "Relaciones entre Domains"
    - "Eventos históricos relevantes"
    - "Contratos activos entre Domains"

  NO_contiene:
    - "Memoria interna de cada Domain"
    - "Evidencia local de cada Domain"
    - "Decisiones internas no publicadas"

  actualizacion:
    - "Cada Domain publica cambios en su grafo público"
    - "La federación mergea los cambios"
    - "Si hay conflictos (dos Domains afirman lo mismo), el Federation Registry decide"
```

---

## Seguridad de la red

```yaml
seguridad:
  autenticacion:
    mecanismo: "Par de claves (pública/privada) por Domain"
    registro: "La clave pública se almacena en el Federation Registry"

  autorizacion:
    mecanismo: "Inter-SRIE Contract define qué puede hacer cada Domain"
    validacion: "Cada mensaje se valida contra el contrato activo"

  cifrado:
    canal: "TLS 1.3+"
    mensaje: "Firma HMAC en cada mensaje"

  identidad:
    formato_id: "DOM-<AREA>-<NUMERO>"
    ejemplo: "DOM-MARKETING-001"
```

---

## Reglas de la red

1. **Cada Domain es un peer.** No hay servidores centrales ni clientes.

2. **El descubrimiento es automático.** No se requiere configuración manual para que dos Domains en la misma red se encuentren.

3. **La conexión es opt-in.** Un Domain descubierto no está automáticamente conectado. Debe aceptar la conexión.

4. **El heartbeat es obligatorio.** Si un Domain deja de emitir heartbeat, se marca como OFFLINE.

5. **El grafo global no reemplaza al grafo local.** Cada Domain mantiene su propio grafo y publica solo lo que el contrato permite.

6. **La red no tiene memoria.** Los eventos pasados no se almacenan en la red. Cada Domain es responsable de su propia memoria.
