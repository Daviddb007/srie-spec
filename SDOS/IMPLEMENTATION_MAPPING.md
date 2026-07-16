# IMPLEMENTATION MAPPING — Traducción de Especificación a Código

---

## ¿Qué es este documento?

IMPLEMENTATION_MAPPING.md define **cómo cada especificación de SRIE se traduce a componentes de software**. No es un diseño de implementación. Es el **puente entre los 64 documentos de especificación y los objetos, servicios, contratos y procesos del Runtime**.

Cada entrada responde:
- ¿Qué componente de software implementa esto?
- ¿Es clase, servicio, contrato, esquema o proceso?
- ¿Quién lo consume? ¿Quién lo produce?
- ¿Representación en memoria? ¿Representación persistente?
- ¿Eventos que genera? ¿Dependencias?

---

## Convención de tipos

```yaml
tipos_componente:
  class: "Objeto con estado y comportamiento"
  service: "Procesador sin estado, ejecuta operaciones"
  contract: "Interfaz o protocolo que otros implementan"
  schema: "Estructura de datos (YAML/JSON)"
  process: "Pipeline o flujo de varios pasos"
  registry: "Catálogo de entidades"
  engine: "Motor de ejecución con ciclo de vida propio"
  store: "Almacenamiento persistente"
  adapter: "Traductor entre protocolos"
  manager: "Coordinador de ciclo de vida"
```

---

## Foundation (FASE 0)

```yaml
SRIE_ENGINE.md:
  tipo: contract
  clase: SRIEConfig
  descripcion: "Configuración y principios del sistema"
  representa: "Constitución del sistema"
  memoria: "SRIEConfig(principles: list, definitions: dict)"
  persistencia: "SDOS/SRIE_ENGINE.md (referencia)"
  eventos: []
  consumido_por: [Kernel, GovernanceEngine]
  producido_por: [Human]

SDOS.md:
  tipo: contract
  clase: SDOSCycle
  descripcion: "Ciclo operativo: fases, transiciones, responsables"
  memoria: "SDOSCycle(phases: list, current_phase: str)"
  persistencia: "SDOS/STATE.md"
  eventos: [PHASE_STARTED, PHASE_COMPLETED, PHASE_FAILED]
  consumido_por: [Scheduler, Engines]
  dependencias: [FOUNDATION.md, IDENTITY.md]

ENGINES.md:
  tipo: registry
  clase: EngineRegistry
  descripcion: "Catálogo de todos los motores disponibles"
  memoria: "EngineRegistry(engines: dict[str, EngineManifest])"
  persistencia: "SDOS/ENGINES.md → compilado a IR"
  eventos: [ENGINE_REGISTERED, ENGINE_DEREGISTERED]
  consumido_por: [EngineLoader, Kernel]
  dependencias: [ENGINE_CONTRACT.md]

SRIE_INDICATORS.md:
  tipo: schema
  clase: MaturityModel
  descripcion: "Modelo de madurez L0-L5 con 8 dominios"
  memoria: "MaturityModel(level: int, domains: dict)"
  persistencia: "SDOS/SRIE_INDICATORS.md → compilado a IR"
  eventos: [MATURITY_LEVEL_CHANGED]
  consumido_por: [IndicatorsEngine, GovernanceEngine]
  dependencias: [INDICATOR_CATALOG.md]
```

---

## Contratos (FASE 1)

```yaml
SRIE_CORE.md:
  tipo: contract
  clase: SRIECore
  descripcion: "Definiciones conceptuales del sistema"
  memoria: "SRIECore(definitions: dict)"
  persistencia: "SDOS/SRIE_CORE.md (referencia)"
  consumido_por: [Todos los componentes]
  eventos: []
  dependencias: [SRIE_ENGINE.md]

ENGINE_CONTRACT.md:
  tipo: contract
  clase: EngineContract
  descripcion: "Ciclo universal: INPUT→VALIDATION→DISCOVERY→PLAN→EXECUTION→VERIFICATION→EVIDENCE→MEMORY→OUTPUT"
  memoria: "EngineContract(stages: list)"
  persistencia: "SDOS/ENGINE_CONTRACT.md → compilado a IR"
  consumido_por: [Todo Engine, EngineLoader]
  eventos: []
  dependencias: []

CAPABILITY_MANIFEST.md:
  tipo: schema
  clase: CapabilityManifest
  descripcion: "Estructura y contrato de una capability"
  memoria: "CapabilityManifest(name, version, dependencies, files)"
  persistencia: "capabilities/<name>/<name>.manifest"
  eventos: [CAPABILITY_CREATED, CAPABILITY_UPDATED, CAPABILITY_DEPRECATED]
  consumido_por: [CapabilityEngine, SandboxEngine, ResourceBroker]
  dependencias: [PROJECT_CONTRACT.md]

PROJECT_CONTRACT.md:
  tipo: schema
  clase: ProjectContract
  descripcion: "Estructura obligatoria del proyecto SRIE"
  memoria: "ProjectContract(required_dirs, required_files)"
  persistencia: "SDOS/PROJECT.yaml"
  eventos: []
  consumido_por: [InitProcess, DiscoveryEngine, GovernanceEngine]
  dependencias: [IDENTITY.md]
```

---

## Meta Modelo (FASE 1.5)

```yaml
GRAPH.md:
  tipo: class
  clase: KnowledgeGraph
  descripcion: "Grafo dirigido de nodos (Project, Domain, Capability) y aristas (contiene, depende_de, afecta)"
  memoria: "KnowledgeGraph(nodes: dict, edges: list)"
  persistencia: "SDOS/GRAPH.md, SDOS/discovery/PROJECT_GRAPH.json"
  eventos: [NODE_ADDED, NODE_UPDATED, NODE_DESYNCED, EDGE_ADDED, EDGE_REMOVED]
  consumido_por: [Todos los Engines]
  dependencias: [ONTOLOGY.md, REGISTRY.md]

DOMAINS.md:
  tipo: schema
  clase: DomainRegistry
  descripcion: "11 dominios con archivos, indicadores y jerarquía"
  memoria: "DomainRegistry(domains: dict[str, Domain])"
  persistencia: "SDOS/DOMAINS.md → compilado a IR"
  consumido_por: [DiscoveryEngine, IndicatorsEngine, Classifier]
  dependencias: [ONTOLOGY.md]

EVENTS.md:
  tipo: schema
  clase: EventRegistry
  descripcion: "Catálogo de todos los eventos del sistema (40+)"
  memoria: "EventRegistry(events: dict[str, EventDefinition])"
  persistencia: "SDOS/EVENTS.md → compilado a IR"
  consumido_por: [EventLoop, Engines, Bus]
  dependencias: [ONTOLOGY.md]

STATES.md:
  tipo: schema
  clase: StateMachine
  descripcion: "5 máquinas de estado (proyecto, capability, engine, artifact, nodo)"
  memoria: "StateMachine(states: dict, transitions: list)"
  persistencia: "SDOS/STATES.md → compilado a IR"
  consumido_por: [LifecycleManager, Scheduler, Engines]
  dependencias: [EVENTS.md]

ARTIFACTS.md:
  tipo: schema
  clase: ArtifactRegistry
  descripcion: "12 tipos de artefacto con formato y ciclo de vida"
  memoria: "ArtifactRegistry(types: dict)"
  persistencia: "SDOS/ARTIFACTS.md → compilado a IR"
  consumido_por: [Engines, Publisher, EvidenceRecorder]
  dependencias: [ONTOLOGY.md]
```

---

## Semántica (FASE 1.8)

```yaml
REGISTRY.md:
  tipo: class
  clase: Registry
  descripcion: "Catálogo central con IDs únicos (ENG-001, CAP-001, DOC-001...)"
  memoria: "Registry(entities: dict[str, Entity])"
  persistencia: "SDOS/REGISTRY.yaml"
  eventos: [ENTITY_REGISTERED, ENTITY_UPDATED, ENTITY_ARCHIVED]
  consumido_por: [Todos los Engines, ResourceBroker, GovernanceEngine]
  dependencias: [ONTOLOGY.md]

ONTOLOGY.md:
  tipo: schema
  clase: Ontology
  descripcion: "Tipos de entidad, relaciones válidas/prohibidas, herencia, cardinalidades"
  memoria: "Ontology(entity_types: dict, valid_relations: list, constraints: list)"
  persistencia: "SDOS/ONTOLOGY.yaml"
  eventos: [ONTOLOGY_CHANGED]
  consumido_por: [KnowledgeGraph, Registry, DiscoveryEngine, Classifier, Correlator]
  dependencias: []
```

---

## Discovery Engine (FASE 2)

```yaml
DISCOVERY_ENGINE.md:
  tipo: engine
  clase: DiscoveryEngine
  descripcion: "Sistema sensorial de SRIE"
  memoria: "DiscoveryEngine(state: EngineState)"
  persistencia: "SDOS/evidence/discovery/"
  eventos: [PROJECT_DISCOVERED, DISCOVERY_UPDATED, DISCOVERY_FAILED]
  consumido_por: [IndicatorsEngine, KnowledgeGraph]
  dependencias: [ENGINE_CONTRACT.md, REGISTRY.md, ONTOLOGY.md, DOMAINS.md]

DISCOVERY_PROTOCOL.md:
  tipo: process
  clase: DiscoveryProtocol
  descripcion: "Observe → Classify → Correlate → Validate → Publish"
  memoria: "DiscoveryProtocol(stages: list)"
  persistencia: "(transitorio, cada stage produce archivos)"
  consumido_por: [DiscoveryEngine]
  dependencias: [DISCOVERY_EVIDENCE.md, GRAPH.md]

DISCOVERY_EVIDENCE.md:
  tipo: contract
  clase: EvidenceValidator
  descripcion: "Qué constituye evidencia válida, fórmula de confidence"
  memoria: "EvidenceValidator(criteria: dict)"
  persistencia: "SDOS/DISCOVERY_EVIDENCE.md → compilado a IR"
  consumido_por: [DiscoveryEngine, IndicatorsEngine, DXEngine, HypothesisEngine]
  dependencias: []

DISCOVERY_OUTPUTS.md:
  tipo: schema
  clase: DiscoveryOutputs
  descripcion: "9 artefactos que genera Discovery"
  memoria: "DiscoveryOutputs(schemas: dict)"
  persistencia: "SDOS/DISCOVERY_OUTPUTS.md → compilado a IR"
  consumido_por: [DiscoveryEngine, Publisher]
  dependencias: [ARTIFACTS.md]

DISCOVERY_PIPELINE.md:
  tipo: process
  clase: DiscoveryPipeline
  descripcion: "Pipeline incremental: EventBus→Dispatcher→Hasher→ChangeDetector→Protocol→Publisher→Cache"
  memoria: "DiscoveryPipeline(state: PipelineState)"
  persistencia: "SDOS/discovery/DISCOVERY_CACHE.yaml"
  eventos: [DISCOVERY_LOCKED, DISCOVERY_TIMEOUT]
  consumido_por: [Scheduler, Kernel]
  dependencias: [EVENT_LOOP.md, DISCOVERY_PROTOCOL.md]
```

---

## Federation (FASE 2.5)

```yaml
FEDERATION.md:
  tipo: contract
  clase: FederationConfig
  descripcion: "Modelo de federación: autonomía + federación + fractalidad + aislamiento + eventos"
  memoria: "FederationConfig(principles: list, architecture: dict)"
  persistencia: "SDOS/FEDERATION.md → compilado a IR"
  consumido_por: [NetworkManager, Bus, DomainManager]
  dependencias: [ONTOLOGY.md, IDENTITY.md]

DOMAIN.md:
  tipo: class
  clase: Domain
  descripcion: "Instancia autónoma de SRIE especializada en un área"
  memoria: "Domain(id, name, type, parent, children, capabilities, state)"
  persistencia: "DOMAIN.yaml (de cada Domain)"
  eventos: [DOMAIN_CREATED, DOMAIN_JOINED, DOMAIN_LEFT, DOMAIN_DEGRADED]
  consumido_por: [FederationManager, NetworkManager, GovernanceEngine]
  dependencias: [IDENTITY.md, FEDERATION.md, SDOS.md]

NETWORK.md:
  tipo: service
  clase: NetworkManager
  descripcion: "Topología de malla parcial, discovery local/remoto, heartbeat"
  memoria: "NetworkManager(peers: dict, connections: list)"
  persistencia: "SDOS/federation/FEDERATION_MAP.md"
  eventos: [DOMAIN_DISCOVERED, HEARTBEAT_RECEIVED, CONNECTION_LOST]
  consumido_por: [Bus, DiscoveryFederation]
  dependencias: [DOMAIN.md, BUS.md]

INTER_SRIE_CONTRACT.md:
  tipo: contract
  clase: InterSRIEContract
  descripcion: "Contrato entre Domains: capacidades, eventos, confianza, sanciones"
  memoria: "InterSRIEContract(id, parties, capabilities, events, metrics, sanctions)"
  persistencia: "SDOS/federation/contracts/ISC-<id>.yaml"
  eventos: [CONTRACT_PROPOSED, CONTRACT_ACTIVATED, CONTRACT_SUSPENDED, CONTRACT_TERMINATED]
  consumido_por: [Bus, DomainManager, GovernanceEngine]
  dependencias: [DOMAIN.md, RESOURCE_CONTRACT.md]

BUS.md:
  tipo: service
  clase: Bus
  descripcion: "4 canales (events/commands/queries/control), 4 tipos de mensaje"
  memoria: "Bus(channels: dict, queues: dict)"
  persistencia: "SDOS/evidence/bus/"
  eventos: [MESSAGE_SENT, MESSAGE_RECEIVED, MESSAGE_FAILED]
  consumido_por: [Todos los Domains, EventLoop]
  dependencias: [NETWORK.md, INTER_SRIE_CONTRACT.md, EVENTS.md]

DISCOVERY_FEDERATION.md:
  tipo: process
  clase: DiscoveryFederation
  descripcion: "3 niveles: auto-descubrimiento, descubrimiento de red, descubrimiento de capacidades"
  memoria: "DiscoveryFederation(state: dict)"
  persistencia: "SDOS/federation/FEDERATION_MAP.md, SDOS/federation/CAPABILITY_CATALOG.md"
  eventos: [DOMAIN_DISCOVERED, CAPABILITY_DISCOVERED]
  consumido_por: [DiscoveryEngine, NetworkManager, ResourceBroker]
  dependencias: [DISCOVERY_PROTOCOL.md, NETWORK.md, INTER_SRIE_CONTRACT.md]
```

---

## Institucional (FASE 2.8)

```yaml
IDENTITY.md:
  tipo: class
  clase: Identity
  descripcion: "Identidad del nodo: domain_id, tipo, cadena de firmas, permisos"
  memoria: "Identity(domain_id, type, parent, authority, chain)"
  persistencia: "SDOS/IDENTITY.yaml"
  eventos: [IDENTITY_CREATED, IDENTITY_VALIDATED, IDENTITY_EXPIRED, IDENTITY_REVOKED]
  consumido_por: [Kernel, InitProcess, Todos los Engines]
  dependencias: [ONTOLOGY.md]

GOVERNANCE.md:
  tipo: class
  clase: GovernancePolicy
  descripcion: "Reglas de gobierno: Domains, Capabilities, Registry, Ontología, Contratos, Conflictos"
  memoria: "GovernancePolicy(rules: dict)"
  persistencia: "SDOS/GOVERNANCE.md → compilado a IR"
  consumido_por: [GovernanceEngine, PolicyEngine, DomainManager]
  dependencias: [IDENTITY.md, AUTHORITY.md, SELF_GOVERNANCE.md]

AUTHORITY.md:
  tipo: schema
  clase: Authority
  descripcion: "5 niveles (CORE→FEDERATION→ORGANIZATION→DOMAIN→CAPABILITY), matriz de poderes"
  memoria: "Authority(level: str, permissions: dict)"
  persistencia: "SDOS/AUTHORITY.md → compilado a IR"
  consumido_por: [Identity, GovernanceEngine, PolicyEngine, ResourceBroker]
  dependencias: [IDENTITY.md]
```

---

## Indicators (FASE 3)

```yaml
FOUNDATION.md:
  tipo: schema
  clase: Foundation
  descripcion: "Núcleo estable: 30 documentos congelados"
  memoria: "Foundation(version: str, documents: list, compatibility: dict)"
  persistencia: "SDOS/FOUNDATION.yaml"
  eventos: [FOUNDATION_CHANGED]
  consumido_por: [Kernel, Compiler]
  dependencias: []

INDICATOR_CATALOG.md:
  tipo: schema
  clase: IndicatorCatalog
  descripcion: "43 indicadores declarativos en 4 niveles (observación/salud/riesgo/predictivo)"
  memoria: "IndicatorCatalog(indicators: dict[str, Indicator])"
  persistencia: "SDOS/INDICATOR_CATALOG.yaml"
  consumido_por: [IndicatorsEngine]
  dependencias: [INDICATOR_MODEL.md]

INDICATOR_MODEL.md:
  tipo: class
  clase: Indicator
  descripcion: "Modelo de medición: valor, confianza, tendencia, threshold, evidencia"
  memoria: "Indicator(id, name, level, domain, value, confidence, trend, threshold)"
  persistencia: "SDOS/evidence/indicators/<id>.json"
  consumido_por: [IndicatorsEngine, DXEngine, KnowledgeEngine]
  dependencias: [DIGITAL_TWIN.md]

INDICATOR_ENGINE.md:
  tipo: engine
  clase: IndicatorsEngine
  descripcion: "Motor de indicadores: CatalogLoader→TwinReader→FormulaEngine→TrendCalculator→ThresholdDetector→Publisher"
  memoria: "IndicatorsEngine(state: EngineState)"
  persistencia: "SDOS/evidence/indicators/"
  eventos: [INDICATORS_CALCULATED, INDICATORS_PARTIAL, todos los *_WARNING/CRITICAL/DEGRADING]
  consumido_por: [DXEngine, GovernanceEngine, KnowledgeEngine]
  dependencias: [ENGINE_CONTRACT.md, INDICATOR_CATALOG.md, DIGITAL_TWIN.md]

INDICATOR_PIPELINE.md:
  tipo: process
  clase: IndicatorsPipeline
  descripcion: "Dispatcher→TwinDiff→IndicatorSelector→FormulaEngine→Trend→Threshold→Scores→TwinUpdate→Publish"
  memoria: "IndicatorsPipeline(state: PipelineState)"
  persistencia: "SDOS/evidence/indicators/INDICATORS_CACHE.yaml"
  consumido_por: [Scheduler, Kernel]
  dependencias: [EVENT_LOOP.md, INDICATOR_ENGINE.md]

INDICATOR_EVENTS.md:
  tipo: schema
  clase: IndicatorEventRegistry
  descripcion: "50+ eventos del sistema de indicadores"
  memoria: "IndicatorEventRegistry(events: dict)"
  consumido_por: [EventLoop, DXEngine, KnowledgeEngine, GovernanceEngine]
  dependencias: [EVENTS.md, INDICATOR_CATALOG.md]

INDICATOR_REPORTS.md:
  tipo: schema
  clase: IndicatorReports
  descripcion: "5 artefactos (SRIE_REPORT, DIGITAL_TWIN, Evidence, Trend, Cache)"
  memoria: "IndicatorReports(schemas: dict)"
  consumido_por: [Publisher, DXEngine, PlannerEngine]
  dependencias: [ARTIFACTS.md, INDICATOR_MODEL.md]
---

## Razonamiento (FASE 3.5)

```yaml
HYPOTHESIS_ENGINE.md:
  tipo: service
  clase: HypothesisEngine
  descripcion: "Genera hipótesis desde indicadores (directas, compuestas, por patrón, por caso)"
  memoria: "HypothesisEngine(hypotheses: list)"
  persistencia: "SDOS/evidence/hypothesis/"
  eventos: [HYPOTHESIS_GENERATED, HYPOTHESIS_VALIDATED, HYPOTHESIS_DISCARDED]
  consumido_por: [DXEngine, DifferentialDiagnosis]
  dependencias: [INDICATOR_ENGINE.md, DIAGNOSIS_PATTERNS.md, CASE_REGISTRY.md]

DIFFERENTIAL_DIAGNOSIS.md:
  tipo: process
  clase: DifferentialDiagnosis
  descripcion: "EvidenceMatcher→ConfidenceCalculator→DifferentialAnalyzer→Diagnosis"
  memoria: "DifferentialDiagnosis(diagnoses: list)"
  persistencia: "SDOS/evidence/dx/"
  eventos: [DIAGNOSIS_READY, DIAGNOSIS_UNCERTAIN, DIAGNOSIS_SPECULATIVE]
  consumido_por: [DXEngine, IntentLayer, PlannerEngine]
  dependencias: [HYPOTHESIS_ENGINE.md, DIAGNOSIS_PATTERNS.md, CASE_REGISTRY.md]

DIAGNOSIS_PATTERNS.md:
  tipo: registry
  clase: DiagnosisPatternRegistry
  descripcion: "14 patrones iniciales en 6 dominios, almacenados en knowledge/diagnosis/"
  memoria: "DiagnosisPatternRegistry(patterns: dict)"
  persistencia: "knowledge/diagnosis/<dominio>/<patron>.md"
  consumido_por: [HypothesisEngine, DifferentialDiagnosis, KnowledgeEngine]
  dependencias: [CASE_REGISTRY.md]

CASE_REGISTRY.md:
  tipo: registry
  clase: CaseRegistry
  descripcion: "Case-Based Reasoning: casos completos con contexto, diagnóstico, intervención, resultado"
  memoria: "CaseRegistry(cases: dict, index: dict)"
  persistencia: "knowledge/cases/CASE-<id>.md"
  eventos: [CASE_CREATED, CASE_RESOLVED, CASE_REOPENED]
  consumido_por: [HypothesisEngine, DifferentialDiagnosis, KnowledgeEngine, PlannerEngine, SimulationEngine]
  dependencias: []
```

---

## Intent Layer (FASE 3.8)

```yaml
INTENT_MODEL.md:
  tipo: service
  clase: IntentResolver
  descripcion: "Toma diagnóstico + PROJECT_INTENT.md → Intención priorizada"
  memoria: "IntentResolver(intents: list)"
  persistencia: "SDOS/evidence/intent/"
  eventos: [INTENT_READY, INTENT_INVALIDATED]
  consumido_por: [PlannerEngine, GovernanceEngine]
  dependencias: [DIFFERENTIAL_DIAGNOSIS.md, GOALS.md, CONSTRAINTS.md, PRIORITIES.md, TRADEOFFS.md, DECISION_REGISTRY.md]

GOALS.md:
  tipo: schema
  clase: GoalRegistry
  descripcion: "7 objetivos estratégicos con pesos, conflictos y sinergias"
  memoria: "GoalRegistry(goals: dict)"
  persistencia: "SDOS/GOALS.md → compilado a IR, SDOS/PROJECT_INTENT.md"
  consumido_por: [IntentResolver, PlannerEngine]
  dependencias: []

PRIORITIES.md:
  tipo: schema
  clase: PriorityCalculator
  descripcion: "Fórmula de priorización (impacto×0.30 + urgencia×0.25 + alineación×0.20 − esfuerzo×0.15 + dependencias×0.10)"
  memoria: "PriorityCalculator(formula: callable)"
  persistencia: "SDOS/PRIORITIES.md → compilado a IR"
  consumido_por: [IntentResolver, Scheduler, PlannerEngine]
  dependencias: [GOALS.md]

CONSTRAINTS.md:
  tipo: registry
  clase: ConstraintRegistry
  descripcion: "7 restricciones (técnicas, operativas, gobierno, seguridad, negocio, federación)"
  memoria: "ConstraintRegistry(constraints: dict)"
  persistencia: "SDOS/CONSTRAINTS.md → compilado a IR"
  eventos: [CONSTRAINT_VIOLATED]
  consumido_por: [IntentResolver, PolicyEngine, PlannerEngine]
  dependencias: [IDENTITY.md, GOVERNANCE.md]

TRADEOFFS.md:
  tipo: service
  clase: TradeoffAnalyzer
  descripcion: "Evalúa tradeoffs entre opciones con fórmula de valor"
  memoria: "TradeoffAnalyzer(tradeoffs: list)"
  persistencia: "SDOS/evidence/tradeoffs/"
  consumido_por: [IntentResolver, PlannerEngine, DecisionRegistry]
  dependencias: [GOALS.md, CONSTRAINTS.md, CASE_REGISTRY.md]

DECISION_REGISTRY.md:
  tipo: registry
  clase: DecisionRegistry
  descripcion: "Memoria estratégica: contexto, alternativas, razón, tradeoff, riesgos, resultado"
  memoria: "DecisionRegistry(decisions: dict)"
  persistencia: "SDOS/decisions/DEC-<id>.yaml"
  eventos: [DECISION_MADE, DECISION_EXITOSA, DECISION_FRACASO]
  consumido_por: [IntentResolver, KnowledgeEngine, EvolutionEngine]
  dependencias: [CASE_REGISTRY.md]
```

---

## Resource Layer (FASE 3.9)

```yaml
RESOURCES.md:
  tipo: schema
  clase: ResourceModel
  descripcion: "Modelo unificado de Resource (Tool, API, Service, External, Federated)"
  memoria: "ResourceModel(types: dict)"
  consumido_por: [ResourceBroker, ResourceRegistry]
  dependencias: [RESOURCE_CONTRACT.md, RESOURCE_REGISTRY.md]

RESOURCE_REGISTRY.md:
  tipo: registry
  clase: ResourceRegistry
  descripcion: "Catálogo central de recursos con ID, estado, health, permisos"
  memoria: "ResourceRegistry(resources: dict[str, Resource])"
  persistencia: "SDOS/resource/RESOURCE_REGISTRY.yaml"
  eventos: [RESOURCE_REGISTERED, RESOURCE_DEGRADED, RESOURCE_OFFLINE]
  consumido_por: [ResourceBroker, ExecutionLayer, PolicyEngine]
  dependencias: [RESOURCES.md, RESOURCE_CONTRACT.md]

RESOURCE_BROKER.md:
  tipo: service
  clase: ResourceBroker
  descripcion: "Resuelve 'necesito X' → 'mejor recurso' con score de selección"
  memoria: "ResourceBroker(cache: dict)"
  eventos: [RESOLUTION_SUCCESS, RESOLUTION_FAILED, NO_RESOURCE_AVAILABLE]
  consumido_por: [ExecutionLayer, CapabilityEngine, PlannerEngine]
  dependencias: [RESOURCE_REGISTRY.md, RESOURCE_CONTRACT.md, NETWORK.md]

RESOURCE_CONTRACT.md:
  tipo: contract
  clase: ResourceContract
  descripcion: "4 métodos: execute(), health(), capabilities(), schema()"
  memoria: "ResourceContract(interface: dict)"
  consumido_por: [ResourceBroker, AdapterSelector, Todo adapter]
  dependencias: []

EXECUTION_LAYER.md:
  tipo: service
  clase: ExecutionLayer
  descripcion: "Action Plan → Resource Broker → Adapter Selector → Adapter Executor → Result Normalizer → Evidence Recorder"
  memoria: "ExecutionLayer(state: ExecutionState)"
  eventos: [EXECUTION_SUCCESS, EXECUTION_FAILED, EXECUTION_TIMEOUT]
  consumido_por: [CapabilityEngine, SandboxEngine, PlannerEngine]
  dependencias: [RESOURCE_BROKER.md, ADAPTERS.md, RESOURCE_CONTRACT.md]

ADAPTERS.md:
  tipo: registry
  clase: AdapterRegistry
  descripcion: "6 adaptadores (MCP, REST, CLI, SSH, SQL, SDK)"
  memoria: "AdapterRegistry(adapters: dict)"
  persistencia: "SDOS/resource/adapters/registry.yaml"
  consumido_por: [ExecutionLayer, AdapterSelector]
  dependencias: [RESOURCE_CONTRACT.md]
```

---

## Runtime (FASE 4)

```yaml
SELF_GOVERNANCE.md:
  tipo: contract
  clase: SelfGovernance
  descripcion: "Meta-gobernanza: cómo SRIE se modifica a sí mismo"
  memoria: "SelfGovernance(levels: dict, process: dict)"
  consumido_por: [GovernanceEngine, Compiler, Kernel]
  dependencias: [GOVERNANCE.md, FOUNDATION.md]

ECOSYSTEM_VERSION.md:
  tipo: schema
  clase: EcosystemVersion
  descripcion: "Versionado semántico del ecosistema completo"
  memoria: "EcosystemVersion(versions: dict)"
  persistencia: "SDOS/ECOSYSTEM_VERSION.yaml"
  consumido_por: [Kernel, Compiler, ResourceBroker]
  dependencias: [FOUNDATION.md]

TIMELINE.md:
  tipo: service
  clase: TimelineNavigator
  descripcion: "Pasado (PROJECT_TIMELINE), Presente (Digital Twin), Futuro (PROJECTION)"
  memoria: "TimelineNavigator(snapshots: dict)"
  eventos: [SNAPSHOT_CREATED, TREND_DETECTED]
  consumido_por: [PlannerEngine, DXEngine, KnowledgeEngine, EvolutionEngine]
  dependencias: [DIGITAL_TWIN.md, PROJECT_TIMELINE.md, SIMULATION_ENGINE.md]

RUNTIME.md:
  tipo: service
  clase: Runtime
  descripcion: "Runtime: Kernel→EventLoop→Scheduler→EngineLoader→Lifecycle→Memory→Knowledge→Twin→Broker→Execution→Observability"
  memoria: "Runtime(state: RuntimeState, manifest: RuntimeManifest)"
  persistencia: "SDOS/RUNTIME.yaml"
  eventos: [RUNTIME_STARTED, RUNTIME_STOPPED, RUNTIME_DEGRADED, RUNTIME_FAILED]
  consumido_por: [System]
  dependencias: [KERNEL.md, EVENT_LOOP.md, SCHEDULER.md, LIFECYCLE.md, OBSERVABILITY.md]

KERNEL.md:
  tipo: service
  clase: Kernel
  descripcion: "Arranque (13 pasos), apagado, Engine Loader, Heartbeat"
  memoria: "Kernel(state: KernelState)"
  eventos: [ENGINE_LOADED, ENGINE_FAILED]
  consumido_por: [Runtime]
  dependencias: [RUNTIME.md]

EVENT_LOOP.md:
  tipo: service
  clase: EventLoop
  descripcion: "Input Queue → Priority Queue → Delay Queue → Dispatcher (pub/sub) → Dead Letter"
  memoria: "EventLoop(queues: dict, subscribers: dict)"
  eventos: [EVENT_RECEIVED, EVENT_DISPATCHED, EVENT_FAILED, EVENT_DEAD_LETTER]
  consumido_por: [Kernel, Runtime]
  dependencias: [EVENTS.md]

SCHEDULER.md:
  tipo: service
  clase: Scheduler
  descripcion: "4 tipos de tarea, ciclo SDOS, tareas cron, límites de concurrencia"
  memoria: "Scheduler(tasks: dict, schedule: dict)"
  eventos: [TASK_STARTED, TASK_COMPLETED, TASK_FAILED, TASK_TIMEOUT]
  consumido_por: [Kernel, Runtime]
  dependencias: [EVENT_LOOP.md, SDOS.md]

LIFECYCLE.md:
  tipo: service
  clase: LifecycleManager
  descripcion: "7 estados, 10 transiciones, políticas de recuperación"
  memoria: "LifecycleManager(state: LifecycleState)"
  eventos: [STATE_CHANGED, RECOVERY_STARTED, RECOVERY_FAILED]
  consumido_por: [Kernel, Runtime, GovernanceEngine]
  dependencias: [STATES.md]

OBSERVABILITY.md:
  tipo: service
  clase: Observability
  descripcion: "Logs estructurados (JSON Lines), 20+ métricas, traces con spans"
  memoria: "Observability(config: ObservabilityConfig)"
  persistencia: "SDOS/logs/, SDOS/metrics/, SDOS/traces/"
  consumido_por: [Kernel, Runtime, Todos los componentes]
  dependencias: [EVENTS.md, ARTIFACTS.md]

POLICY_ENGINE.md:
  tipo: engine
  clase: PolicyEngine
  descripcion: "9 políticas ejecutables en 3 categorías (pre/post/continuous)"
  memoria: "PolicyEngine(policies: dict)"
  persistencia: "SDOS/policies/POL-<id>.yaml"
  eventos: [POLICY_VIOLATED, POLICY_EXCEPTIONED]
  consumido_por: [Kernel, ExecutionLayer, GovernanceEngine]
  dependencias: [CONSTRAINTS.md, GOVERNANCE.md]

SIMULATION_ENGINE.md:
  tipo: engine
  clase: SimulationEngine
  descripcion: "Proyecta estado futuro bajo escenarios, compara alternativas"
  memoria: "SimulationEngine(cache: dict)"
  eventos: [SIMULATION_COMPLETED, PROJECTION_UPDATED]
  consumido_por: [PlannerEngine, IntentResolver, DXEngine]
  dependencias: [DIGITAL_TWIN.md, CASE_REGISTRY.md, INDICATOR_CATALOG.md, DIAGNOSIS_PATTERNS.md]
```
