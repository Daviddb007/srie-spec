# DECISION REGISTRY — Memoria de Decisiones Estratégicas

---

## ¿Qué es este documento?

DECISION_REGISTRY.md define el **registro de decisiones estratégicas** de SRIE. Mientras que los ADR (Architectural Decision Records) registran decisiones técnicas, el Decision Registry registra **decisiones de estrategia**: por qué se eligió un objetivo sobre otro, por qué se aceptó un tradeoff, por qué se priorizó un diagnóstico sobre otro.

Las organizaciones viven de decisiones, no de código. Este registro preserva la memoria estratégica del sistema.

---

## ¿Qué es una decisión estratégica?

```yaml
decision:
  id: "DEC-001"
  titulo: "Priorizar recuperación de documentación sobre nuevas features"
  timestamp: "2026-07-15T10:00:00Z"
  tipo: "priorizacion"         # estratégica | táctica | operativa

  contexto: |
    El indicador documentation_health cayó de 82 a 68 en 30 días.
    ARCHITECTURE.md no existe. La memoria no se actualiza.
    El diagnóstico DX-001 identificó DOCUMENTATION_DECAY.

  alternativas_consideradas:
    - alternativa: "Ignorar la documentación y seguir con features"
      valoracion: "Riesgo ALTO de que el deterioro continúe"
      razon_descarte: "Violaría GOAL-003 (Gobernanza sostenible)"

    - alternativa: "Documentación parcial (solo ARCHITECTURE.md)"
      valoracion: "Riesgo MEDIO de dejar memoria desactualizada"
      razon_descarte: "No resuelve la raíz del problema"

    - alternativa: "Dedicar 2 semanas completas a documentación"
      valoracion: "Riesgo BAJO, solución completa"
      seleccionada: true

  tradeoff_aplicado: "TO-001 (Velocidad vs Documentación)"
  tradeoff_valor: 0.72

  riesgos_identificados:
    - riesgo: "2 semanas sin features pueden afectar time to market"
      mitigacion: "El equipo priorizó documentación sobre features temporalmente"
      severidad: "MEDIA"

  resultado_esperado:
    - "documentation_health >= 80 en 30 días"
    - "architecture_exists = true"
    - "memory_freshness >= 70"

  resultado_obtenido:
    cumplido: true
    documentation_health_post: 85
    mejoria: "+17 puntos"
    desviacion: "+3 puntos sobre lo esperado"
    fecha_verificacion: "2026-08-19"
    lecciones: "El tradeoff fue correcto. Documentar al inicio del proyecto habría evitado el problema."

  revisada_por: "Intent Layer"
  proxima_revision: "2026-10-15"

  tags:
    - "documentacion"
    - "gobernanza"
    - "tradeoff"
```

---

## Decision Registry

Las decisiones se almacenan en `SDOS/decisions/`:

```
SDOS/
└── decisions/
    ├── index.yaml              # Índice de todas las decisiones
    ├── DEC-001.yaml            # Cada decisión es un archivo
    ├── DEC-002.yaml
    └── ...
```

### index.yaml

```yaml
# SDOS/decisions/index.yaml
decisions:
  - id: "DEC-001"
    titulo: "Priorizar recuperación de documentación"
    tipo: "priorizacion"
    fecha: "2026-07-15"
    dominio: "gobernanza"
    estado: "EXITOSA"

  - id: "DEC-002"
    titulo: "No refactorizar auth por ahora"
    tipo: "arquitectura"
    fecha: "2026-07-10"
    dominio: "seguridad"
    estado: "PENDIENTE"

  - id: "DEC-003"
    titulo: "Aceptar deuda técnica en calendar por deadline"
    tipo: "tradeoff"
    fecha: "2026-06-20"
    dominio: "desarrollo"
    estado: "FRACASO"

metricas:
  total: 15
  exitosas: 11
  pendientes: 3
  fracasos: 1
  ultima_decision: "DEC-015"
```

---

## Ciclo de vida de una decisión

```yaml
decision_lifecycle:
  PROPUESTA: "Decisión propuesta por Intent Layer o Planner"
    │
    ▼
  APROBADA: "Decisión aceptada. Intervención en curso."
    │
    ▼
  EN_EJECUCION: "La intervención asociada está ejecutándose"
    │
    ├── EXITOSA: "Resultado obtenido cumple o supera lo esperado"
    │
    ├── PARCIAL: "Resultado obtenido parcialmente (< 70% de lo esperado)"
    │
    └── FRACASO: "Resultado obtenido no cumple lo esperado"
```

---

## Consulta de decisiones

```yaml
consultas_soportadas:
  - nombre: "decisiones_por_dominio"
    query: "Todas las decisiones de un dominio"
    ejemplo: "decisiones de gobernanza"

  - nombre: "decisiones_por_resultado"
    query: "Decisiones con un resultado específico"
    ejemplo: "decisiones EXITOSAS"

  - nombre: "decisiones_recientes"
    query: "Últimas N decisiones"
    ejemplo: "últimas 5 decisiones"

  - nombre: "decisiones_similares"
    query: "Decisiones con contexto similar"
    ejemplo: "decisiones sobre documentación"

  - nombre: "decisiones_por_tradeoff"
    query: "Decisiones que usaron un tradeoff específico"
    ejemplo: "decisiones con TO-001"
```

---

## Relación con ADR

| Aspecto | ADR | Decision Registry |
|---------|-----|-------------------|
| ¿Qué registra? | Decisiones técnicas | Decisiones estratégicas |
| ¿Quién lo crea? | Memory Engine | Intent Layer + Planner |
| ¿Cuándo se crea? | Durante implementación | Durante planificación |
| ¿Naturaleza? | Técnica (código, arquitectura) | Estratégica (objetivos, tradeoffs) |
| ¿Relación? | ADR puede implementar una decisión | Decisión puede requerir ADR |

Ambos coexisten. Una decisión estratégica (ej: "priorizar documentación") puede resultar en múltiples ADR técnicos (ej: "estructura de ARCHITECTURE.md", "formato de memory").

---

## Integración con Knowledge Engine

```yaml
integracion_knowledge:
  - evento: "DECISION_EXITOSA"
    accion: "Knowledge Engine analiza por qué fue exitosa y extrae patrón"

  - evento: "DECISION_FRACASO"
    accion: "Knowledge Engine analiza por qué falló y extrae antipatrón"

  - evento: "3+ DECISIONES_SIMILARES_EXITOSAS"
    accion: "Knowledge Engine propone regla general"
```

---

## Decision Memory vs Case Registry

| Aspecto | Case Registry | Decision Registry |
|---------|---------------|-------------------|
| Nivel | Táctico (casos de diagnóstico) | Estratégico (decisiones) |
| Entrada | Diagnósticos | Intenciones + Tradeoffs |
| Salida | Patrones de diagnóstico | Memoria estratégica |
| Horizonte | Inmediato (resolver ahora) | Largo plazo (por qué hicimos esto) |
| Consumido por | DX Engine (razonamiento) | Evolution Engine (aprendizaje) |
