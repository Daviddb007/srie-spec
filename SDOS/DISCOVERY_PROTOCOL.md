# DISCOVERY PROTOCOL — Observe → Classify → Correlate → Validate → Publish

---

## ¿Qué es este documento?

DISCOVERY_PROTOCOL.md define el **protocolo científico** que ejecuta el Discovery Engine. No describe componentes internos. Describe **acciones**: las 5 etapas que transforman un proyecto en un modelo estructurado y verificado del sistema.

Cada etapa responde una pregunta:

| Etapa | Pregunta | Entrada | Salida |
|-------|----------|---------|--------|
| **Observe** | ¿Qué existe? | Sistema de archivos | Evidencia cruda |
| **Classify** | ¿Qué significa? | Evidencia cruda | Evidencia clasificada por dominio |
| **Correlate** | ¿Cómo se relaciona? | Evidencia clasificada | Knowledge Graph |
| **Validate** | ¿Es cierto? | Knowledge Graph | Hipótesis verificadas + confianza |
| **Publish** | ¿Qué queda registrado? | Hipótesis verificadas | Artefactos publicados |

---

## Etapa 1: OBSERVE

### Propósito
Capturar evidencia cruda del proyecto sin interpretar. Esta etapa no opina. Solo recolecta hechos observables del sistema de archivos, git, dependencias y configuración.

### Entrada
- Ruta del proyecto
- Lista de archivos a observar (todos en modo full, solo cambios en incremental)
- Hash del proyecto (para incremental)

### Proceso

```
OBSERVE
├── 1.1 Escanear estructura de directorios
├── 1.2 Detectar archivos de configuración
├── 1.3 Detectar dependencias (package.json, pyproject.toml, etc.)
├── 1.4 Detectar framework y lenguaje
├── 1.5 Detectar documentación
├── 1.6 Detectar git (ramas, commits, autores)
├── 1.7 Detectar tests
├── 1.8 Detectar Docker
├── 1.9 Detectar CI/CD
├── 1.10 Detectar variables de entorno y secretos
├── 1.11 Detectar APIs y endpoints
└── 1.12 Detectar arquitectura (patrones de directorios)
```

### Salida: Evidencia cruda

```json
{
  "etapa": "OBSERVE",
  "timestamp": "2026-07-15T10:00:00Z",
  "evidencia": [
    {
      "tipo": "archivo",
      "path": "pyproject.toml",
      "hash": "abc123",
      "tamano": 1024,
      "ultima_modificacion": "2026-07-14T15:00:00Z",
      "contenido_observado": {
        "dependencias": ["fastapi", "sqlalchemy", "pytest"],
        "python_version": ">=3.12"
      }
    },
    {
      "tipo": "archivo",
      "path": "app/main.py",
      "hash": "def456",
      "tamano": 2048,
      "contenido_observado": {
        "imports": ["fastapi", "app.routers", "app.models"],
        "patrones": ["FastAPI app", "router inclusion"]
      }
    },
    {
      "tipo": "git",
      "branch": "main",
      "commits": 47,
      "autores": ["usuario1", "usuario2"],
      "ultimo_commit": {
        "hash": "abc789",
        "mensaje": "feat: add calendar capability",
        "fecha": "2026-07-14"
      }
    }
  ],
  "metricas_observacion": {
    "archivos_escaneados": 128,
    "archivos_configuracion": 5,
    "dependencias_detectadas": 12,
    "tiempo_observacion_ms": 1200
  }
}
```

### Reglas de OBSERVE

1. No interpreta nada. Solo extrae hechos observables.
2. No lee archivos binarios ni ejecutables.
3. No lee archivos de secretos (`.env`, `*.key`, `*.pem`, `credentials.*`).
4. Calcula hash de cada archivo para detección de cambios.
5. El contenido observado se limita a metadatos y estructura (no almacena el contenido completo del archivo).
6. Respeta `.gitignore` para determinar qué archivos escanear.

---

## Etapa 2: CLASSIFY

### Propósito
Clasificar la evidencia cruda dentro de la ontología de SRIE. Cada pieza de evidencia recibe un tipo, un dominio, un nivel de relevancia y una hipótesis preliminar.

### Entrada
- Evidencia cruda (de OBSERVE)
- DOMAINS.md (definición de dominios)
- ONTOLOGY.md (definición de tipos de entidades)

### Proceso

```
CLASSIFY
├── 2.1 Asignar tipo de entidad (cada archivo → tipo ontológico)
├── 2.2 Asignar dominio (ver DOMAINS.md)
├── 2.3 Asignar relevancia (CRITICAL | HIGH | MEDIUM | LOW)
├── 2.4 Generar hipótesis preliminar
└── 2.5 Detectar entidades ya registradas (vs nuevas)
```

### Mapeo archivo → tipo ontológico

```yaml
mapeo:
  - patron: "README.md"
    tipo: Document
    subtipo: especificacion
    dominio: business
    relevancia: CRITICAL

  - patron: "ARCHITECTURE.md"
    tipo: Document
    subtipo: especificacion
    dominio: technical
    relevancia: CRITICAL

  - patron: "capabilities/*/*.manifest"
    tipo: Manifest
    dominio: functional
    relevancia: CRITICAL

  - patron: "Dockerfile"
    tipo: Document
    subtipo: configuracion
    dominio: infrastructure
    relevancia: HIGH

  - patron: "memory/ADR/*"
    tipo: Adr
    dominio: technical
    relevancia: HIGH

  - patron: "SDOS/*"
    tipo: Document
    subtipo: configuracion
    dominio: governance
    relevancia: CRITICAL
```

### Hipótesis preliminar

Cada entidad clasificada genera una hipótesis que será validada en la etapa VALIDATE:

```yaml
hipotesis:
  entidad: "app/main.py"
  tipo_asignado: Document
  dominio_asignado: functional
  afirmaciones:
    - "El proyecto usa FastAPI como framework web"
    - "La aplicación sigue una estructura de routers"
    - "El entry point está en app/main.py"
  evidencia_parcial:
    - "Import directo de fastapi"
    - "Patrón de creación de app FastAPI"
```

### Salida de CLASSIFY

```json
{
  "etapa": "CLASSIFY",
  "timestamp": "2026-07-15T10:00:05Z",
  "entidades_clasificadas": [
    {
      "tipo": "Document",
      "nombre": "README.md",
      "dominio": "business",
      "relevancia": "CRITICAL",
      "hipotesis": ["README.md contiene documentación del proyecto"],
      "ya_registrada": false
    },
    {
      "tipo": "Document",
      "nombre": "app/main.py",
      "dominio": "functional",
      "relevancia": "HIGH",
      "hipotesis": [
        "Framework: FastAPI",
        "Patrón: Routers",
        "Entry point: app/main.py"
      ],
      "ya_registrada": false
    }
  ],
  "entidades_actualizadas": [],
  "entidades_no_encontradas": []
}
```

---

## Etapa 3: CORRELATE

### Propósito
Construir el Knowledge Graph. No descubre archivos individuales. **Descubre relaciones** entre entidades. Aquí nace el grafo del proyecto.

### Entrada
- Entidades clasificadas (de CLASSIFY)
- GRAPH.md (esquema del grafo)
- ONTOLOGY.md (relaciones válidas y prohibidas)

### Proceso

```
CORRELATE
├── 3.1 Identificar relaciones estructurales (directorios → módulos)
├── 3.2 Identificar relaciones de código (imports, dependencias)
├── 3.3 Identificar relaciones de documentación (ADR → Architecture)
├── 3.4 Identificar relaciones de dependencia (capabilities)
├── 3.5 Construir matriz de adyacencia
├── 3.6 Validar contra ontología (relaciones prohibidas)
└── 3.7 Detectar nodos huérfanos y desincronizados
```

### Tipos de relaciones que descubre

```yaml
relaciones:
  - tipo: contiene
    ejemplo: "CAPABILITIES → CAP:calendar"
    evidencia: "Estructura de directorios"

  - tipo: depende_de
    ejemplo: "CAP:calendar → CAP:auth"
    evidencia: "Import o declaración de dependencia"

  - tipo: documenta
    ejemplo: "ADR-001 → ARCHITECTURE"
    evidencia: "ADR menciona decisión arquitectónica"

  - tipo: alimenta
    ejemplo: "MEMORY → INDICATORS"
    evidencia: "Los indicadores usan datos de memoria"

  - tipo: afecta
    ejemplo: "ARCHITECTURE → CAP:calendar"
    evidencia: "Cambio en arquitectura impacta calendar"
```

### Salida de CORRELATE

```json
{
  "etapa": "CORRELATE",
  "timestamp": "2026-07-15T10:00:10Z",
  "nodos": [
    {"id": "business", "tipo": "dominio", "estado": "SYNCED"},
    {"id": "CAP:auth", "tipo": "capability", "estado": "SYNCED"}
  ],
  "relaciones": [
    {
      "origen": "PROJECT",
      "destino": "business",
      "tipo": "contiene",
      "confianza": 0.99,
      "evidencia": "README.md existe en raíz del proyecto"
    },
    {
      "origen": "CAP:calendar",
      "destino": "CAP:auth",
      "tipo": "depende_de",
      "confianza": 0.95,
      "evidencia": "calendar.manifest declara dependencia de auth >=2.0.0"
    }
  ],
  "nodos_huerfanos": [],
  "nodos_desincronizados": []
}
```

---

## Etapa 4: VALIDATE

### Propósito
Validar cada hipótesis contra evidencia concreta. Calcular confidence score para cada afirmación. Identificar afirmaciones especulativas (confianza baja). Esta etapa elimina la alucinación por diseño.

### Entrada
- Hipótesis de CLASSIFY
- Relaciones de CORRELATE
- Evidencia cruda de OBSERVE
- DISCOVERY_EVIDENCE.md (criterios de validación)

### Proceso

```
VALIDATE
├── 4.1 Por cada hipótesis, buscar evidencia que la respalde
├── 4.2 Por cada hipótesis, buscar evidencia que la contradiga
├── 4.3 Calcular confidence score
├── 4.4 Calcular hallucination risk
├── 4.5 Clasificar confianza (ALTA | MEDIA | BAJA | ESPECULATIVA)
├── 4.6 Si confianza < 0.30, marcar como especulativa
└── 4.9 Generar reporte de validación
```

### Cálculo de confianza

Ver DISCOVERY_EVIDENCE.md para la fórmula detallada.

```yaml
confidence:
  formula: |
    Confidence = (source_quality × 0.30)
              + (evidence_diversity × 0.25)
              + (consistency_score × 0.25)
              + (specificity_score × 0.20)

  hallucination_risk: 1.0 - Confidence

  clasificacion:
    ALTA:        "Confidence >= 0.85"
    MEDIA:       "Confidence >= 0.60"
    BAJA:        "Confidence >= 0.30"
    ESPECULATIVA: "Confidence < 0.30"
```

### Salida de VALIDATE

```json
{
  "etapa": "VALIDATE",
  "timestamp": "2026-07-15T10:00:15Z",
  "validaciones": [
    {
      "hipotesis": "El proyecto usa FastAPI como framework web",
      "resultado": "CONFIRMADA",
      "confidence": 0.97,
      "hallucination_risk": 0.03,
      "evidencia": [
        {"fuente": "pyproject.toml", "tipo": "dependencia_explicita", "peso": 0.4},
        {"fuente": "app/main.py", "tipo": "import_directo", "peso": 0.3},
        {"fuente": "app/routers/", "tipo": "patron_arquitectura", "peso": 0.3}
      ]
    },
    {
      "hipotesis": "El proyecto usa arquitectura hexagonal",
      "resultado": "NO_CONFIRMADA",
      "confidence": 0.35,
      "hallucination_risk": 0.65,
      "clasificacion": "BAJA",
      "evidencia": [
        {"fuente": "app/domain/", "tipo": "directorio_domain", "peso": 0.3},
        {"fuente": "app/application/", "tipo": "directorio_application", "peso": 0.3}
      ],
      "evidencia_faltante": [
        "No se encontraron puertos (interfaces)",
        "No se encontraron adaptadores",
        "No se encontraron repositorios como interfaz"
      ],
      "nota": "Existe estructura parcial de domain/application pero faltan puertos y adaptadores. Arquitectura híbrida posible."
    }
  ],
  "metricas_validacion": {
    "hipotesis_confirmadas": 12,
    "hipotesis_especulativas": 1,
    "hipotesis_rechazadas": 0,
    "confidence_promedio": 0.89
  }
}
```

---

## Etapa 5: PUBLISH

### Propósito
Producir los artefactos finales del descubrimiento. No genera opiniones. Genera **evidencia estructurada** que otros Engines pueden consumir.

### Entrada
- Todo el resultado de VALIDATE
- Registry existente
- Estado actual del sistema (STATE.md)

### Proceso

```
PUBLISH
├── 5.1 Generar/actualizar PROJECT_STATE.md
├── 5.2 Generar/actualizar PROJECT_TIMELINE.md
├── 5.3 Generar/actualizar GRAPH.md
├── 5.4 Generar DISCOVERY_REPORT.md
├── 5.5 Generar PROJECT_GRAPH.json
├── 5.6 Generar/actualizar DISCOVERY_INDEX.md
├── 5.7 Generar DISCOVERY_EVIDENCE.json
├── 5.8 Actualizar DISCOVERY_CACHE.yaml (hash)
├── 5.9 Actualizar REGISTRY.yaml
└── 5.10 Emitir evento PROJECT_DISCOVERED
```

### Salida de PUBLISH

Los artefactos generados se detallan en DISCOVERY_OUTPUTS.md. El resultado final es:

```yaml
publish:
  artefactos_generados: 9
  artefactos_actualizados: 3
  registry:
    entidades_nuevas: 5
    entidades_actualizadas: 8
  eventos_emitidos:
    - "PROJECT_DISCOVERED"
    - "DISCOVERY_COMPLETED"
  timestamp: "2026-07-15T10:00:20Z"
```

---

## Garantías del Protocolo

1. **Determinismo** — Mismo proyecto produce el mismo resultado (misma versión de ontología).
2. **No intrusividad** — Discovery nunca modifica el proyecto fuente.
3. **Confianza explícita** — Toda afirmación incluye confidence score.
4. **Incrementalidad** — Solo analiza cambios, nunca repite trabajo completo.
5. **Auditabilidad** — Cada afirmación tiene evidencia trazable.
6. **Publicación estructurada** — Todos los artefactos siguen formatos definidos.
