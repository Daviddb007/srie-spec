# GOVERNANCE — Gobierno del Ecosistema SRIE

---

## ¿Qué es este documento?

GOVERNANCE.md define **las reglas de gobierno** del ecosistema SRIE. Mientras que SRIE_ENGINE.md define la filosofía y los principios, GOVERNANCE.md define **las reglas operativas**: quién puede hacer qué, cómo se toman las decisiones, cómo se resuelven los conflictos y cómo se mantiene la coherencia del sistema a medida que crece.

---

## Principios de gobernanza

1. **La autoridad es jerárquica.** Core → Federation → Organization → Domain → Capability. Nunca al revés.
2. **Las decisiones se toman en el nivel más bajo posible.** Un Domain no debería escalar lo que puede decidir solo.
3. **Toda decisión deja evidencia.** No hay decisiones sin registro.
4. **Las reglas son explícitas.** No hay gobierno implícito. Si no está escrito, no es una regla.
5. **La gobernanza es auditable.** Cualquier decisión puede ser revisada por el nivel superior.

---

## Ámbitos de gobierno

### 1. Gobierno de Domains

```yaml
gobierno_domains:
  crear_domain:
    quien: "Cualquier Domain con authority ≥ Organization"
    requisitos:
      - "Propósito claramente definido"
      - "Propuesta de charter"
      - "Nivel de madurez mínimo L1 del padre"
    proceso:
      - "Solicitar creación al padre"
      - "Padre valida: ¿el propósito no duplica otro Domain?"
      - "Padre aprueba o rechaza"
      - "Si aprueba: crear identidad, registrar en federation registry"

  eliminar_domain:
    quien: "Solo el padre del Domain"
    requisitos:
      - "Domain no tiene sub-Domains activos"
      - "Domain no tiene contratos activos"
      - "Memoria exportada"
    proceso:
      - "Padre notifica al Domain con 30 días de anticipación"
      - "Domain exporta memoria"
      - "Padre revoca identidad"
      - "Padre elimina del federation registry"

  modificar_domain:
    quien: "El Domain mismo (cambios no estructurales)"
    restricciones:
      - "No puede cambiar domain_id"
      - "No puede cambiar tipo"
      - "No puede cambiar parent"
      - "Puede cambiar mission, charter, capabilities"
```

### 2. Gobierno de Capabilities

```yaml
gobierno_capabilities:
  crear_capability:
    quien: "Cualquier Domain con authority ≥ Domain"
    proceso:
      - "Domain verifica que no existe capability duplicada"
      - "Domain crea capability localmente"
      - "Si la capability será pública, registra en federation registry"

  publicar_capability:
    quien: "Domain propietario"
    requisitos:
      - "Capability en estado STABLE"
      - "Manifest completo"
      - "Tests pasan"
      - "Confianza ≥ 0.80"
    proceso:
      - "Publicar en el catálogo de capacidades del Domain"
      - "Si aplica, publicar en el catálogo federado"

  consumir_capability:
    quien: "Cualquier Domain con contrato activo"
    proceso:
      - "Domain consulta catálogo"
      - "Domain negocia Inter-SRIE Contract"
      - "Domain usa la capability según el contrato"
```

### 3. Gobierno del Registry

```yaml
gobierno_registry:
  modificar_registry_local:
    quien: "El Domain propietario"
    alcance: "Entidades locales del Domain"
    proceso: "Libre, dentro de las reglas del Registry"

  modificar_federation_registry:
    quien: "Solo SRIE Core o Federation"
    alcance: "Entidades federadas (Domains, contratos)"
    proceso: |
      - Solo el Core puede agregar/eliminar organizaciones
      - Solo la Federation puede agregar/eliminar Domains
      - Los Domains pueden actualizar su propia información

  consultar_registry:
    quien: "Cualquier Domain autenticado"
    alcance: "Cualquier registry (local, federation, global)"
```

### 4. Gobierno de la Ontología

```yaml
gobierno_ontologia:
  modificar_ontologia:
    quien: "Solo SRIE Core"
    proceso: |
      - Cualquier Domain puede proponer cambios
      - El Core evalúa: ¿rompe compatibilidad? ¿afecta contratos existentes?
      - Si hay breaking changes, se versiona la ontología
      - Los Domains existentes no se migran automáticamente
      - Cada Domain elige cuándo migrar a la nueva versión

  versiones_ontologia:
    - version: "1.0"
      estado: "VIGENTE"
    - version: "2.0"
      estado: "PROPUESTA"
```

### 5. Gobierno de Contratos

```yaml
gobierno_contratos:
  proponer_contrato:
    quien: "Cualquier Domain con authority ≥ Domain"
    proceso: "Ver INTER_SRIE_CONTRACT.md"

  aceptar_contrato:
    quien: "Domain proveedor"
    proceso: "Verifica que el consumidor cumple los requisitos"

  suspender_contrato:
    quien: "Cualquiera de las partes"
    causas:
      - "Confianza por debajo del umbral"
      - "Incumplimiento de evidencias requeridas"
      - "Timeout reiterado"

  terminar_contrato:
    quien: "Cualquiera de las partes"
    causas:
      - "Expiración del contrato"
      - "Mutuo acuerdo"
      - "Violación grave"
    proceso: |
      - Notificar a la otra parte con 7 días de anticipación
      - Exportar evidencia del contrato
      - Registrar evento CONTRACT_TERMINATED
```

### 6. Gobierno de Conflictos

```yaml
gobierno_conflictos:
  niveles:
    - nivel: 1
      nombre: "Automático"
      quien: "Los Domains involucrados"
      proceso: "Comparar evidencia. Mayor confianza gana."

    - nivel: 2
      nombre: "Mediación"
      quien: "Domain padre común más cercano"
      proceso: "Padre revisa evidencia de ambas partes y dictamina"

    - nivel: 3
      nombre: "Arbitraje"
      quien: "SRIE Core o Federation"
      proceso: "Revisión completa. Decisión final e inapelable."

    - nivel: 4
      nombre: "Humano"
      quien: "Humano designado"
      proceso: "Se genera reporte completo para revisión humana"
```

---

## Constitución de cada Domain

Cada Domain debe tener su propia **constitución** — un conjunto de documentos que definen su propósito y alcance:

```
SDOS/
├── charter/
│   ├── MISSION.md       → ¿Por qué existe este Domain?
│   ├── VISION.md        → ¿Qué aspira ser?
│   ├── CHARTER.md       → ¿Cuál es su alcance y límites?
│   └── CONSTITUTION.md  → Reglas específicas del Domain
```

### MISSION.md

```markdown
# Marketing SRIE — Misión

## Propósito
Gestionar campañas de marketing, captura de leads y analítica de mercado
para impulsar el crecimiento de la organización.

## Alcance
- Campañas digitales y tradicionales
- Captura y calificación de leads
- Analítica de mercado y competencia
- Branding y posicionamiento

## Límites
- NO gestiona ventas (eso es Sales SRIE)
- NO gestiona producto (eso es Product SRIE)
- NO gestiona finanzas (eso es Finance SRIE)
```

### CHARTER.md

```markdown
# Marketing SRIE — Charter

## Autoridad
- Puede crear capabilities de marketing sin aprobación externa
- Puede firmar contratos con Sales y Product automáticamente
- No puede crear capabilities financieras

## Obligaciones
- Mantener memoria actualizada de todas las campañas
- Publicar eventos de leads capturados
- Reportar indicadores al padre trimestralmente
```

---

## Ciclo de revisión de gobernanza

```yaml
revision_gobernanza:
  frecuencia: "Anual"
  quien: "SRIE Core + Federation"
  proceso:
    1. "Auditar todos los Domains activos"
    2. "Verificar cumplimiento de charters"
    3. "Verificar cadena de firmas de identidad"
    4. "Revisar conflictos del período"
    5. "Actualizar reglas de gobernanza si es necesario"
    6. "Publicar GOVERNANCE_REPORT.md"
```

---

## Sanciones

```yaml
sanciones:
  - violacion: "Modificar registry sin autorización"
    sancion: "SUSPENDER Domain por 7 días"

  - violacion: "Crear Domain duplicado"
    sancion: "ELIMINAR Domain duplicado + advertencia"

  - violacion: "Incumplir contrato reiteradamente"
    sancion: "REVOCAR capacidad de firmar contratos por 30 días"

  - violacion: "Violación grave de identidad (suplantación)"
    sancion: "REVOCAR identidad permanentemente"

  - violacion: "No mantener memoria mínima"
    sancion: "BAJAR nivel de madurez en 1"
```
