# RESOURCE CONTRACT — Contrato Universal de Recursos

---

## ¿Qué es este documento?

RESOURCE_CONTRACT.md define el **contrato universal** que todo recurso debe implementar para ser usado por SRIE. No importa si es un MCP Server, una API REST, una base de datos o un comando CLI — todos implementan exactamente el mismo contrato.

---

## El contrato universal

```yaml
resource_contract:
  version: "1.0.0"

  metodos:
    - nombre: "execute"
      descripcion: "Ejecutar una acción en el recurso"
      entrada:
        action: string            # Acción a ejecutar
        params: {}                # Parámetros de la acción
        timeout_ms: integer       # Timeout máximo
      salida:
        status: string            # SUCCESS | ERROR | TIMEOUT
        data: {}                  # Datos de respuesta
        error: {}                 # Error si ocurrió
        metadata:
          duration_ms: integer
          resource_id: string

    - nombre: "health"
      descripcion: "Verificar que el recurso está funcionando"
      salida:
        status: string            # HEALTHY | DEGRADED | OFFLINE
        latencia_ms: integer
        ultimo_ok: string

    - nombre: "capabilities"
      descripcion: "Listar las capacidades que ofrece este recurso"
      salida:
        capabilities: [string]    # Lista de capacidades

    - nombre: "schema"
      descripcion: "Obtener el esquema de entrada/salida del recurso"
      salida:
        actions: {}               # Schema por acción
```

---

## Adaptación de cada tipo

### MCP Server

```yaml
adapter: "mcp"
contrato:
  execute: "MCP tool call → std JSON-RPC"
  health: "MCP resources/health → std"
  capabilities: "MCP tools/list → std"
  schema: "MCP tools/{name} → inputSchema"
```

### REST API

```yaml
adapter: "rest"
contrato:
  execute: "HTTP request (method, path, body) → response"
  health: "GET /health → 200"
  capabilities: "GET /capabilities → lista"
  schema: "OpenAPI spec → parse"
```

### CLI Tool

```yaml
adapter: "cli"
contrato:
  execute: "comando args → stdout/stderr"
  health: "comando --version → exit code 0"
  capabilities: "comando --help → parse flags"
  schema: "comando --help → parse"
```

### SSH

```yaml
adapter: "ssh"
contrato:
  execute: "ssh comando → stdout/stderr"
  health: "ssh uptime → exit code 0"
  capabilities: "ssh which → binarios disponibles"
  schema: "manual del comando"
```

### Database (SQL)

```yaml
adapter: "sql"
contrato:
  execute: "SQL query → rows"
  health: "SELECT 1 → 1 row"
  capabilities: "SHOW TABLES, \dt → schema"
  schema: "DESCRIBE table, \d table → columns"
```

---

## Schema de acciones

Cada recurso expone un schema de las acciones que soporta:

```yaml
# GitHub MCP → schema
actions:
  create_repository:
    input:
      name: { type: string, required: true }
      description: { type: string, required: false }
      private: { type: boolean, default: false }
    output:
      id: { type: integer }
      name: { type: string }
      url: { type: string }
    timeout_ms: 10000

  list_repositories:
    input:
      org: { type: string, required: false }
    output:
      repositories: { type: array }
    timeout_ms: 5000
```

---

## Validación de contrato

```yaml
validation:
  prerequisitos:
    - "El recurso responde a execute()"
    - "El recurso responde a health()"
    - "El recurso responde a capabilities()"
    - "El recurso responde a schema()"

  periodic_check:
    intervalo: "cada 5 minutos"
    accion: "Ejecutar health()"
    si_falla: "Marcar como DEGRADED, luego OFFLINE"

  onboarding:
    proceso: |
      1. Descubrir recurso
      2. Ejecutar health() → debe responder
      3. Ejecutar capabilities() → debe devolver lista
      4. Ejecutar schema() → debe devolver schema
      5. Registrar en Resource Registry como VERIFIED
```
