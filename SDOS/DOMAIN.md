# DOMAIN — Qué es un Domain SRIE

---

## ¿Qué es este documento?

DOMAIN.md define el concepto de **Domain** dentro del sistema SRIE. Un Domain es una instancia completa y autónoma de SRIE especializada en un área de negocio o técnica. Cada Domain es un SRIE completo que puede vivir solo o unirse a una federación.

---

## ¿Qué es un Domain?

Un **Domain** es un SRIE autónomo que:

- Tiene un **propósito** específico dentro de una organización (Marketing, Ventas, Producto, Ingeniería, Finanzas, Legal, etc.)
- Ejecuta su propio **ciclo SDOS** completo
- Tiene su propia **memoria**, su propio **registry** y sus propias **capabilities**
- Puede operar **completamente solo** sin depender de otros Domains
- Puede **federarse** con otros Domains mediante la SRIE Network
- Tiene su propio **nivel de madurez** (L0 a L5)
- Puede contener **sub-Domains** (arquitectura fractal)

Un Domain no es:
- Un módulo de software
- Un microservicio
- Un equipo (aunque puede ser operado por uno)
- Una capa de la aplicación

Un Domain es un **SRIE completo y autónomo** con un dominio de negocio específico.

---

## Anatomía de un Domain

```
<domain>/
├── SDOS/                          # Sistema Operativo del Domain
│   ├── STATE.md                    # Estado del ciclo SDOS local
│   ├── PROJECT_STATE.md            # Estado del Domain
│   ├── REGISTRY.yaml               # Registry local del Domain
│   ├── GRAPH.md                    # Knowledge Graph local
│   ├── SRIE_REPORT.md              # Indicadores locales
│   └── evidence/                   # Evidencia local
│
├── capabilities/                   # Capacidades del Domain
├── memory/                         # Memoria del Domain (privada)
│   ├── index.md
│   ├── ADR/
│   └── context/
│
├── knowledge/                      # Conocimiento local
│
├── federation/                     # Conexión con la federación
│   ├── FEDERATION.yaml             # Configuración federada
│   ├── NETWORK_KEY                 # Clave de la red (si aplica)
│   └── contracts/                  # Inter-SRIE Contracts activos
│
├── DOMAIN.yaml                     # Manifest del Domain
└── README.md                       # Propósito del Domain
```

---

## Domain Manifest

```yaml
# DOMAIN.yaml
domain:
  id: "DOM-MARKETING"
  nombre: "Marketing SRIE"
  version: "1.0.0"
  estado: "ACTIVE"

  proposito: "Gestionar campañas de marketing, captura de leads y analítica de mercado"

  nivel_madurez: "L3"

  padre: null                          # ID del Domain padre (si es sub-Domain)
  hijos: []                            # IDs de sub-Domains

  capabilities_propias:
    - "lead_capture"
    - "campaign_analytics"
    - "email_marketing"

  capabilities_requeridas:
    - domain: "DOM-SALES"
      capability: "quote_generation"
      contrato: "ISC-001"

  eventos_que_produce:
    - "NEW_LEAD_CAPTURED"
    - "CAMPAIGN_COMPLETED"
    - "MARKETING_BUDGET_CHANGED"

  eventos_que_consume:
    - "NEW_PRODUCT_LAUNCHED"
    - "SALES_TARGET_CHANGED"
    - "CUSTOMER_FEEDBACK_COLLECTED"

  federation:
    conectado: true
    network_id: "NET-001"
    ultimo_heartbeat: "2026-07-15T10:00:00Z"
    confianza_promedio: 0.92

  metricas:
    capabilities_count: 3
    engines_count: 5
    memory_entries: 47
    discovery_ejecuciones: 23
```

---

## Ciclo de vida de un Domain

```yaml
# Estados
STANDALONE: "Domain operando de forma independiente"
DISCOVERED: "Domain detectado por otro Domain o por la federación"
REGISTERED: "Domain registrado en un Federation Registry"
CONNECTED: "Domain conectado al BUS de la federación"
ACTIVE: "Domain intercambiando eventos activamente"
IDLE: "Domain conectado pero sin actividad reciente (30min+)"
DEGRADED: "Domain con confianza baja en sus afirmaciones"
DISCONNECTED: "Domain desconectado voluntariamente"
ARCHIVED: "Domain eliminado del registro federado"

# Transiciones
STANDALONE → DISCOVERED: "Otro Domain o federación lo detecta"
DISCOVERED → REGISTERED: "Se registra en el Federation Registry"
REGISTERED → CONNECTED: "Se conecta al BUS"
CONNECTED → ACTIVE: "Comienza a intercambiar eventos"
ACTIVE → IDLE: "30 minutos sin actividad"
IDLE → ACTIVE: "Nuevo evento emitido o recibido"
ACTIVE → DEGRADED: "Confianza baja detectada"
DEGRADED → ACTIVE: "Confianza recuperada"
ACTIVE → DISCONNECTED: "Abandono voluntario de la federación"
DISCONNECTED → ARCHIVED: "Confirmación de salida"
```

### Reglas del ciclo de vida

1. **Un Domain nace en STANDALONE.** Siempre. Ningún Domain debería nacer ya federado.
2. **Un Domain puede estar STANDALONE indefinidamente.** No necesita la federación para operar.
3. **Un Domain en STANDALONE puede descubrir a otros y ser descubierto.** El discovery no requiere registro.
4. **Un Domain puede abandonar la federación en cualquier momento** y conservar toda su memoria.
5. **Un Domain ARCHIVED puede reactivarse** pero pierde su lugar en la federación (debe reconectarse).

---

## Nacimiento de un Domain

Un Domain nace cuando alguien (o algún Engine) decide que existe un área de la organización que merece su propia instancia de SRIE.

### Criterios para crear un Domain

```
Un Domain debe crearse cuando:
1. El área tiene un propósito claramente diferenciado
2. El área opera con suficiente autonomía (toma sus propias decisiones)
3. El área produce y consume información que otros necesitan
4. El área merece su propia memoria y evolución
```

### Proceso de creación

```yaml
creacion:
  disparador: "Decisión explícita o detección automática"
  pasos:
    1. "Definir propósito del Domain"
    2. "Crear DOMAIN.yaml"
    3. "Ejecutar srie bootstrap (SDOS local)"
    4. "Ejecutar Discovery local (escaneo del área)"
    5. "Calcular nivel de madurez inicial"
    6. "Registrar en Federation Registry (si aplica)"
    7. "Conectar al BUS (si aplica)"
```

---

## Muerte de un Domain

Un Domain muere cuando el área que representa deja de existir o se fusiona con otra.

### Criterios para archivar un Domain

```
Un Domain debe archivarse cuando:
1. El área dejó de existir
2. El área se fusionó con otro Domain
3. El área perdió toda autonomía
4. El Domain no ha tenido actividad en 6+ meses
```

### Proceso de archivo

```yaml
archivo:
  pasos:
    1. "Notificar a todos los Domains que dependen de este"
    2. "Esperar confirmación de reasignación de dependencias"
    3. "Exportar memoria a formato de archivo"
    4. "Desconectar del BUS"
    5. "Eliminar del Federation Registry"
    6. "Marcar DOMAIN.yaml como ARCHIVED"
```

---

## Sub-Domains (arquitectura fractal)

Un Domain puede contener sub-Domains. Esto crea una estructura fractal donde el mismo patrón se repite a cualquier escala.

### Ejemplo

```
SRIE Tecnología (Domain padre)
├── SDOS/
├── capabilities/
├── memory/
│
├── SRIE Backend (sub-Domain)
│   ├── SDOS/
│   ├── capabilities/
│   └── memory/
│
├── SRIE Frontend (sub-Domain)
│   ├── SDOS/
│   ├── capabilities/
│   └── memory/
│
└── SRIE Data (sub-Domain)
    ├── SDOS/
    ├── capabilities/
    └── memory/
```

### Reglas de sub-Domains

1. **Un sub-Domain es un SRIE completo.** Tiene todo lo que un Domain standalone tiene.
2. **Un sub-Domain hereda la configuración de red** del Domain padre pero puede tener la suya propia.
3. **Un sub-Domain puede abandonar al padre** y convertirse en standalone o unirse a otro Domain.
4. **El padre no puede leer la memoria interna del sub-Domain** sin un contrato explícito.
5. **El sub-Domain reporta indicadores al padre** para el cálculo de madurez global.
