# INTER-SRIE CONTRACT — Contrato Universal entre Domains

---

## ¿Qué es este documento?

INTER_SRIE_CONTRACT.md define el **contrato universal** que gobierna la comunicación entre dos o más Domains SRIE dentro de una federación. No es una API. Es un acuerdo formal que especifica qué puede pedir un Domain, qué puede publicar, qué puede consumir, qué evidencias debe adjuntar y qué nivel de confianza mínimo exige.

---

## ¿Por qué un contrato y no una API?

Una API define cómo llamar a un servicio. Un contrato define **cómo cooperan dos sistemas autónomos**.

| API | Inter-SRIE Contract |
|-----|---------------------|
| Define endpoints | Define capacidades |
| Es técnica | Es semántica |
| Síncrona | Asíncrona por defecto |
| Versionado por URL | Versionado por acuerdo |
| Sin memoria | Con memoria de confianza |
| Sin auditoría | Cada interacción deja evidencia |

---

## Estructura del contrato

```yaml
# contracts/ISC-001.yaml
inter_srie_contract:
  id: "ISC-001"
  nombre: "Lead-to-Deal"
  version: "1.0.0"
  estado: "ACTIVE"              # DRAFT | ACTIVE | SUSPENDED | TERMINATED

  partes:
    proveedor:
      domain_id: "DOM-SALES"
      nombre: "Sales SRIE"
      nivel_minimo: "L2"

    consumidor:
      domain_id: "DOM-MARKETING"
      nombre: "Marketing SRIE"
      nivel_minimo: "L2"

  capacidades:
    - id: "quote_generation"
      descripcion: "Generar cotización para un lead calificado"
      tipo: "comando"            # command | query | event_subscription

      input:
        schema: {...}             # Schema JSON o referencia
        confianza_minima: 0.70

      output:
        schema: {...}
        timeout_ms: 30000

      reglas:
        - "Solo se puede invocar si el lead está calificado (>80% fit)"
        - "El proveedor debe responder en menos de 30s"
        - "La cotización debe incluir pricing vigente"

      evidencias_requeridas:
        - "Lead qualification score ≥ 80"
        - "Contract templates actualizados"

  eventos:
    - evento: "NEW_LEAD_CAPTURED"
      producer: "DOM-MARKETING"
      consumer: "DOM-SALES"
      descripcion: "Marketing capturó un nuevo lead"
      payload:
        schema: {...}
      condiciones:
        - "Solo se envía si el lead tiene datos de contacto válidos"

    - evento: "DEAL_WON"
      producer: "DOM-SALES"
      consumer: "DOM-MARKETING"
      descripcion: "Venta cerrada exitosamente"
      payload:
        schema: {...}

  metricas:
    - nombre: "confianza_promedio"
      esperado: ">= 0.85"
      periodo: "30 días"
    - nombre: "tasa_exito_comandos"
      esperado: ">= 0.95"
    - nombre: "tiempo_respuesta_p95_ms"
      esperado: "< 5000"
    - nombre: "eventos_perdidos"
      esperado: "0"

  sanciones:
    - condicion: "confianza_promedio < 0.70 por 7 días"
      accion: "SUSPENDER contrato"
    - condicion: "tasa_exito < 0.80 por 30 días"
      accion: "REVISAR contrato"
    - condicion: "3+ eventos perdidos en 24h"
      accion: "AUDITAR conexión"

  validez:
    creado: "2026-07-15"
    expira: "2027-07-15"
    renovacion: "automática si confianza ≥ 0.85"

  firmas:
    proveedor: "hash_firma_sales"
    consumidor: "hash_firma_marketing"
    testigos:
      - domain_id: "DOM-CORE"
        firma: "hash_firma_core"
```

---

## Ciclo de vida de un contrato

```
DRAFT ──── En negociación, aún no vigente
  │
  ▼
ACTIVE ─── Vigente, ambas partes cumplen
  │
  ├──► SUSPENDED ── Temporalmente detenido por incumplimiento
  │       │
  │       ▼
  │   ACTIVE ─── Se resolvió la causa de suspensión
  │
  └──► TERMINATED ── Finalizado (por expiración, mutuo acuerdo o incumplimiento grave)
```

### Eventos del ciclo de vida

| Evento | Significado |
|--------|-------------|
| `CONTRACT_PROPOSED` | Un Domain propone un contrato a otro |
| `CONTRACT_ACCEPTED` | El otro Domain acepta el contrato |
| `CONTRACT_REJECTED` | El otro Domain rechaza el contrato |
| `CONTRACT_ACTIVATED` | El contrato entra en vigencia |
| `CONTRACT_SUSPENDED` | El contrato se suspende temporalmente |
| `CONTRACT_RESUMED` | El contrato se reanuda |
| `CONTRACT_TERMINATED` | El contrato finaliza |

---

## Negociación de contrato

```yaml
negociacion:
  pasos:
    1. "Domain A descubre que necesita una capacidad de Domain B"
    2. "Domain A consulta el catálogo de capacidades de B (en Federation Registry)"
    3. "Domain A propone un contrato (CONTRACT_PROPOSED)"
    4. "Domain B revisa:"
       - "¿A tiene nivel de madurez suficiente?"
       - "¿La capacidad solicitada está disponible?"
       - "¿Las condiciones son aceptables?"
    5. "Domain B acepta (CONTRACT_ACCEPTED) o rechaza (CONTRACT_REJECTED)"
    6. "Si acepta, ambas partes firman el contrato"
    7. "El contrato se registra en ambos Domains y en el Federation Registry"
```

---

## Tipos de capacidad en un contrato

### Command

Una solicitud que espera una respuesta (síncrona o asíncrona).

```yaml
capacidad:
  tipo: "command"
  nombre: "generate_quote"
  input:
    lead_id: string
    discount_percent: float
  output:
    quote_id: string
    total: float
    pdf_url: string
  timeout_ms: 30000
```

### Query

Una consulta de estado (siempre síncrona, siempre read-only).

```yaml
capacidad:
  tipo: "query"
  nombre: "get_lead_status"
  input:
    lead_id: string
  output:
    status: string
    confidence: float
    last_updated: string
```

### Event Subscription

Un Domain se suscribe a eventos de otro Domain.

```yaml
capacidad:
  tipo: "event_subscription"
  evento: "DEAL_WON"
  condiciones:
    - "Solo deals con monto > $10,000"
    - "Incluir deal_id y customer_name"
  entrega: "BUS (ver BUS.md)"
```

---

## Confianza en el contrato

Cada interacción entre Domains registra un nivel de confianza:

```yaml
confianza_interaccion:
  formula: "Confianza = (cumplimiento_contrato × 0.5) + (calidad_evidencia × 0.3) + (tiempo_respuesta × 0.2)"
  donde:
    cumplimiento_contrato: "¿La respuesta cumple el esquema acordado?"
    calidad_evidencia: "¿Las evidencias adjuntas tienen calidad suficiente?"
    tiempo_respuesta: "¿Respondió dentro del timeout?"
```

Cada Domain mantiene una **matriz de confianza** por cada contrato activo:

```yaml
matriz_confianza:
  contrato: "ISC-001"
  proveedor: "DOM-SALES"
  consumidor: "DOM-MARKETING"
  interacciones_ultimos_30_dias: 150
  confianza_promedio: 0.92
  confianza_minima: 0.45
  tendencia: "mejorando"
```

---

## Resolución de disputas

Cuando dos Domains tienen un desacuerdo sobre el cumplimiento del contrato:

```yaml
disputa:
  niveles:
    - nivel: 1
      nombre: "Automático"
      descripcion: "El sistema detecta la anomalía y reintenta"
      accion: "Reintentar hasta 3 veces con backoff exponencial"

    - nivel: 2
      nombre: "Con evidencia"
      descripcion: "Ambos Domains presentan su evidencia"
      accion: "Comparar evidencia de ambas partes. Si hay contradicción, usar evidencia con mayor confidence score"

    - nivel: 3
      nombre: "Árbitro"
      descripcion: "Un tercer Domain actúa como árbitro"
      accion: "SRIE Core (o un Domain designado) revisa ambas evidencias y dictamina"

    - nivel: 4
      nombre: "Humano"
      descripcion: "Intervención humana requerida"
      accion: "Se genera un reporte de disputa para revisión humana"
```

---

## Reglas de los contratos

1. **Todo contrato debe tener al menos una capacidad o evento definido.** No existen contratos vacíos.

2. **El nivel de madurez mínimo es L2.** Un Domain en L0 o L1 no puede firmar contratos de federación.

3. **El contrato expira.** No existen contratos perpetuos. La expiración máxima es 1 año.

4. **Cada interacción deja evidencia.** No hay interacciones no registradas.

5. **La confianza se mide continuamente.** Si cae por debajo del umbral, el contrato se suspende automáticamente.

6. **Un Domain puede tener múltiples contratos** con diferentes Domains, incluso sobre la misma capacidad.
