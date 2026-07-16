# RESOURCE REGISTRY — Registro Central de Recursos

---

## ¿Qué es este documento?

RESOURCE_REGISTRY.md define el **registro central de recursos** de SRIE. Es el catálogo de todo con lo que el sistema puede ejecutar capacidades. Cada recurso tiene un ID único, un estado, un contrato y un perfil de capacidades.

---

## Resource Registry

El Resource Registry se materializa en `SDOS/resource/RESOURCE_REGISTRY.yaml`:

```yaml
# SDOS/resource/RESOURCE_REGISTRY.yaml
resource_registry:
  version: "1.0.0"
  ultima_actualizacion: "2026-07-15T10:00:00Z"

  recursos:
    RES-GH-001:
      nombre: "GitHub MCP"
      tipo: "external"
      subtipo: "mcp"
      estado: "ACTIVE"
      capacidad: "gestionar_repositorio_git"
      provider: "github.com"
      confianza: 0.95

    RES-DOCKER-001:
      nombre: "Docker CLI"
      tipo: "tool"
      subtipo: "cli"
      estado: "ACTIVE"
      capacidad: "gestionar_contenedores"
      provider: "local"
      confianza: 0.99

    RES-PG-001:
      nombre: "Postgres Database"
      tipo: "service"
      subtipo: "database"
      estado: "ACTIVE"
      capacidad: "almacenar_datos_relacionales"
      provider: "local"
      confianza: 0.98

    RES-OPENAI-001:
      nombre: "OpenAI API"
      tipo: "service"
      subtipo: "ai_model"
      estado: "ACTIVE"
      capacidad: "ejecutar_modelo_ia"
      provider: "openai.com"
      confianza: 0.93

  metricas:
    total_recursos: 15
    activos: 12
    degradados: 2
    offline: 1
    ultimo_discovery: "2026-07-15T09:00:00Z"
```

---

## Formato de registro

```yaml
resource_entry:
  id: "RES-GH-001"
  nombre: "GitHub MCP"
  tipo: "external"
  subtipo: "mcp"

  capacidad:
    id: "gestionar_repositorio_git"
    descripcion: "Crear, leer, actualizar, eliminar repositorios y gestionar issues/PRs"

  provider: "github.com"
  version: "1.0.0"
  estado: "ACTIVE"

  contrato:
    input_schema: {...}
    output_schema: {...}
    timeout_ms: 30000
    rate_limit: 5000          # peticiones por hora

  autenticacion:
    metodo: "token"
    config_ref: "GITHUB_TOKEN"
    scopes: ["repo", "issues"]

  health:
    ultimo_check: "2026-07-15T10:00:00Z"
    latencia_p50_ms: 230
    latencia_p95_ms: 890
    tasa_exito: 0.97
    estado: "HEALTHY"

  domains:
    permitidos:
      - "DOM-BACKEND"
      - "DOM-DEVOPS"
    prohibidos: []

  metadata:
    descubierto_por: "Tool Discovery"
    descubierto_en: "2026-07-10"
    ultimo_uso: "2026-07-15T10:00:00Z"
    usos_totales: 234
    tags: ["git", "github", "mcp"]
```

---

## Consultas del Resource Registry

```yaml
consultas:
  - nombre: "recursos_por_capacidad"
    query: "¿Qué recursos pueden ejecutar X?"
    ejemplo: "recursos para gestionar_repositorio_git"

  - nombre: "recursos_por_domain"
    query: "¿Qué recursos tiene disponibles el Domain X?"
    ejemplo: "recursos para DOM-BACKEND"

  - nombre: "recursos_activos"
    query: "¿Qué recursos están funcionando ahora?"
    ejemplo: "todos con estado ACTIVE"

  - nombre: "mejor_recurso"
    query: "¿Cuál es el mejor recurso para X?"
    ejemplo: "mejor recurso para ejecutar_modelo_ia (menor latencia, mayor confianza)"
```

---

## Integración con Identity

Al crear un Domain, el Resource Registry se precarga automáticamente según su tipo:

```yaml
auto_provisioning:
  domain_type: "backend"
  recursos:
    - "GitHub MCP"
    - "Docker CLI"
    - "Postgres Database"
    - "SSH Access"
    - "CI/CD Pipeline"

  domain_type: "marketing"
  recursos:
    - "Canva MCP"
    - "Meta Ads API"
    - "Google Analytics API"
    - "LinkedIn API"
    - "HubSpot MCP"
```
