# DISCOVERY PIPELINE — Proceso Incremental, Continuo y Event-Driven

---

## ¿Qué es este documento?

DISCOVERY_PIPELINE.md define **cómo se ejecuta Discovery** en el tiempo. No es un comando único. Es un **pipeline continuo** que se activa por eventos, procesa cambios incrementalmente y mantiene el modelo del proyecto siempre actualizado.

---

## La filosofía del pipeline

Discovery deja de ser un comando que se ejecuta una vez. Se convierte en un **servicio** que:

1. Se ejecuta completo la primera vez
2. A partir de ahí, solo procesa diferencias
3. Se activa automáticamente por eventos del proyecto
4. Mantiene el Knowledge Graph, Registry y artefactos siempre sincronizados
5. Permite que otros Engines consulten el estado actual sin esperar

---

## Arquitectura del pipeline

```
┌──────────────────────────────────────────────────────────────────┐
│                        DISCOVERY PIPELINE                        │
│                                                                   │
│  ┌──────────────┐    ┌──────────────┐    ┌───────────────────┐   │
│  │  EVENT BUS   │───►│  DISPATCHER  │───►│  PROJECT HASHER   │   │
│  └──────────────┘    └──────────────┘    └────────┬──────────┘   │
│                                                    │              │
│                                                    ▼              │
│                                          ┌───────────────────┐   │
│                                          │  CHANGE DETECTOR  │   │
│                                          └────────┬──────────┘   │
│                                                    │              │
│                          ┌─────────────────────────┼─────────┐    │
│                          │                         │         │    │
│                    ┌─────▼─────┐           ┌───────▼──────┐  │    │
│                    │ NO CHANGE │           │  CHANGES     │  │    │
│                    │ (exit 10) │           │  DETECTED    │  │    │
│                    └───────────┘           └───────┬──────┘  │    │
│                                                     │         │    │
│                                                     ▼         │    │
│                                          ┌───────────────────┐│    │
│                                          │  PROTOCOL ENGINE  ││    │
│                                          │  (Observe → ...)  ││    │
│                                          └────────┬──────────┘│    │
│                                                    │         │    │
│                                                    ▼         │    │
│                                          ┌───────────────────┐│    │
│                                          │  PUBLISHER        ││    │
│                                          │  (artefactos)     ││    │
│                                          └────────┬──────────┘│    │
│                                                    │         │    │
│                                                    ▼         │    │
│                                          ┌───────────────────┐│    │
│                                          │  EVENT EMITTER    ││    │
│                                          └────────┬──────────┘│    │
│                                                    │         │    │
│                                                    ▼         │    │
│                                          ┌───────────────────┐│    │
│                                          │  CACHE UPDATER    ││    │
│                                          └───────────────────┘│    │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

---

## Componentes del pipeline

### 1. Event Bus

Canal de entrada para eventos que activan Discovery.

```yaml
event_bus:
  eventos_que_activan_discovery:
    - "PROJECT_BOOTSTRAPPED"     # Después de srie bootstrap
    - "FILE_CHANGED"              # Watcher del sistema de archivos (modo watch)
    - "GIT_COMMIT"                # Nuevo commit en el repositorio
    - "DEPLOY_COMPLETED"          # Después de un deploy
    - "MANUAL_REQUEST"            # srie discover
    - "SCHEDULED"                 # Discovery programado (cron)

  formato_evento:
    tipo: string
    timestamp: string
    datos: {}
```

### 2. Dispatcher

Determina cómo ejecutar Discovery según el tipo de evento.

```yaml
dispatcher:
  reglas:
    - evento: "PROJECT_BOOTSTRAPPED"
      modo: "full"
      prioridad: "alta"

    - evento: "FILE_CHANGED"
      modo: "incremental"
      prioridad: "media"
      debounce_ms: 2000

    - evento: "GIT_COMMIT"
      modo: "incremental"
      prioridad: "alta"

    - evento: "DEPLOY_COMPLETED"
      modo: "incremental"
      prioridad: "media"

    - evento: "MANUAL_REQUEST"
      modo: "segun parametro"
      prioridad: "alta"

    - evento: "SCHEDULED"
      modo: "incremental"
      prioridad: "baja"
```

### 3. Project Hasher

Calcula un hash del estado actual del proyecto para detectar cambios.

```yaml
project_hasher:
  entrada: "Ruta del proyecto"
  proceso:
    - "Calcular hash de cada archivo (SHA256 del contenido)"
    - "Calcular hash del árbol de directorios"
    - "Calcular hash de git (último commit + branch)"
    - "Combinar en un hash único del proyecto"
  salida:
    project_hash: string
    file_hashes: {path: hash}
    git_state: {commit: string, branch: string}
```

### 4. Change Detector

Compara el estado actual con el último caché y determina qué cambió.

```yaml
change_detector:
  entrada:
    current_hash: string
    file_hashes: {path: hash}
    git_state: {}
  proceso:
    - "Comparar project_hash con DISCOVERY_CACHE.project_hash"
    - "Si iguales → NO_ACTION_NEEDED"
    - "Si diferentes → identificar archivos con hash distinto"
    - "Clasificar cambios:"
      - "ARCHIVOS_CREADOS"
      - "ARCHIVOS_MODIFICADOS"
      - "ARCHIVOS_ELIMINADOS"
      - "GIT_CHANGED"
  salida:
    hay_cambios: boolean
    archivos_creados: [path]
    archivos_modificados: [path]
    archivos_eliminados: [path]
    git_cambio: boolean
```

### 5. Protocol Engine

Ejecuta el protocolo de Discovery (Observe → Classify → Correlate → Validate → Publish) solo sobre los cambios detectados.

En modo incremental:

```yaml
protocol_engine_incremental:
  entrada: "Lista de cambios"
  proceso:
    OBSERVE:
      - "Solo escanear archivos en la lista de cambios"
      - "Para archivos eliminados, marcar como eliminados"
    CLASSIFY:
      - "Clasificar solo nuevas entidades"
      - "Re-clasificar entidades modificadas"
      - "Eliminar entidades cuyos archivos fueron eliminados"
    CORRELATE:
      - "Re-calcular relaciones que involucran entidades afectadas"
      - "Detectar nuevas relaciones"
      - "Marcar relaciones huérfanas"
    VALIDATE:
      - "Validar solo afirmaciones afectadas por los cambios"
      - "Mantener confianza de afirmaciones no afectadas"
    PUBLISH:
      - "Actualizar solo artefactos afectados"
      - "No regenerar artefactos completos si no es necesario"
```

### 6. Publisher

Genera o actualiza los artefactos.

```yaml
publisher:
  modo_full:
    - "Generar todos los artefactos desde cero"
    - "Emitir evento DISCOVERY_COMPLETED"

  modo_incremental:
    - "Actualizar PROJECT_STATE.md solo secciones afectadas"
    - "Actualizar GRAPH.md solo nodos y relaciones afectados"
    - "Agregar entrada a DISCOVERY_INDEX.md"
    - "Emitir evento DISCOVERY_UPDATED"
```

### 7. Event Emitter

Emite eventos para que otros Engines reaccionen al descubrimiento.

```yaml
event_emitter:
  eventos:
    - tipo: "DISCOVERY_STARTED"
      datos: {modo: string, discovery_id: string}

    - tipo: "DISCOVERY_COMPLETED"
      datos: {modo: string, discovery_id: string, entidades: integer}

    - tipo: "DISCOVERY_UPDATED"
      datos: {modo: string, discovery_id: string, cambios: [string]}

    - tipo: "DISCOVERY_FAILED"
      datos: {modo: string, error: string}
```

### 8. Cache Updater

Actualiza el archivo de caché con el nuevo estado.

```yaml
cache_updater:
  proceso:
    - "Actualizar project_hash"
    - "Actualizar file_hashes"
    - "Actualizar git_state"
    - "Actualizar métricas"
    - "Escribir DISCOVERY_CACHE.yaml"
```

---

## Flujo completo: primera ejecución

```
1. Evento: PROJECT_BOOTSTRAPPED
2. Dispatcher: modo = full
3. Project Hasher: calcular hash de todos los archivos
4. Change Detector: no hay caché previa → cambios = todos los archivos
5. Protocol Engine (full):
   5a. OBSERVE: escanear todos los archivos
   5b. CLASSIFY: clasificar todas las entidades
   5c. CORRELATE: construir grafo completo
   5d. VALIDATE: validar todas las hipótesis
   5e. PUBLISH: generar todos los artefactos
6. Event Emitter: DISCOVERY_COMPLETED
7. Cache Updater: escribir caché
```

---

## Flujo: ejecución incremental (sin cambios)

```
1. Evento: FILE_CHANGED (falso positivo, watcher)
2. Dispatcher: modo = incremental, debounce 2s
3. Project Hasher: calcular hash
4. Change Detector: comparar con caché
   → project_hash IDÉNTICO
   → NO_ACTION_NEEDED (exit 10)
5. No se ejecuta nada más
```

---

## Flujo: ejecución incremental (con cambios)

```
1. Evento: GIT_COMMIT
2. Dispatcher: modo = incremental, prioridad alta
3. Project Hasher: calcular hash
4. Change Detector:
   → project_hash DIFERENTE
   → archivos_modificados: [app/calendar.py, tests/test_calendar.py]
   → archivos_creados: [capabilities/calendar/calendar.manifest]
5. Protocol Engine (incremental):
   5a. OBSERVE: solo app/calendar.py, tests/test_calendar.py, capabilities/calendar/
   5b. CLASSIFY: clasificar calendar.manifest como nueva Capability
   5c. CORRELATE: conectar Calendar con Auth (dependencia), con CAPABILITIES
   5d. VALIDATE: validar solo las nuevas afirmaciones de Calendar
   5e. PUBLISH: actualizar PROJECT_STATE.md (capabilities), GRAPH.md (nuevos nodos)
6. Event Emitter: DISCOVERY_UPDATED {modo: incremental, cambios: [calendar]}
7. Cache Updater: actualizar hash, file_hashes
```

---

## Flujo: modo WATCH (servicio continuo)

```
1. Iniciar watcher del sistema de archivos
2. Por cada evento de archivo:
   2a. Acumular cambios (debounce 2s)
   2b. Ejecutar pipeline incremental
   2c. Si hay cambios → emitir DISCOVERY_UPDATED
   2d. Si no hay cambios → no hacer nada
3. Por cada commit de git:
   3a. Ejecutar pipeline incremental inmediato
4. Discovery programado (cada N horas):
   4a. Ejecutar pipeline completo (full) para recalibrar
```

---

## Estrategia de hashing

```yaml
hashing:
  algoritmo: "SHA-256"
  alcance:
    - "Contenido de cada archivo de código fuente"
    - "Archivos de configuración"
    - "Estructura de directorios"
    - "Estado de git (HEAD commit + branch)"
  exclusiones:
    - "SDOS/ (cambios en SDOS no activan rediscovery)"
    - "node_modules/, __pycache__/, .git/"
    - "Archivos binarios"
    - "Archivos de más de 10MB"
  granularidad:
    full: "Hash de todos los archivos"
    incremental: "Hash de archivos modificados + hash estructural"
```

---

## Concurrencia y bloqueos

```yaml
concurrencia:
  reglas:
    - "Solo una ejecución de Discovery a la vez"
    - "Si Discovery ya está corriendo, los eventos se encolan"
    - "Prioridad: manual > git > file_change > scheduled"
    - "Si llega un evento de mayor prioridad, se cancela la ejecución actual"
  bloqueo:
    archivo: "SDOS/discovery/DISCOVERY_LOCK"
    timeout_ms: 60000
    accion_si_timeout: "Forzar desbloqueo + emitir DISCOVERY_FAILED"
```

---

## Integración con el ciclo SDOS

```yaml
integracion_sdos:
  despues_de_bootstrap:
    - "Ejecutar Discovery completo"
    - "Al completar → SDOS avanza a INDICATORS"

  durante_ciclo:
    - "Discovery puede ejecutarse incrementalmente en cualquier fase"
    - "Si Discovery detecta cambios que afectan indicadores → notificar a Indicators Engine"
    - "Si Discovery detecta cambios que afectan el plan → notificar a Planner Engine"

  al_finalizar_ciclo:
    - "Ejecutar Discovery completo para recalibrar"
    - "Comparar estado real vs estado esperado del plan"

  bajo_demanda:
    - "srie discover → ejecuta pipeline completo o incremental según flag"
    - "srie discover --watch → inicia modo servicio continuo"
```

---

## Códigos de salida del pipeline

| Código | Estado | Significado |
|--------|--------|-------------|
| 0 | `DISCOVERY_COMPLETED` | Pipeline ejecutado exitosamente |
| 10 | `NO_ACTION_NEEDED` | Incremental: no hay cambios |
| 20 | `DISCOVERY_PARTIAL` | Completado con advertencias (confianza baja) |
| 30 | `DISCOVERY_LOCKED` | Otro Discovery está en ejecución |
| 40 | `DISCOVERY_TIMEOUT` | Excedió el tiempo máximo |
| 99 | `DISCOVERY_FAILED` | Error no recuperable |
