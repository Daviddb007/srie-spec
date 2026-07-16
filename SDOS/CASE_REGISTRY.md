# CASE REGISTRY — Registro de Casos y Razonamiento Basado en Experiencia

---

## ¿Qué es este documento?

CASE_REGISTRY.md define el **registro de casos** de SRIE. Es el mecanismo por el cual el sistema aprende de la experiencia: cada diagnóstico, cada intervención y cada resultado se registra como un caso. Cuando el sistema enfrenta una situación similar, consulta los casos previos antes de razonar desde cero.

Este es el fundamento del **Case-Based Reasoning (CBR)** en SRIE: no aprende reglas abstractas, aprende casos concretos. Como un consultor senior que ha visto cien proyectos y reconoce patrones.

---

## ¿Qué es un caso?

Un caso es el **registro completo** de un ciclo de razonamiento: qué se observó, qué se diagnosticó, qué se hizo y qué resultado se obtuvo.

```yaml
caso:
  id: "CASE-003"
  titulo: "Deterioro de documentación por presión de features"
  fecha: "2026-06-15"
  estado: "RESUELTO"          # ABIERTO | EN_PROGRESO | RESUELTO | ARCHIVADO

  contexto:
    dominio: "gobernanza"
    nivel_madurez: "L2"
    tamano_proyecto: "mediano"
    equipo_size: 3
    antiguedad_meses: 8

  sintomas:
    - indicador: "documentation_health"
      valor_inicial: 82
      valor_final: 68
      tendencia: "DECLINING"
      periodo_dias: 30
    - indicador: "architecture_exists"
      valor: false
    - indicador: "memory_freshness"
      valor: 45

  diagnostico:
    principal: "DOCUMENTATION_DECAY"
    confianza: 0.72
    alternativas: ["MEMORY_STAGNATION", "FEATURE_PRESSURE"]

  intervencion:
    acciones:
      - "Crear ARCHITECTURE.md"
      - "Actualizar memory/index.md"
      - "Agregar documentación al definition of done"
    duracion_dias: 5
    recursos_invertidos: "2 desarrolladores × 3 días"

  resultado:
    documentation_health_post: 85
    mejoria: "+17 puntos"
    tiempo_mejoria_dias: 35
    efectos_secundarios: "Ninguno"
    satisfaccion: 0.90        # 0.0 - 1.0

  lecciones:
    - "La documentación debe ser parte del definition of done desde el inicio"
    - "Un ARCHITECTURE.md al inicio ahorra 10x tiempo después"
    - "El equipo respondió bien a la sesión semanal de documentación"

  patrones_relacionados:
    - "DOCUMENTATION_DECAY"

  tags:
    - "documentacion"
    - "gobernanza"
    - "equipo_pequeno"
```

---

## Estructura del Case Registry

```
knowledge/
├── cases/
│   ├── CASE-001.md           # Cada caso es un archivo
│   ├── CASE-002.md
│   ├── CASE-003.md
│   └── ...
│
├── cases_index.yaml           # Índice de todos los casos
│
└── case_templates/            # Plantillas para nuevos casos
    ├── template_governance.yaml
    ├── template_architecture.yaml
    └── template_security.yaml
```

### cases_index.yaml

```yaml
# knowledge/cases_index.yaml
cases:
  - id: "CASE-001"
    titulo: "Proyecto nuevo sin estructura SDOS"
    dominio: "gobernanza"
    fecha: "2026-05-01"
    estado: "RESUELTO"
    similitud_activa: null

  - id: "CASE-002"
    titulo: "Deuda técnica por dependencias desactualizadas"
    dominio: "arquitectura"
    fecha: "2026-05-15"
    estado: "RESUELTO"
    similitud_activa: null

  - id: "CASE-003"
    titulo: "Deterioro de documentación por presión de features"
    dominio: "gobernanza"
    fecha: "2026-06-15"
    estado: "RESUELTO"
    similitud_activa: 0.78   # Caso similar al diagnóstico actual

  - id: "CASE-004"
    titulo: "Dependencia con vulnerabilidad crítica"
    dominio: "seguridad"
    fecha: "2026-06-20"
    estado: "RESUELTO"

metricas:
  total_cases: 15
  resueltos: 12
  en_progreso: 2
  abiertos: 1
  ultimo_caso: "CASE-015"
  casos_por_dominio:
    gobernanza: 5
    arquitectura: 4
    seguridad: 3
    operacion: 2
    ia: 1
```

---

## Búsqueda de casos similares

Cuando el DX Engine necesita consultar experiencia previa, ejecuta una búsqueda de casos similares.

```yaml
case_search:
  entrada: "Diagnóstico actual (indicadores, dominio, contexto)"

  proceso:
    1. "Filtrar casos del mismo dominio"
    2. "Para cada caso, calcular similitud:"
       similitud = (match_sintomas × 0.40)
                 + (match_dominio × 0.25)
                 + (match_contexto × 0.20)
                 + (match_patrones × 0.15)

    3. "Ordenar por similitud descendente"
    4. "Seleccionar top 3 con similitud ≥ 0.50"
    5. "Si hay un caso con similitud ≥ 0.85: sugerir como 'CASO CASI IDÉNTICO'"

  salida:
    mejores_coincidencias:
      - caso: "CASE-003"
        similitud: 0.78
        match_sintomas: 0.82
        match_dominio: 1.0
        match_contexto: 0.70
        match_patrones: 0.60
        intervencion_usada: "Crear ARCHITECTURE.md + memory update + definition of done"
        resultado: "Documentation health mejoró +17 puntos en 35 días"
```

---

## Cálculo de similitud entre casos

```yaml
similitud_entre_casos:
  formula: |
    Similitud = (mismos_sintomas × 0.35)
              + (mismo_dominio × 0.25)
              + (mismo_contexto × 0.20)
              + (mismos_patrones × 0.20)

  donde:
    mismos_sintomas: |
      Proporción de indicadores con valores similares
      (diferencia < 10% del rango) entre ambos casos
    mismo_contexto: |
      Coincidencia en: nivel_madurez, tamano_proyecto,
      equipo_size, antiguedad
    mismos_patrones: |
      Proporción de patrones compartidos
```

---

## Ciclo de vida de un caso

```yaml
case_lifecycle:
  CREADO: "Caso creado a partir de un diagnóstico"
    │
    ▼
  ABIERTO: "Intervención en curso"
    │
    ▼
  EN_PROGRESO: "Intervención ejecutándose"
    │
    ▼
  RESUELTO: "Intervención completada. Resultados registrados."
    │
    ├── ARCHIVADO: "Caso histórico (12+ meses)"
    │
    └── REABIERTO: "Problema recurrente"
```

### Reglas del ciclo de vida

1. **Todo diagnóstico genera un caso.** No hay diagnósticos que no queden registrados.
2. **Un caso se considera RESUELTO solo cuando se registra el resultado.** No antes.
3. **Si un caso se reabre 3+ veces, se debe crear un patrón de diagnóstico nuevo.**
4. **Los casos con satisfacción < 0.50 se marcan para revisión del Knowledge Engine.**

---

## Generación automática de casos

Cuando el DX Engine completa un diagnóstico, puede generar automáticamente un caso:

```yaml
generacion_automatica:
  disparador: "Diagnóstico CONFIRMADO o MODERADO"
  proceso:
    1. "Tomar el diagnóstico actual"
    2. "Completar sintomas desde los indicadores actuales"
    3. "Registrar intervención_sugerida como acciones propuestas"
    4. "Dejar resultado en PENDIENTE (se completará cuando la intervención termine)"
    5. "Asignar siguiente ID disponible (CASE-NNN)"
    6. "Registrar en cases_index.yaml"

  ejemplo:
    diagnostico_origen: "DX-001"
    case_id: "CASE-016"
    estado_inicial: "ABIERTO"
    resultado_inicial: null  # Se completará después de la intervención
```

---

## Actualización de resultados

Cuando una intervención se completa, el caso se actualiza con los resultados:

```yaml
actualizacion_resultados:
  disparador: "Planner Engine reporta intervención completada"
  proceso:
    1. "Localizar el caso asociado al diagnóstico"
    2. "Registrar acciones_realizadas (pueden diferir de las sugeridas)"
    3. "Registrar resultado (métricas post-intervención)"
    4. "Calcular mejoria (diferencia pre-post)"
    5. "Registrar lecciones (extraídas por Knowledge Engine)"
    6. "Cambiar estado a RESUELTO"
    7. "Si mejoria < 0: marcar como FRACASO para análisis"
```

---

## Case-Based Reasoning vs Rule-Based Reasoning

| Aspecto | Rule-Based | Case-Based (SRIE) |
|---------|------------|-------------------|
| Base de conocimiento | Reglas abstractas | Casos concretos |
| Aprendizaje | Requiere reescribir reglas | Agregar nuevo caso |
| Mantenimiento | Alto (reglas interactúan) | Bajo (casos independientes) |
| Explicabilidad | "Porque la regla X dice" | "Porque el caso Y funcionó" |
| Generalización | Explícita (reglas genéricas) | Emergente (casos similares) |
| Rareza | Casos raros no cubiertos | Casos raros se registran igual |

SRIE usa **Case-Based Reasoning** como mecanismo primario y **Rule-Based** como respaldo para situaciones sin casos previos.

---

## Integración con otros componentes

```yaml
integracion:
  con_hypothesis_engine:
    - "El Hypothesis Engine consulta casos similares para generar hipótesis"
    - "Un caso con alta similitud (≥ 0.75) puede generar hipótesis directa"

  con_differential_diagnosis:
    - "El Differential Analyzer usa casos previos para calibrar confianza"
    - "Si un caso similar tuvo buena intervención, se prioriza esa línea"

  con_knowledge_engine:
    - "El Knowledge Engine consolida casos exitosos como patrones"
    - "Si 3+ casos del mismo tipo tienen éxito, se crea un patrón"

  con_evolution_engine:
    - "El Evolution Engine analiza tendencias en casos"
    - "Si un tipo de caso se repite, sugiere cambios estructurales"

  con_planner_engine:
    - "El Planner Engine usa casos previos para estimar esfuerzo"
    - "Un caso similar indica cuánto tomó la intervención anterior"
```
