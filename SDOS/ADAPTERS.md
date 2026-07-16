# ADAPTERS — Patrones de Adaptadores para Recursos

---

## ¿Qué es este documento?

ADAPTERS.md define los **patrones de adaptadores** que la Execution Layer usa para comunicarse con diferentes tipos de recursos. Cada adaptador implementa el Resource Contract de forma específica para su protocolo.

Los adaptadores son el **límite del sistema**: traducen entre el mundo SRIE (capacidades, intenciones, evidencia) y el mundo externo (APIs, CLIs, servicios, bases de datos).

---

## El catálogo de adaptadores

```yaml
adapters:
  - tipo: "mcp"
    nombre: "MCPAdapter"
    protocolo: "Model Context Protocol (JSON-RPC)"
    implementa: "RESOURCE_CONTRACT.md v1.0"

  - tipo: "rest"
    nombre: "RESTAdapter"
    protocolo: "HTTP/1.1 + JSON"
    implementa: "RESOURCE_CONTRACT.md v1.0"

  - tipo: "cli"
    nombre: "CLIAdapter"
    protocolo: "Subprocess + stdout/stderr"
    implementa: "RESOURCE_CONTRACT.md v1.0"

  - tipo: "ssh"
    nombre: "SSHAdapter"
    protocolo: "SSH + remote command"
    implementa: "RESOURCE_CONTRACT.md v1.0"

  - tipo: "sql"
    nombre: "SQLAdapter"
    protocolo: "SQL sobre driver nativo"
    implementa: "RESOURCE_CONTRACT.md v1.0"

  - tipo: "sdk"
    nombre: "SDKAdapter"
    protocolo: "Librería nativa del lenguaje"
    implementa: "RESOURCE_CONTRACT.md v1.0"
```

---

## 1. MCPAdapter

Adaptador para servidores MCP (Model Context Protocol).

```yaml
adapter:
  tipo: "mcp"
  protocolo: "JSON-RPC 2.0 sobre stdio, WebSocket o HTTP"

  execute:
    entrada:
      server_script: string      # Ruta al script del servidor MCP
      tool_name: string          # Nombre del tool MCP
      arguments: {}              # Argumentos del tool
    proceso:
      - "Iniciar servidor MCP como subprocess"
      - "Enviar JSON-RPC: {jsonrpc: '2.0', method: 'tools/call', params: {name, arguments}}"
      - "Recibir respuesta JSON-RPC"
      - "Cerrar servidor"
    salida: "content del tool response"

  health:
    proceso: "Enviar JSON-RPC: {method: 'resources/health', params: {}}"
    espera: "Respuesta con status HEALTHY"

  capabilities:
    proceso: "Enviar JSON-RPC: {method: 'tools/list', params: {}}"
    salida: "Lista de tools con sus inputSchema"

  schema:
    proceso: "Del tools/list, extraer inputSchema de cada tool"
    salida: "Schema por tool"

  config_ejemplo:
    server_script: "node /path/to/mcp-server/index.js"
    env:
      GITHUB_TOKEN: "$GITHUB_TOKEN"
```

---

## 2. RESTAdapter

Adaptador para APIs REST.

```yaml
adapter:
  tipo: "rest"
  protocolo: "HTTP/1.1 + JSON"

  execute:
    entrada:
      base_url: string
      method: string              # GET | POST | PUT | DELETE
      path: string
      headers: {}
      body: {}
    proceso:
      - "Construir request HTTP"
      - "Enviar con timeout configurable"
      - "Parsear respuesta JSON"
      - "Mapear códigos HTTP a status del contrato"
    salida: "body de la respuesta"

  health:
    proceso: "GET {base_url}/health"
    espera: "200 OK"

  capabilities:
    proceso: "GET {base_url}/capabilities o parsear OpenAPI spec"
    salida: "Lista de endpoints disponibles"

  mapeo_http:
    200: "SUCCESS"
    201: "SUCCESS"
    400: "ERROR (INVALID_INPUT)"
    401: "ERROR (UNAUTHORIZED)"
    403: "ERROR (FORBIDDEN)"
    404: "ERROR (NOT_FOUND)"
    429: "ERROR (RATE_LIMITED)"
    500: "ERROR (SERVICE_ERROR)"
```

---

## 3. CLIAdapter

Adaptador para herramientas de línea de comandos.

```yaml
adapter:
  tipo: "cli"
  protocolo: "Subprocess + stdout/stderr + exit code"

  execute:
    entrada:
      command: string             # Comando a ejecutar
      args: [string]              # Argumentos
      env: {}                     # Variables de entorno adicionales
      working_dir: string         # Directorio de trabajo
    proceso:
      - "Ejecutar comando como subprocess"
      - "Capturar stdout, stderr, exit code"
      - "Timeout configurable"
    salida:
      stdout: string
      stderr: string
      exit_code: integer

  health:
    proceso: "Ejecutar {comando} --version"
    espera: "exit_code = 0"

  capabilities:
    proceso: "Ejecutar {comando} --help o listar binarios disponibles"
    salida: "Flags y subcomandos parseados"

  mapeo_exit_code:
    0: "SUCCESS"
    1: "ERROR"
    otros: "ERROR específico del comando"
```

---

## 4. SSHAdapter

Adaptador para conexiones SSH remotas.

```yaml
adapter:
  tipo: "ssh"
  protocolo: "SSH2 sobre TCP"

  execute:
    entrada:
      host: string
      port: integer               # Default: 22
      username: string
      auth_method: "key | password"
      command: string
    proceso:
      - "Establecer conexión SSH"
      - "Ejecutar comando remoto"
      - "Capturar stdout/stderr"
      - "Cerrar conexión"
    salida:
      stdout: string
      stderr: string
      exit_code: integer

  health:
    proceso: "SSH uptime"
    espera: "Conexión exitosa + exit_code = 0"

  capabilities:
    proceso: "SSH which <comandos> para detectar binarios disponibles"
    salida: "Lista de binarios disponibles en el servidor remoto"
```

---

## 5. SQLAdapter

Adaptador para bases de datos SQL.

```yaml
adapter:
  tipo: "sql"
  protocolo: "SQL sobre driver nativo (psycopg2, mysql-connector, etc.)"

  execute:
    entrada:
      connection_string: string
      query: string
      params: []
      timeout_ms: integer
    proceso:
      - "Conectar a la base de datos"
      - "Ejecutar query con parámetros"
      - "Fetch resultados"
      - "Cerrar conexión"
    salida:
      rows: [{column: value}]
      row_count: integer
      columns: [string]

  health:
    proceso: "SELECT 1"
    espera: "1 fila devuelta"

  capabilities:
    proceso: "SHOW TABLES o SELECT table_name FROM information_schema.tables"
    salida: "Lista de tablas y schemas disponibles"
```

---

## 6. SDKAdapter

Adaptador para librerías nativas del lenguaje.

```yaml
adapter:
  tipo: "sdk"
  protocolo: "Llamada a función/método de librería"

  execute:
    entrada:
      library: string             # Nombre del módulo/librería
      function: string            # Función a llamar
      args: []                    # Argumentos posicionales
      kwargs: {}                  # Argumentos de keyword
    proceso:
      - "Importar librería"
      - "Ejecutar función con argumentos"
      - "Capturar resultado"
      - "Manejar excepciones"
    salida: "Resultado de la función"

  health:
    proceso: "Importar librería exitosamente"
    espera: "No lanza excepción de import"

  capabilities:
    proceso: "Inspeccionar funciones disponibles de la librería"
    salida: "Lista de funciones y firmas"
```

---

## Registro de adaptadores

Los adaptadores se registran en `SDOS/resource/adapters/`:

```yaml
# SDOS/resource/adapters/registry.yaml
adapter_registry:
  version: "1.0.0"
  adapters:
    - tipo: "mcp"
      nombre: "MCPAdapter"
      version: "1.0.0"
      disponible: true

    - tipo: "rest"
      nombre: "RESTAdapter"
      version: "1.0.0"
      disponible: true

    - tipo: "cli"
      nombre: "CLIAdapter"
      version: "1.0.0"
      disponible: true

    - tipo: "ssh"
      nombre: "SSHAdapter"
      version: "1.0.0"
      disponible: false    # Requiere configuración SSH

    - tipo: "sql"
      nombre: "SQLAdapter"
      version: "1.0.0"
      disponible: true

    - tipo: "sdk"
      nombre: "SDKAdapter"
      version: "1.0.0"
      disponible: true
```

---

## Extensibilidad

Para agregar un nuevo tipo de adaptador:

```yaml
nuevo_adapter:
  proceso:
    1. "Implementar los 4 métodos del Resource Contract:"
       - execute(action, params) → Result
       - health() → HealthStatus
       - capabilities() → [string]
       - schema() → Schema
    2. "Registrar en adapter_registry.yaml"
    3. "Agregar regla en Adapter Selector"
    4. "Crear tests para el nuevo adaptador"
    5. "El resto del sistema funciona sin cambios"
```
