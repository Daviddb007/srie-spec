# BUS — Sistema de Mensajería entre Domains

---

## ¿Qué es este documento?

BUS.md define el **sistema de mensajería** que permite la comunicación entre Domains SRIE dentro de una federación. No es un Engine. Es la **infraestructura de comunicación** — el canal por el que fluyen eventos, comandos y consultas entre Domains autónomos.

---

## Filosofía del BUS

1. **Todo pasa por el BUS.** No hay comunicación directa entre Domains fuera del BUS.
2. **Asíncrono por defecto.** Los Domains nunca esperan una respuesta síncrona de otro Domain (excepto queries explícitas).
3. **Durable.** Los mensajes persisten aunque el Domain receptor esté caído.
4. **Gobernado por contratos.** Solo se pueden intercambiar mensajes definidos en un Inter-SRIE Contract activo.
5. **Auditable.** Cada mensaje deja evidencia en ambos Domains.

---

## Tipos de mensajes

### 1. EVENT

Notificación de que algo ocurrió. No espera respuesta.

```yaml
message:
  tipo: "EVENT"
  id: "evt_001"
  event_type: "NEW_LEAD_CAPTURED"
  producer: "DOM-MARKETING"
  timestamp: "2026-07-15T10:00:00Z"
  payload:
    lead_id: "LD-001"
    name: "Empresa X"
    source: "website"
    score: 85
  contract_id: "ISC-001"
  confidence: 0.95
  signature: "..."              # Firma HMAC

reglas:
  - "No espera respuesta"
  - "Se entrega a todos los suscriptores del evento"
  - "Si no hay suscriptores, el evento se descarta (fire-and-forget)"
```

### 2. COMMAND

Solicitud de acción que espera una respuesta (asíncrona).

```yaml
message:
  tipo: "COMMAND"
  id: "cmd_001"
  command: "generate_quote"
  consumer: "DOM-SALES"
  producer: "DOM-MARKETING"
  timestamp: "2026-07-15T10:00:00Z"
  payload:
    lead_id: "LD-001"
    discount_percent: 10
  contract_id: "ISC-001"
  confidence: 0.88
  timeout_ms: 30000
  reply_to: "DOM-MARKETING"

# Respuesta (mensaje separado)
message:
  tipo: "COMMAND_RESPONSE"
  id: "cmd_001_resp"
  command_id: "cmd_001"
  producer: "DOM-SALES"
  consumer: "DOM-MARKETING"
  timestamp: "2026-07-15T10:00:01Z"
  status: "SUCCESS"             # SUCCESS | FAILED | TIMEOUT
  payload:
    quote_id: "QTE-001"
    total: 15000.00
    pdf_url: "srie://dom-sales/quotes/QTE-001.pdf"
  confidence: 0.97

reglas:
  - "Espera una respuesta COMMAND_RESPONSE"
  - "Timeout definido en el contrato"
  - "Si timeout, el consumidor debe reintentar según el contrato"
```

### 3. QUERY

Consulta de estado (síncrona, read-only).

```yaml
message:
  tipo: "QUERY"
  id: "qry_001"
  query: "get_lead_status"
  consumer: "DOM-SALES"
  producer: "DOM-MARKETING"
  timestamp: "2026-07-15T10:00:00Z"
  payload:
    lead_id: "LD-001"
  contract_id: "ISC-001"
  max_wait_ms: 5000

# Respuesta
message:
  tipo: "QUERY_RESPONSE"
  id: "qry_001_resp"
  query_id: "qry_001"
  producer: "DOM-MARKETING"
  consumer: "DOM-SALES"
  timestamp: "2026-07-15T10:00:00.5Z"
  status: "SUCCESS"
  payload:
    status: "qualified"
    score: 85
    last_contact: "2026-07-14"
  confidence: 0.92

reglas:
  - "No modifica estado"
  - "Debe responder en menos de max_wait_ms"
  - "Si timeout, el consumidor decide qué hacer"
```

### 4. HEARTBEAT

Mensaje de mantenimiento (ver NETWORK.md).

---

## Canales del BUS

```yaml
canales:
  - nombre: "events"
    tipo: "pub/sub"
    proposito: "Distribución de eventos a suscriptores"
    patron: "fire-and-forget"
    durabilidad: "ninguna (si no hay suscriptor, se descarta)"

  - nombre: "commands"
    tipo: "cola punto-a-punto"
    proposito: "Entrega de comandos a un Domain específico"
    patron: "request/response asíncrono"
    durabilidad: "persistente (el mensaje espera hasta que el consumidor lo reciba)"

  - nombre: "queries"
    tipo: "request/response síncrono"
    proposito: "Consultas de estado entre Domains"
    patron: "request/response con timeout"
    durabilidad: "ninguna (solo vive durante la petición)"

  - nombre: "control"
    tipo: "pub/sub"
    proposito: "Heartbeats, discovery, mensajes de control"
    patron: "fire-and-forget"
    durabilidad: "ninguna"
```

---

## Formato universal del mensaje

```json
{
  "protocol_version": "1.0",
  "message_id": "uuid",
  "message_type": "EVENT | COMMAND | QUERY | HEARTBEAT | CONTROL",
  "timestamp": "ISO8601",
  "producer": {
    "domain_id": "string",
    "instance_id": "string"
  },
  "consumer": {
    "domain_id": "string | null",    // null = broadcast
    "contract_id": "string | null"
  },
  "payload": {},
  "metadata": {
    "confidence": 0.0,
    "retry_count": 0,
    "signature": "string",
    "trace_id": "string"
  }
}
```

---

## Entrega y garantías

| Tipo | Garantía | Reintento | Persistencia |
|------|----------|-----------|--------------|
| EVENT | At most once | No | No |
| COMMAND | At least once | Sí (hasta 3) | Sí (en cola) |
| QUERY | At most once | No | No |
| HEARTBEAT | At most once | No | No |

### Estrategia de reintentos

```yaml
reintentos:
  command:
    max_attempts: 3
    backoff: "exponencial (1s, 4s, 9s)"
    accion_si_agota: "Emite COMMAND_FAILED con timeout"

  event:
    max_attempts: 1
    accion_si_falla: "Emite EVENT_DELIVERY_FAILED"
```

---

## Suscripción a eventos

Los Domains se suscriben a eventos de otros Domains mediante contratos:

```yaml
suscripcion:
  domain: "DOM-SALES"
  suscripciones:
    - evento: "NEW_LEAD_CAPTURED"
      producer: "DOM-MARKETING"
      contrato: "ISC-001"
      filtro:
        - "lead.score >= 80"
        - "lead.source in ['website', 'referral']"
      canal: "events"
```

---

## Enrutamiento

```yaml
enrutamiento:
  local:
    - "Si producer y consumer están en el mismo Domain, el mensaje no sale al BUS"
    - "Se entrega directamente al Engine correspondiente"

  federado:
    - "Si producer y consumer están en distintos Domains, el mensaje pasa por el BUS"
    - "El BUS consulta el Federation Registry para encontrar la dirección del consumer"
    - "Entrega directa si los Domains están en conexión directa"
    - "Entrega enrutada si hay un Domain intermedio"

  broadcast:
    - "Si consumer es null, el mensaje se entrega a todos los Domains suscritos"
    - "Cada Domain decide si procesa o ignora según sus contratos"
```

---

## Seguridad del BUS

```yaml
seguridad:
  autenticacion:
    - "Cada mensaje incluye firma HMAC del producer"
    - "El consumer verifica la firma contra la clave pública del producer"

  autorizacion:
    - "Solo se procesan mensajes de contratos activos"
    - "Mensajes sin contrato válido se rechazan (RESPONSE: UNAUTHORIZED)"

  integridad:
    - "Firma HMAC-SHA256 sobre el payload + metadata"
    - "Si la firma no coincide, el mensaje se descarta"

  no_repudio:
    - "Cada mensaje recibido se registra en la evidencia del consumer"
    - "Cada mensaje enviado se registra en la evidencia del producer"
```

---

## Monitoreo del BUS

```yaml
monitoreo:
  metricas:
    - "mensajes_por_segundo"
    - "tasa_entrega_exitosa"
    - "latencia_promedio_ms"
    - "cola_comandos_actual"
    - "eventos_perdidos"
    - "contratos_activos"
    - "domains_conectados"

  alertas:
    - condicion: "tasa_entrega < 0.95"
      severidad: "warning"
    - condicion: "cola_comandos > 1000"
      severidad: "critical"
    - condicion: "domain_offline > 5min"
      severidad: "warning"
```

---

## Ciclo de vida de un mensaje en el BUS

```
1. PRODUCER crea el mensaje
2. PRODUCER firma el mensaje (HMAC)
3. PRODUCER registra el mensaje en su evidencia local
4. Mensaje entra al BUS
5. BUS verifica: contrato activo?, consumer existe?, firma válida?
6. BUS enruta al consumer (directo o enrutado)
7. CONSUMER recibe el mensaje
8. CONSUMER verifica: contrato activo?, firma válida?, confianza suficiente?
9. CONSUMER procesa el mensaje
10. CONSUMER registra el mensaje en su evidencia local
11. CONSUMER envía respuesta (si aplica: COMMAND_RESPONSE o QUERY_RESPONSE)
```

### Registro de evidencia

Cada Domain mantiene un log de todos los mensajes del BUS que le conciernen:

```yaml
# SDOS/evidence/bus/<fecha>.jsonl
{"ts":"...","type":"EVENT","id":"evt_001","direction":"SENT","status":"DELIVERED"}
{"ts":"...","type":"COMMAND","id":"cmd_001","direction":"RECEIVED","status":"PROCESSED"}
{"ts":"...","type":"QUERY","id":"qry_001","direction":"SENT","status":"TIMEOUT"}
```
