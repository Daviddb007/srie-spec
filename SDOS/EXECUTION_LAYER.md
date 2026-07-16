# EXECUTION LAYER — Capa de Ejecución del Sistema

---

## ¿Qué es este documento?

EXECUTION_LAYER.md define la **capa de ejecución** de SRIE — la interfaz entre las Capacidades (qué quiere hacer el sistema) y los Recursos (con qué lo hace). Es el middleware que traduce "ejecutar acción X" en "llamar al adaptador Y con parámetros Z".

---

## La arquitectura completa

```
CAPABILITY ENGINE
    │
    ▼
Action Plan: [{accion, params, recursos}]
    │
    ▼
┌──────────────────────────────────────────────────────────┐
│                   EXECUTION LAYER                        │
│                                                          │
│   ┌──────────────┐                                       │
│   │  RESOURCE    │  Resuelve capacidad → recurso           │
│   │  BROKER      │                                       │
│   └──────┬───────┘                                       │
│          │                                               │
│          ▼                                               │
│   ┌──────────────┐                                       │
│   │  ADAPTER     │  Traduce acción → llamada concreta     │
│   │  SELECTOR    │                                       │
│   └──────┬───────┘                                       │
│          │                                               │
│          ▼                                               │
│   ┌──────────────┐                                       │
│   │  ADAPTER     │  MCP / REST / CLI / SSH / SQL          │
│   │  EXECUTOR    │                                       │
│   └──────┬───────┘                                       │
│          │                                               │
│          ▼                                               │
│   ┌──────────────┐                                       │
│   │  RESULT      │  Estandarizar respuesta, registrar      │
│   │  NORMALIZER  │                                       │
│   └──────┬───────┘                                       │
│          │                                               │
│          ▼                                               │
│   ┌──────────────┐                                       │
│   │  EVIDENCE    │  Registrar ejecución en evidencia       │
│   │  RECORDER    │                                       │
│   └──────────────┘                                       │
│                                                          │
└──────────────────────────────────────────────────────────┘
    │
    ▼
Resultado estandarizado
```

---

## Componentes

### 1. Resource Broker

Resuelve "capacidad X" → "mejor recurso disponible" (ver RESOURCE_BROKER.md).

### 2. Adapter Selector

Selecciona el adaptador correcto según el tipo de recurso.

```yaml
adapter_selector:
  reglas:
    - tipo: "mcp"
      adapter: "MCPAdapter"
    - tipo: "rest"
      adapter: "RESTAdapter"
    - tipo: "cli"
      adapter: "CLIAdapter"
    - tipo: "ssh"
      adapter: "SSHAdapter"
    - tipo: "sql"
      adapter: "SQLAdapter"
    - tipo: "sdk"
      adapter: "SDKAdapter"
```

### 3. Adapter Executor

Ejecuta la acción en el recurso usando el adaptador concreto.

```yaml
adapter_executor:
  entrada:
    adapter_type: string
    action: string
    params: {}
    resource_config: {}
  proceso:
    - "Construir llamada según el protocolo del adaptador"
    - "Ejecutar con timeout del recurso"
    - "Manejar errores según el tipo de adaptador"
    - "Reintentar si el error es recuperable"
  salida:
    status: string
    data: {}
    error: {}
    metadata: { duration_ms, adapter_type }
```

### 4. Result Normalizer

Estandariza la respuesta del recurso a un formato único.

```yaml
result_normalizer:
  entrada: "Respuesta cruda del adaptador"
  proceso:
    - "Extraer status (SUCCESS / ERROR / TIMEOUT)"
    - "Extraer data (payload de la respuesta)"
    - "Extraer error (si existe, normalizar mensaje)"
    - "Calcular metadata (duración, confianza)"
  salida: "Respuesta estandarizada"
```

### 5. Evidence Recorder

Registra la ejecución en la evidencia del sistema.

```yaml
evidence_recorder:
  entrada: "Ejecución completa"
  registro:
    path: "SDOS/evidence/execution/<fecha>/<ejecucion_id>.json"
    contenido:
      - capacidad_solicitada
      - recurso_usado
      - adapter_type
      - duracion_ms
      - status
      - confianza
  eventos:
    - "EXECUTION_SUCCESS"
    - "EXECUTION_FAILED"
    - "EXECUTION_TIMEOUT"
```

---

## Flujo de ejecución completo

```
1. Capability define: "necesito gestionar_repositorio_git, crear repo X"
2. Capability llama a Execution Layer con: {capacidad, accion, params}
3. Execution Layer:
   a. Resource Broker: "mejor recurso para gestionar_repositorio_git"
      → GitHub MCP (score: 0.88)
   b. Adapter Selector: "GitHub MCP es tipo 'mcp'"
      → seleccionar MCPAdapter
   c. Adapter Executor: "MCPAdapter.create_repository(name: X)"
      → llamar a GitHub MCP
   d. Result Normalizer: estandarizar respuesta
   e. Evidence Recorder: registrar en evidencia
4. Execution Layer devuelve: {status: SUCCESS, data: {repo_url: ...}}
```

---

## Action Plan

Las Capacidades no ejecutan directamente. Producen un **Action Plan** que la Execution Layer interpreta:

```yaml
action_plan:
  id: "AP-001"
  capacidad: "gestionar_repositorio_git"
  acciones:
    - orden: 1
      accion: "create_repository"
      params:
        name: "mi-proyecto"
        private: true
      dependencia: null

    - orden: 2
      accion: "add_collaborator"
      params:
        repo: "mi-proyecto"
        collaborator: "usuario"
      dependencia: [1]           # Esperar a que la acción 1 termine

  contexto:
    domain_id: "DOM-BACKEND"
    intencion_id: "INT-001"
```

---

## Timeouts y reintentos

```yaml
execution_policy:
  timeout_default_ms: 30000
  max_retries: 3
  backoff: "exponential(1000, 5000, 15000)"

  retryable_errors:
    - "TIMEOUT"
    - "RATE_LIMITED"
    - "SERVICE_UNAVAILABLE"
    - "NETWORK_ERROR"

  non_retryable_errors:
    - "INVALID_INPUT"
    - "UNAUTHORIZED"
    - "NOT_FOUND"
    - "FORBIDDEN"
```

---

## Seguridad de ejecución

```yaml
execution_security:
  aislamiento:
    - "Cada ejecución se aísla del resto"
    - "No se comparten contextos entre ejecuciones de diferentes Domains"

  autorizacion:
    - "Antes de ejecutar, verificar que el Domain tiene permiso para usar el recurso"
    - "Si no: rechazar con UNAUTHORIZED"

  registro:
    - "Toda ejecución queda registrada en evidencia"
    - "No hay ejecuciones no registradas"

  secretos:
    - "Los tokens y keys se leen desde el Resource Registry, nunca desde código"
    - "Los secretos no se incluyen en la evidencia"
```
