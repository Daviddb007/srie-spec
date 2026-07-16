# DISCOVERY ENGINE — Especificación del Motor

---

## ¿Qué es el Discovery Engine?

El Discovery Engine es el **sistema sensorial** de SRIE. No es un escáner de archivos. Es un **protocolo científico** que observa el estado real del proyecto, formula hipótesis, busca evidencia, calcula confianza, valida hallazgos y publica resultados estructurados.

Discovery no descubre archivos. **Descubre realidad.** Los archivos son solamente evidencia de esa realidad.

---

## Rol en el ciclo SDOS

Discovery es la primera fase del ciclo operativo después de BOOT:

```
BOOT (INIT)
  │
  ▼
DISCOVERY ←─── Discovery Engine
  │
  ▼
INDICATORS
  │
  ▼
...
```

Discovery se ejecuta:

1. **Completo** — después de `srie bootstrap` (primera vez)
2. **Incremental** — después de cada cambio significativo en el proyecto
3. **Bajo demanda** — con `srie discover`
4. **Como servicio continuo** — observando cambios en el sistema de archivos

---

## Arquitectura del motor

Discovery no es un script monolítico. Es un **protocolo de 5 etapas** que se ejecutan en secuencia:

```
┌──────────────────────────────────────────────────────────┐
│                    DISCOVERY PROTOCOL                     │
│                                                          │
│   OBSERVE                                                │
│     │   Capturar evidencia cruda sin interpretar         │
│     ▼                                                    │
│   CLASSIFY                                               │
│     │   Clasificar evidencia en la ontología             │
│     ▼                                                    │
│   CORRELATE                                              │
│     │   Construir relaciones → Knowledge Graph           │
│     ▼                                                    │
│   VALIDATE                                               │
│     │   Calcular confianza, validar hipótesis            │
│     ▼                                                    │
│   PUBLISH                                                │
│         Producir artefactos (estado, grafo, timeline)    │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

Cada etapa tiene su propia especificación detallada (ver DISCOVERY_PROTOCOL.md).

---

## Contrato del Engine

```yaml
engine:
  nombre: "Discovery Engine"
  version: "1.0.0"
  tipo_engine: discovery

  contrato:
    input:
      archivos_requeridos: []
      parametros:
        - nombre: project_path
          tipo: string
          requerido: true
          descripcion: "Ruta del proyecto a descubrir"
        - nombre: modo
          tipo: string
          requerido: false
          descripcion: "full | incremental | watch"
          defecto: "full"
        - nombre: project_hash
          tipo: string
          requerido: false
          descripcion: "Hash del estado previo (para incremental)"

    output:
      archivos_generados:
        - path: "SDOS/PROJECT_STATE.md"
          condicion: "modo = full o cambios estructurales"
        - path: "SDOS/GRAPH.md"
          condicion: "siempre"
        - path: "SDOS/PROJECT_TIMELINE.md"
          condicion: "siempre"
        - path: "SDOS/discovery/DISCOVERY_REPORT.md"
          condicion: "siempre"
        - path: "SDOS/discovery/PROJECT_GRAPH.json"
          condicion: "siempre"
        - path: "SDOS/discovery/DISCOVERY_INDEX.md"
          condicion: "siempre"
        - path: "SDOS/discovery/DISCOVERY_EVIDENCE.json"
          condicion: "siempre"
        - path: "SDOS/discovery/DISCOVERY_CACHE.yaml"
          condicion: "modo = incremental"
        - path: "SDOS/REGISTRY.yaml"
          condicion: "siempre (actualización)"
      estado_salida:
        exito: "PROJECT_DISCOVERED"
        error: "PROJECT_NOT_FOUND"
        parcial: "DISCOVERY_PARTIAL"

    archivos:
      puede_modificar:
        - "SDOS/PROJECT_STATE.md"
        - "SDOS/GRAPH.md"
        - "SDOS/PROJECT_TIMELINE.md"
        - "SDOS/discovery/"
        - "SDOS/REGISTRY.yaml"
        - "SDOS/evidence/discovery/"
      prohibido_modificar:
        - "*.py, *.js, *.ts, *.go, *.rs"
        - "README.md, ARCHITECTURE.md, DESIGN.md"
        - "package.json, pyproject.toml, Cargo.toml"
        - "CUALQUIER archivo del proyecto fuente"

    reglas:
      - "Discovery NUNCA modifica archivos del proyecto fuente"
      - "Discovery solo escribe dentro de SDOS/ y SDOS/discovery/"
      - "En modo incremental, solo analiza diferencias desde el último hash"
      - "Toda afirmación debe incluir confidence_score"
      - "Si una afirmación tiene confidence < 0.30, debe registrarse como especulativa"

    dependencias:
      engines: []
      archivos:
        - "SDOS/DOMAINS.md"
        - "SDOS/ONTOLOGY.md"
        - "SDOS/GRAPH.md"
        - "SDOS/REGISTRY.md"
      sistema:
        - "acceso_lectura_al_sistema_de_archivos"
        - "git (si el proyecto usa git)"

    indicadores:
      - nombre: "tiempo_ejecucion_ms"
        esperado: "< 5000 (incremental), < 30000 (full)"
      - nombre: "archivos_descubiertos"
      - nombre: "nodos_creados"
      - nombre: "relaciones_creadas"
      - nombre: "confidence_promedio"

    tiempo_esperado_ms: 30000
    nivel_srie_requerido: "L0"
```

---

## Modos de ejecución

### Modo FULL

Ejecuta el protocolo completo: OBSERVE → CLASSIFY → CORRELATE → VALIDATE → PUBLISH. Escanea todo el proyecto desde cero. Genera todos los artefactos.

**Cuándo usarlo:**
- Primera ejecución (después de `srie bootstrap`)
- Cuando se solicita explícitamente (`srie discover --full`)

### Modo INCREMENTAL

Calcula un hash del estado actual del proyecto. Compara con el último hash almacenado en `DISCOVERY_CACHE.yaml`. Solo analiza las diferencias.

**Cuándo usarlo:**
- Ejecuciones posteriores a la primera
- Después de cambios en el proyecto
- Activado por eventos del sistema de archivos

**Flujo incremental:**

```
1. Calcular hash del proyecto (archivos, git, dependencias)
2. Comparar con DISCOVERY_CACHE.yaml
3. Si no hay cambios → NO_ACTION_NEEDED (salida con código 10)
4. Si hay cambios → identificar archivos modificados, creados, eliminados
5. Ejecutar OBSERVE solo sobre los cambios
6. Ejecutar CLASSIFY sobre los cambios
7. Ejecutar CORRELATE (puede requerir re-correlacionar nodos afectados)
8. Ejecutar VALIDATE sobre el subgrafo afectado
9. Ejecutar PUBLISH (solo artefactos afectados)
10. Actualizar DISCOVERY_CACHE.yaml con nuevo hash
```

### Modo WATCH

Modo continuo (servicio). Observa el sistema de archivos y ejecuta Discovery incremental automáticamente cuando detecta cambios.

**Cuándo usarlo:**
- Durante sesiones de desarrollo activo
- Integrado con editores/IDEs

**Flujo watch:**

```
1. Iniciar watcher del sistema de archivos
2. Por cada cambio:
   a. Esperar 2 segundos (debounce)
   b. Ejecutar Discovery incremental
   c. Emitir evento DISCOVERY_UPDATED
   d. Notificar a Indicators Engine (recalcular si es necesario)
```

---

## Artefactos que produce

| Artefacto | Descripción | Ver DISCOVERY_OUTPUTS.md |
|-----------|-------------|--------------------------|
| `SDOS/PROJECT_STATE.md` | Inventario completo del proyecto | Sí |
| `SDOS/PROJECT_TIMELINE.md` | Línea temporal basada en git | Sí |
| `SDOS/GRAPH.md` | Knowledge Graph actualizado | Sí |
| `SDOS/discovery/DISCOVERY_REPORT.md` | Reporte del descubrimiento | Sí |
| `SDOS/discovery/PROJECT_GRAPH.json` | Grafo en formato JSON | Sí |
| `SDOS/discovery/DISCOVERY_INDEX.md` | Índice de todos los descubrimientos | Sí |
| `SDOS/discovery/DISCOVERY_EVIDENCE.json` | Evidencia cruda recolectada | Sí |
| `SDOS/discovery/DISCOVERY_CACHE.yaml` | Caché para ejecución incremental | Sí |
| `SDOS/REGISTRY.yaml` | Registry actualizado con nuevas entidades | Sí |

Ver DISCOVERY_OUTPUTS.md para la especificación detallada de cada artefacto.

---

## Integración con Confidence

Toda afirmación que Discovery produce incluye:

```yaml
afirmacion:
  entidad: "Framework"
  valor: "FastAPI"
  evidencia:
    - fuente: "pyproject.toml"
      contenido: "fastapi = \">=0.100.0\""
      tipo: "dependencia_explicita"
    - fuente: "app/main.py"
      contenido: "from fastapi import FastAPI"
      tipo: "import_directo"
    - fuente: "app/routers/"
      contenido: "Patrón de routers de FastAPI"
      tipo: "patron_arquitectura"
  confidence:
    score: 0.97
    evidence_count: 3
    hallucination_risk: 0.03
    clasificacion: "ALTA"
```

Ver DISCOVERY_EVIDENCE.md para el detalle del cálculo de confianza.

---

## Pipeline de ejecución

Ver DISCOVERY_PIPELINE.md para el detalle del proceso incremental, continuo y event-driven.

---

## Códigos de salida

| Código | Estado | Significado |
|--------|--------|-------------|
| 0 | `PROJECT_DISCOVERED` | Discovery completado exitosamente |
| 1 | `PROJECT_NOT_FOUND` | La ruta del proyecto no existe |
| 2 | `DISCOVERY_PARTIAL` | Discovery completado con advertencias (confianza baja en alguna afirmación) |
| 10 | `NO_ACTION_NEEDED` | Modo incremental: no hay cambios desde el último discovery |
| 99 | `UNKNOWN_ERROR` | Error no categorizado |
