# RESOURCE BROKER — Broker de Recursos

---

## ¿Qué es este documento?

RESOURCE_BROKER.md define el **Resource Broker** de SRIE — el componente que resuelve "necesito hacer X" en "usar el recurso Y". Es la capa de **inversión de dependencias** del sistema: las Capabilities nunca piden herramientas concretas. Piden capacidades abstractas. El Broker decide qué recurso usar.

---

## Principio fundamental

Una Capability nunca debería decir:

> "Necesito GitHub."

Debería decir:

> **"Necesito gestionar un repositorio Git."**

Y el Resource Broker decide:

- GitHub MCP
- GitLab MCP
- Azure DevOps
- CLI de Git local

---

## Arquitectura del Broker

```
CAPABILITY
    │
    ▼
"Necesito: gestionar_repositorio_git"
    │
    ▼
RESOURCE BROKER
    │
    ├── 1. Consultar Resource Registry
    ├── 2. Filtrar por Domain (permisos)
    ├── 3. Evaluar disponibilidad (ACTIVE? DEGRADED?)
    ├── 4. Evaluar confianza
    ├── 5. Evaluar latencia
    └── 6. Seleccionar el mejor recurso
    │
    ▼
"Usar: GitHub MCP (confianza: 0.95, latencia: 230ms)"
    │
    ▼
EXECUTION LAYER (adaptador concreto)
```

---

## Contrato del Broker

```yaml
resource_broker:
  metodo_principal: "resolve(capacidad, contexto) → Recurso"

  resolve:
    entrada:
      capacidad:
        id: string              # "gestionar_repositorio_git"
        parametros: {}          # Opcional: parámetros específicos
      contexto:
        domain_id: string       # ¿Quién solicita?
        intencion_id: string    # ¿Para qué?
        prioridad: string       # baja | media | alta | critica

    proceso:
      1. "Consultar Resource Registry por capacidad"
      2. "Filtrar por domains_permitidos del domain solicitante"
      3. "Ordenar por: confianza × (1 - latencia_normalizada) × disponibilidad"
      4. "Seleccionar el de mayor score"
      5. "Si ningún recurso tiene score > 0.50: devolver NO_RESOURCE_AVAILABLE"

    salida:
      recurso:
        id: "RES-GH-001"
        nombre: "GitHub MCP"
        confianza: 0.95
        score: 0.88
        adapter: "mcp"
        config: { token: "$GITHUB_TOKEN" }

  metodos_adicionales:
    - nombre: "list_available(capacidad)"
      descripcion: "Listar todos los recursos disponibles para una capacidad"

    - nombre: "get_health(recurso_id)"
      descripcion: "Obtener estado de salud de un recurso"

    - nombre: "select_best(capacidad, criterios)"
      descripcion: "Seleccionar el mejor recurso según criterios específicos"
```

---

## Score de selección

```yaml
selection_score:
  formula: |
    Score = (confianza × 0.35)
          + (disponibilidad × 0.25)
          + (1.0 - latencia_normalizada) × 0.20
          + (costo_inverso_normalizado × 0.10)
          + (prioridad_domain × 0.10)

  donde:
    confianza: "Confianza histórica del recurso (0.0 - 1.0)"
    disponibilidad: "Tasa de éxito reciente (0.0 - 1.0)"
    latencia_normalizada: "Latencia relativa al máximo aceptable"
    costo_inverso_normalizado: "1.0 - (costo / costo_máximo)"
    prioridad_domain: "Prioridad del domain solicitante (0.0 - 1.0)"
```

---

## Resolución por capacidad

```yaml
resoluciones_ejemplo:
  - capacidad: "gestionar_repositorio_git"
    recursos:
      - id: "RES-GH-001"       # GitHub MCP
        score: 0.88
      - id: "RES-GL-001"       # GitLab MCP
        score: 0.72
      - id: "RES-GIT-001"      # Git CLI local
        score: 0.65

  - capacidad: "enviar_mensaje"
    recursos:
      - id: "RES-SLACK-001"    # Slack MCP
        score: 0.91
      - id: "RES-EMAIL-001"    # Email SMTP
        score: 0.45

  - capacidad: "ejecutar_modelo_ia"
    recursos:
      - id: "RES-OPENAI-001"   # OpenAI API
        score: 0.88
      - id: "RES-ANTHROPIC-001" # Anthropic API
        score: 0.82
      - id: "RES-OLLAMA-001"   # Local LLM
        score: 0.55

  - capacidad: "almacenar_archivos"
    recursos:
      - id: "RES-S3-001"       # AWS S3
        score: 0.85
      - id: "RES-FS-001"       # Local Filesystem
        score: 0.75
```

---

## Caché de resolución

```yaml
resolution_cache:
  tipo: "LRU (Least Recently Used)"
  max_entries: 100
  ttl_segundos: 300            # 5 minutos

  reglas:
    - "Si la misma capacidad se solicita con el mismo contexto, usar caché"
    - "Si un recurso cambia de estado (ACTIVE → DEGRADED), invalidar caché"
    - "Si hay un nuevo recurso disponible, no invalidar hasta que expire el TTL"
```

---

## Fallback

```yaml
fallback:
  mecanismo: "Si el recurso seleccionado falla, probar el siguiente en la lista"
  max_intentos: 3
  backoff_ms: [1000, 5000, 15000]

  si_todos_fallan:
    accion: "Emitir evento RESOURCE_UNAVAILABLE"
    respuesta: "No hay recursos disponibles para la capacidad solicitada"
```

---

## Integración con Federation

```yaml
federation:
  tipo: "Un recurso puede vivir en otro SRIE Domain"
  ejemplo: |
    Marketing SRIE tiene Canva MCP.
    Producto SRIE necesita editar un diseño.
    Producto solicita "editar_disenio" al Resource Broker.
    Broker resuelve: Canva MCP en DOM-MARKETING.
    Broker solicita acceso mediante contrato federado.
    Marketing concede acceso.
    Producto usa Canva MCP sin tenerlo instalado.

  reglas:
    - "Recursos federados se resuelven igual que locales"
    - "El contrato federado determina si el acceso está permitido"
    - "La latencia de recursos federados se penaliza en el score"
```
