# RESOURCES — Modelo de Recursos del Sistema

---

## ¿Qué es este documento?

RESOURCES.md define el **modelo de recursos** de SRIE. No solo herramientas. **Recursos**: cualquier cosa que SRIE pueda descubrir, registrar, autorizar y utilizar para ejecutar capacidades.

Un recurso puede ser:
- Un MCP Server
- Una API REST
- Una base de datos
- Un modelo de IA
- Almacenamiento
- Un sistema de colas
- Un servicio externo
- Otro SRIE federado
- Un comando CLI
- Un SDK
- Una conexión SSH

---

## La jerarquía de recursos

```
RESOURCE (contrato universal)
├── Tool
│   ├── MCP Server
│   ├── CLI Tool
│   └── SDK Library
│
├── API
│   ├── REST API
│   ├── GraphQL API
│   └── gRPC API
│
├── Service
│   ├── Database (Postgres, MySQL, etc.)
│   ├── Queue (RabbitMQ, SQS, etc.)
│   ├── Storage (S3, GCS, etc.)
│   └── AI Model (OpenAI, Anthropic, etc.)
│
├── External
│   ├── GitHub, GitLab, Bitbucket
│   ├── Jira, Linear, Notion
│   ├── Slack, Discord
│   ├── AWS, Azure, GCP
│   └── ... cualquier servicio externo
│
└── Federated
    └── Otro SRIE Domain (recurso federado)
```

---

## Formato de un recurso

```yaml
resource:
  id: "RES-GH-001"
  nombre: "GitHub API"
  tipo: "external"              # tool | api | service | external | federated
  subtipo: "mcp"                # mcp | rest | cli | sdk | ssh | graphql | grpc

  descripcion: "Acceso a repositorios GitHub mediante MCP"

  capacidad:
    accion: "gestionar_repositorio_git"
    entrada: "Crear, leer, actualizar, eliminar repositorios"
    salida: "Resultado de la operación en repositorio"

  provider: "github.com"
  protocolo: "mcp"
  version_protocolo: "2025-03-26"

  autenticacion:
    metodo: "token"              # token | oauth | key | certificate | none
    config: "GITHUB_TOKEN"

  estado: "ACTIVE"               # ACTIVE | DEGRADED | OFFLINE | UNKNOWN
  confianza: 0.95

  identity:
    domain_id: null              # null = recurso externo
    resource_type: "external"

  metricas:
    latencia_p50_ms: 230
    tasa_exito: 0.97
    ultimo_uso: "2026-07-15T10:00:00Z"

  domains_permitidos:
    - "DOM-BACKEND"
    - "DOM-DEVOPS"
  domains_prohibidos: []
```

---

## Capacidades de recursos

Cada recurso expresa su capacidad en términos abstractos para que el Resource Broker pueda resolverlos independientemente del proveedor:

```yaml
capacidades_recurso:
  - capacidad: "gestionar_repositorio_git"
    recursos_posibles:
      - "GitHub MCP"
      - "GitLab MCP"
      - "Azure DevOps"
      - "Git CLI"

  - capacidad: "enviar_mensaje"
    recursos_posibles:
      - "Slack MCP"
      - "Discord MCP"
      - "Telegram API"
      - "Email SMTP"

  - capacidad: "almacenar_archivos"
    recursos_posibles:
      - "AWS S3"
      - "Google Cloud Storage"
      - "Azure Blob"
      - "Local Filesystem"

  - capacidad: "ejecutar_modelo_ia"
    recursos_posibles:
      - "OpenAI API"
      - "Anthropic API"
      - "Local LLM"
      - "Ollama"

  - capacidad: "gestionar_tareas"
    recursos_posibles:
      - "Jira MCP"
      - "Linear MCP"
      - "Notion API"
      - "GitHub Issues"
```

---

## Ciclo de vida de un recurso

```yaml
resource_lifecycle:
  DISCOVERED: "Recurso encontrado por Tool Discovery"
    │
    ▼
  REGISTERED: "Recurso registrado en Resource Registry"
    │
    ▼
  VERIFIED: "Conexión verificada, funciona correctamente"
    │
    ▼
  ACTIVE: "Recurso disponible para uso"
    │
    ├── DEGRADED: "Funciona con problemas (latencia alta, errores intermitentes)"
    │
    ├── OFFLINE: "No responde, intentar reconectar"
    │
    └── RETIRED: "Ya no está disponible, reemplazar"
```

---

## Discovery de recursos

```yaml
tool_discovery:
  automático:
    - "Escáner de MCP servidores en la red local"
    - "Detección de variables de entorno con tokens/keys"
    - "Archivos de configuración (mcp.json, .env)"
    - "Registro de recursos del sistema operativo"

  manual:
    - "Registro explícito en RESOURCE_REGISTRY.yaml"
    - "Declaración en SDOS/resource/"

  federado:
    - "Descubrimiento a través de la federación SRIE"
    - "Un Domain publica sus recursos disponibles"
```

---

## Identity → Tools mapping

La identidad del Domain determina qué recursos tiene disponibles automáticamente:

| Domain | Recursos automáticos |
|--------|---------------------|
| Backend | GitHub, Docker, Postgres, SSH, CI/CD |
| Marketing | Canva, Meta Ads, Google Analytics, LinkedIn, HubSpot |
| Ventas | Salesforce, HubSpot, Mailchimp, Calendly |
| Legal | Word, PDF, Firma Digital, Normativa |
| Finanzas | QuickBooks, Stripe, Xero, SAP |
| Producto | Figma, Jira, Notion, Amplitude |
| Data | BigQuery, Snowflake, Airbyte, dbt |
| DevOps | AWS, Azure, K8s, Terraform, Datadog |
