# ENGINE CONTRACT — Contrato Universal de Ejecución para todos los Engines

---

## ¿Qué es este documento?

ENGINE_CONTRACT.md define el **contrato único de ejecución** que todo Engine del sistema SRIE debe implementar. No importa si es Discovery, Planner, Sandbox o Repair — todos siguen exactamente el mismo ciclo de vida, las mismas etapas y el mismo formato de salida.

Este documento es la **interfaz estable** del ecosistema SRIE. Equivale a una interfaz (`interface`) o clase abstracta en programación orientada a objetos. Cualquier Engine presente o futuro debe implementar este contrato.

---

## El ciclo universal de un Engine

```
┌────────────────────────────────────────────────────────────┐
│                  ENGINE UNIVERSAL CYCLE                     │
│                                                             │
│   INPUT                                                     │
│     │                                                       │
│     ▼                                                       │
│   VALIDATION                                                │
│     │                                                       │
│     ▼                                                       │
│   DISCOVERY                                                 │
│     │                                                       │
│     ▼                                                       │
│   PLAN                                                      │
│     │                                                       │
│     ▼                                                       │
│   EXECUTION                                                 │
│     │                                                       │
│     ▼                                                       │
│   VERIFICATION                                              │
│     │                                                       │
│     ▼                                                       │
│   EVIDENCE                                                  │
│     │                                                       │
│     ▼                                                       │
│   MEMORY                                                    │
│     │                                                       │
│     ▼                                                       │
│   OUTPUT                                                    │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

**Da igual cuál sea el Engine. Discovery. Planner. Sandbox. Repair. Todos.**

---

## Etapa 1: INPUT

### Propósito
Recibir todo lo necesario para que el Engine pueda ejecutarse.

### Contrato

```yaml
input:
  fuentes:
    - type: archivo
      descripcion: Archivos que el Engine debe leer para ejecutarse
      requeridos:
        - path: string            # Ruta absoluta o relativa a SDOS/
          descripcion: string
          validacion: string      # Cómo validar que el archivo es correcto
      opcionales:
        - path: string
          descripcion: string
          defecto: any            # Valor por defecto si no existe

    - type: parametro
      descripcion: Parámetros de línea de comando o configuración
      items:
        - nombre: string
          tipo: string
          requerido: boolean
          descripcion: string
          valores_posibles: []     # Opcional: lista de valores válidos

    - type: estado
      descripcion: Estado actual del sistema (desde STATE.md)
      campos:
        - ciclo_actual: integer
        - fase_actual: string
        - nivel_madurez: string

  reglas:
    - Si un archivo requerido no existe, el Engine debe fallar con ERROR_INPUT_MISSING
    - Si un parámetro requerido no se provee, el Engine debe fallar con ERROR_PARAM_MISSING
    - El Engine no debe modificar ningún archivo de entrada
    - El Engine debe leer los archivos de entrada una sola vez al inicio
```

### Códigos de error de INPUT

| Código | Significado |
|--------|-------------|
| `ERROR_INPUT_MISSING` | Falta un archivo requerido |
| `ERROR_PARAM_MISSING` | Falta un parámetro requerido |
| `ERROR_INPUT_INVALID` | Un archivo de entrada tiene formato inválido |
| `ERROR_PARAM_INVALID` | Un parámetro tiene un valor no válido |

---

## Etapa 2: VALIDATION

### Propósito
Verificar que las entradas cumplen el contrato esperado antes de ejecutar cualquier transformación.

### Contrato

```yaml
validation:
  validaciones_pre:
    - tipo: existencia
      descripcion: Verificar que los archivos requeridos existen y son legibles
      accion_si_falla: ERROR_INPUT_MISSING

    - tipo: formato
      descripcion: Verificar que los archivos tienen el formato esperado (YAML, JSON, MD)
      accion_si_falla: ERROR_INPUT_INVALID

    - tipo: contenido
      descripcion: Verificar que los archivos contienen los campos obligatorios
      accion_si_falla: ERROR_INPUT_INVALID

    - tipo: permisos
      descripcion: Verificar que el Engine tiene permiso para leer las entradas
      accion_si_falla: ERROR_PERMISSION_DENIED

    - tipo: estado
      descripcion: Verificar que el estado del sistema permite ejecutar este Engine
      accion_si_falla: ERROR_STATE_CONFLICT

  reglas:
    - Si cualquier validación falla, el Engine no debe continuar a EXECUTION
    - Las validaciones no deben tener efectos secundarios
    - Las validaciones deben ser deterministas (misma entrada = mismo resultado)
```

### Códigos de error de VALIDATION

| Código | Significado |
|--------|-------------|
| `ERROR_PERMISSION_DENIED` | El Engine no tiene permiso para leer las entradas |
| `ERROR_STATE_CONFLICT` | El estado del sistema no permite ejecutar este Engine ahora |
| `ERROR_VALIDATION_FAILED` | Una validación de contenido falló |

---

## Etapa 3: DISCOVERY

### Propósito
Explorar el contexto relevante para la ejecución del Engine. No es el Discovery Engine — es la fase interna donde cada Engine descubre lo que necesita saber sobre su dominio específico.

### Contrato

```yaml
discovery:
  alcance:
    - tipo: archivos
      descripcion: Archivos que el Engine necesita explorar en su dominio
      patrones:
        - pattern: string        # Glob pattern para encontrar archivos
          descripcion: string
          opcional: boolean

    - tipo: estado
      descripcion: Estado del sistema relevante para este Engine
      fuentes:
        - SDOS/STATE.md
        - SDOS/PROJECT_STATE.md

    - tipo: memoria
      descripcion: Memoria existente relevante para este Engine
      fuentes:
        - memory/ADR/
        - memory/context/

  reglas:
    - DISCOVERY solo lee. No modifica nada.
    - DISCOVERY debe registar lo que encontró en un archivo de descubrimiento interno
    - Si DISCOVERY no encuentra lo esperado, debe registrarlo (no fallar)
    - DISCOVERY alimenta la etapa PLAN con su resultado
```

### Salida de DISCOVERY

Cada Engine genera un archivo `engine_discovery.json` (temporal, no persiste entre ejecuciones) con:

```json
{
  "engine": "Discovery Engine",
  "ejecucion_id": "uuid",
  "archivos_encontrados": ["...", "..."],
  "archivos_faltantes": ["...", "..."],
  "estado_sistema": { "...": "..." },
  "memoria_relevante": ["...", "..."],
  "timestamp": "2026-07-15T10:00:00Z"
}
```

---

## Etapa 4: PLAN

### Propósito
Definir qué acciones va a ejecutar el Engine, en qué orden y con qué criterios de éxito. No es el Planner Engine — es la planificación interna de cada Engine.

### Contrato

```yaml
plan:
  estructura:
    - accion: string             # Qué va a hacer
      orden: integer             # En qué orden
      entrada: string            # Qué usa como entrada para esta acción
      salida_esperada: string    # Qué produce
      criterio_exito: string     # Cómo se sabe que funcionó
      riesgo: string             # Qué puede salir mal

  reglas:
    - PLAN debe generar un plan explícito antes de EXECUTION
    - PLAN no ejecuta nada. Solo planifica.
    - Si PLAN determina que no hay nada que hacer, el Engine debe terminar con estado NO_ACTION_NEEDED
    - PLAN debe incluir al menos una acción o indicar NO_ACTION_NEEDED
```

### Salida de PLAN

Cada Engine genera `engine_plan.json` (temporal):

```json
{
  "engine": "Discovery Engine",
  "ejecucion_id": "uuid",
  "acciones_planificadas": 3,
  "acciones": [
    {
      "accion": "escanear_directorio_src",
      "orden": 1,
      "entrada": "project_path",
      "salida_esperada": "lista_archivos_src",
      "criterio_exito": "se_encontraron_archivos",
      "riesgo": "directorio_no_existe"
    }
  ],
  "estado": "PLAN_READY",
  "timestamp": "2026-07-15T10:00:00Z"
}
```

---

## Etapa 5: EXECUTION

### Propósito
Ejecutar las acciones planificadas y producir las salidas esperadas.

### Contrato

```yaml
execution:
  comportamiento:
    - Las acciones se ejecutan en el orden definido en PLAN
    - Si una acción falla, el Engine debe decidir: continuar, reintentar o abortar
    - Cada acción debe registrar su resultado individualmente
    - EXECUTION no debe modificar archivos del proyecto sin autorización explícita
    - EXECUTION solo debe modificar archivos en su dominio definido

  reintentos:
    maximo: 3                    # Máximo de reintentos por acción
    espera: 1000                 # Milisegundos entre reintentos
    condicion: "error_reintentable"

  timeouts:
    por_accion: 30000            # 30 segundos por acción
    total: 300000                # 5 minutos totales

  reglas:
    - Si EXECUTION excede el timeout, debe fallar con ERROR_TIMEOUT
    - Si EXECUTION encuentra un error irrecuperable, debe fallar con ERROR_EXECUTION_FAILED
    - EXECUTION debe registrar progreso para poder reanudar si falla
```

### Códigos de error de EXECUTION

| Código | Significado |
|--------|-------------|
| `ERROR_TIMEOUT` | La ejecución excedió el tiempo máximo |
| `ERROR_EXECUTION_FAILED` | Una acción falló sin posibilidad de recuperación |
| `ERROR_DEPENDENCY_FAILED` | Una dependencia necesaria falló |
| `ERROR_PERMISSION_WRITE` | El Engine intentó modificar un archivo prohibido |

---

## Etapa 6: VERIFICATION

### Propósito
Confirmar que las salidas producidas cumplen los criterios de éxito definidos en PLAN y calcular el confidence score de cada afirmación generada.

### Contrato

```yaml
verification:
  validaciones_post:
    - tipo: existencia
      descripcion: Verificar que los archivos de salida se generaron
      accion_si_falla: ERROR_OUTPUT_MISSING

    - tipo: formato
      descripcion: Verificar que las salidas tienen el formato esperado
      accion_si_falla: ERROR_OUTPUT_INVALID

    - tipo: contenido
      descripcion: Verificar que las salidas contienen los datos esperados
      accion_si_falla: ERROR_OUTPUT_INVALID

    - tipo: coherencia
      descripcion: Verificar que las salidas son coherentes con las entradas
      accion_si_falla: ERROR_OUTPUT_INCONSISTENT

    - tipo: integridad
      descripcion: Verificar que no se modificaron archivos fuera del dominio
      accion_si_falla: ERROR_SIDE_EFFECT

    - tipo: confianza
      descripcion: Calcular confidence score para cada afirmación generada
      metricas:
        - evidence_count: integer
          descripcion: "Cantidad de piezas de evidencia que respaldan la afirmación"
        - source_quality: float
          descripcion: "Calidad promedio de las fuentes (0.0-1.0)"
        - consistency_score: float
          descripcion: "Consistencia interna de la evidencia (0.0-1.0)"
        - contradiction_count: integer
          descripcion: "Cantidad de contradicciones encontradas"

  calculo_confianza:
    formula: "Confidence = (source_quality * 0.4) + (consistency_score * 0.4) + (1 - (contradiction_count / max_contradictions)) * 0.2"
    umbrales:
      alto: ">= 0.85"
      medio: ">= 0.60"
      bajo: ">= 0.30"
      critico: "< 0.30"
    accion_si_bajo: "Registrar advertencia en la evidencia. El Engine puede continuar."
    accion_si_critico: "El Engine debe buscar más evidencia antes de continuar. Si no es posible, registrar como baja confianza explícitamente."

  reglas:
    - Si VERIFICATION falla, el Engine no debe continuar a EVIDENCE
    - Si VERIFICATION detecta efectos secundarios no autorizados, debe reportarlos
    - VERIFICATION debe calcular confidence score para toda afirmación generada
    - VERIFICATION debe registrar hallucination_risk = 1.0 - confidence_score
```

### Códigos de error de VERIFICATION

| Código | Significado |
|--------|-------------|
| `ERROR_OUTPUT_MISSING` | Falta un archivo de salida esperado |
| `ERROR_OUTPUT_INVALID` | Un archivo de salida tiene formato inválido |
| `ERROR_OUTPUT_INCONSISTENT` | Las salidas no son coherentes con las entradas |
| `ERROR_SIDE_EFFECT` | Se modificaron archivos fuera del dominio autorizado |

---

## Etapa 7: EVIDENCE

### Propósito
Registrar la ejecución completa del Engine en archivos de evidencia persistentes.

### Contrato

```yaml
evidence:
  archivos_a_generar:
    - path: SDOS/evidence/<engine>/<ejecucion_id>.json
      contenido:
        - engine: string
        - ejecucion_id: string
        - input_summary: object
        - validation_result: object
        - discovery_result: object
        - plan: object
        - execution_result: object
        - verification_result: object
        - errores: []
        - metricas:
            tiempo_ejecucion_ms: integer
            acciones_ejecutadas: integer
            acciones_fallidas: integer
            archivos_generados: integer
            archivos_modificados: integer
        - confianza:
            confidence_score: float
            hallucination_risk: float
            evidence_count: integer
            source_quality: float
            contradiction_count: integer
        - timestamp_inicio: string
        - timestamp_fin: string
        - estado_final: string

  reglas:
    - EVIDENCE siempre se ejecuta, incluso si EXECUTION o VERIFICATION fallaron
    - Si el Engine falló, EVIDENCE registra el error con todo el contexto disponible
    - La evidencia es inmutable. No se modifica después de generada.
    - La evidencia debe incluir métricas de ejecución
```

---

## Etapa 8: MEMORY

### Propósito
Actualizar la memoria del sistema con las decisiones, resultados y aprendizajes de esta ejecución.

### Contrato

```yaml
memory:
  actualizaciones:
    - destino: memory/ADR/
      condicion: "si se tomaron decisiones de ingeniería"
      contenido:
        - titulo: string
        - contexto: string
        - decision: string
        - alternativas_consideradas: []
        - justificacion: string
        - engine: string
        - ejecucion_id: string

    - destino: memory/context/
      condicion: "si el estado del proyecto cambió"
      contenido:
        - iteracion: integer
        - engine: string
        - cambios_realizados: []
        - estado_anterior: object
        - estado_nuevo: object

    - destino: memory/index.md
      condicion: "siempre"
      contenido:
        - actualizacion_del_indice

  reglas:
    - MEMORY no debe fallar si la memoria no existe (puede crearla)
    - MEMORY no debe borrar memoria existente. Solo agregar o actualizar.
    - MEMORY debe registrar incluso las ejecuciones que fallaron (para aprender de errores)
```

---

## Etapa 9: OUTPUT

### Propósito
Devolver el control al sistema con el código de salida correspondiente.

### Contrato

```yaml
output:
  codigos_salida:
    - codigo: 0
      estado: SUCCESS
      significado: "Engine completado exitosamente"
      accion_sdos: "Avanzar a la siguiente fase del ciclo SDOS"

    - codigo: 1
      estado: ERROR_INPUT
      significado: "Error en la etapa INPUT"
      accion_sdos: "Detener el ciclo. Requiere intervención."

    - codigo: 2
      estado: ERROR_VALIDATION
      significado: "Error en la etapa VALIDATION"
      accion_sdos: "Detener el ciclo. Requiere intervención."

    - codigo: 3
      estado: ERROR_EXECUTION
      significado: "Error en la etapa EXECUTION"
      accion_sdos: "Reintentar o invocar Repair Engine."

    - codigo: 4
      estado: ERROR_VERIFICATION
      significado: "Error en la etapa VERIFICATION"
      accion_sdos: "Invocar Repair Engine para diagnosticar."

    - codigo: 5
      estado: ERROR_SIDE_EFFECT
      significado: "Se modificaron archivos fuera del dominio"
      accion_sdos: "Detener el ciclo. Requiere intervención."

    - codigo: 10
      estado: NO_ACTION_NEEDED
      significado: "El Engine determinó que no hay nada que hacer"
      accion_sdos: "Avanzar a la siguiente fase."

    - codigo: 99
      estado: UNKNOWN_ERROR
      significado: "Error no categorizado"
      accion_sdos: "Detener el ciclo. Requiere intervención."

  salida_estandar:
    - Al finalizar, el Engine imprime un resumen en formato JSON a stdout
    - El resumen incluye: engine, estado, codigo, tiempo_ejecucion, archivos_generados
```

---

## Engine Manifest (`engine.yaml`)

Cada Engine debe incluir un archivo `engine.yaml` en su directorio raíz que describa su contrato completo.

### Formato del Manifest

```yaml
# engine.yaml
engine:
  nombre: "Discovery Engine"
  version: "1.0.0"
  responsable: "Discovery Engine"

  descripcion:
    corta: "Escanea el proyecto y genera un inventario estructurado"
    larga: "El Discovery Engine recorre el sistema de archivos del proyecto..."

  responsabilidades:
    - "Escaneo del sistema de archivos"
    - "Detección de lenguaje y framework"
    - "Detección de documentación existente"

  entradas:
    archivos:
      requeridos: []
      opcionales:
        - path: "SDOS/PROJECT_STATE.md"
          descripcion: "Estado previo si existe"
    parametros:
      - nombre: "project_path"
        tipo: "string"
        requerido: true
        descripcion: "Ruta del proyecto a escanear"

  salidas:
    archivos_generados:
      - path: "SDOS/PROJECT_STATE.md"
        condicion: "siempre"
        descripcion: "Inventario del proyecto"
    codigos_salida:
      - 0: "SUCCESS"
      - 1: "PROJECT_NOT_FOUND"

  archivos:
    puede_modificar:
      - "SDOS/PROJECT_STATE.md"
      - "SDOS/evidence/discovery/"
    prohibido_modificar:
      - "*.py"
      - "*.js"
      - "README.md"
      - "package.json"
      - "cualquier archivo fuera de SDOS/"

  dependencias:
    engines: []
    archivos: []
    sistema:
      - "acceso_lectura_al_sistema_de_archivos"

  indicadores:
    - nombre: "tiempo_ejecucion"
      descripcion: "Tiempo total de ejecución en ms"
      esperado: "< 5000"
    - nombre: "archivos_descubiertos"
      descripcion: "Cantidad de archivos encontrados"

  tiempo_esperado_ms: 5000

  nivel_srie_requerido: "L0"

  reglas_adicionales:
    - "No modifica ningún archivo del proyecto existente"
    - "Solo lee el sistema de archivos"
```

---

## Ciclo de vida completo de un Engine

```
CREADO
  │
  ▼
REGISTRADO (en SDOS, con su manifest)
  │
  ▼
INVOCADO (por SDOS o por otro Engine)
  │
  ▼
EJECUTÁNDOSE (siguiendo el ciclo universal)
  │
  ▼
FINALIZADO (con código de salida)
  │
  ├── SUCCESS → Próxima fase SDOS
  ├── ERROR → Repair Engine o intervención
  └── NO_ACTION → Próxima fase SDOS
```

---

## Garantías del contrato

1. **Determinismo** — Misma entrada produce exactamente la misma salida en todas las etapas.
2. **Idempotencia** — Ejecutar el mismo Engine dos veces con las mismas entradas no debe producir efectos secundarios acumulativos.
3. **Aislamiento** — Un Engine nunca modifica archivos fuera de su dominio definido en el manifest.
4. **Evidencia completa** — Incluso los errores generan evidencia.
5. **Contrato explícito** — El manifest es la fuente de verdad. No hay comportamiento implícito.
