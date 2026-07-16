# SELF GOVERNANCE — Meta-Gobernanza del Sistema SRIE

---

## ¿Qué es este documento?

SELF_GOVERNANCE.md define **cómo SRIE se gobierna a sí mismo**. Hasta ahora definimos la gobernanza de los proyectos que SRIE gobierna. Pero SRIE también necesita reglas para su propia evolución: ¿quién modifica la Foundation? ¿cómo se versiona? ¿cómo se migra? ¿cómo se deprecia?

Mientras que GOVERNANCE.md gobierna los Domains de la federación, SELF_GOVERNANCE.md gobierna al sistema SRIE mismo.

---

## Principios de auto-gobernanza

1. **El sistema puede modificarse a sí mismo**, pero bajo reglas explícitas.
2. **Toda modificación al sistema deja evidencia.** No hay cambios silenciosos.
3. **La Foundation es semánticamente inmutable.** Los conceptos no cambian de significado.
4. **Las modificaciones tienen período de transición.** Nada cambia de inmediato.
5. **El sistema no puede violar sus propias reglas.** La gobernanza aplica al gobernante.

---

## Niveles de modificación

```yaml
niveles_modificacion:
  - nivel: "PATCH"
    descripcion: "Corrección, aclaración, error"
    aprueba: "El Engine responsable"
    periodo_transicion: "Inmediato"
    requiere_aprobacion: false

  - nivel: "MINOR"
    descripcion: "Adición compatible hacia atrás"
    aprueba: "SRIE Core"
    periodo_transicion: "1 ciclo SDOS"
    requiere_aprobacion: true

  - nivel: "MAJOR"
    descripcion: "Cambio incompatible en la Foundation"
    aprueba: "Votación de Domains activos (≥67%)"
    periodo_transicion: "3 ciclos SDOS"
    requiere_aprobacion: true
```

---

## Lo que SRIE puede modificar

```yaml
puede_modificar:
  - "Sus propios Engines (mejorarlos, versionarlos)"
  - "Sus Capabilities (crear, actualizar, deprecar)"
  - "Su Registry (registrar nuevas entidades)"
  - "Su Knowledge Base (agregar patrones, casos)"
  - "Su configuración (recursos, políticas)"

no_puede_modificar:
  - "Los principios fundacionales de SRIE_ENGINE.md"
  - "El contrato universal de ENGINE_CONTRACT.md"
  - "La ontología base (sin proceso MAJOR)"
  - "Su propia identidad raíz"
  - "Las reglas de gobernanza de nivel superior"
```

---

## Proceso de cambio Foundation

```yaml
foundation_change:
  proposal:
    formato: "RFC (Request for Comments)"
    contenido:
      - "¿Qué cambiar?"
      - "¿Por qué?"
      - "¿Impacto en compatibilidad?"
      - "¿Plan de migración?"
      - "¿Riesgos?"

  revision:
    - "Revisión técnica por SRIE Core (7 días)"
    - "Período de comentarios abierto (14 días)"
    - "Votación de Domains activos (si MAJOR)"

  migracion:
    - "Anunciar cambio con período de transición"
    - "Versión antigua y nueva coexisten durante transición"
    - "Registry versiona la migración"
    - "Cada Domain elige cuándo migrar"

  post_migracion:
    - "Verificar que todos los Domains migraron"
    - "Archivar versión anterior"
    - "Actualizar ECOSYSTEM_VERSION.md"
```

---

## Auto-auditoría

```yaml
self_audit:
  frecuencia: "Cada 3 meses"
  quien: "Governance Engine (modo self)"
  alcance:
    - "Verificar que todos los documentos de la Foundation existen"
    - "Verificar que ningún Engine violó su contrato"
    - "Verificar que el Registry es consistente"
    - "Verificar que la cadena de identidad es válida"
    - "Verificar que las políticas se cumplen"

  reporte: "SDOS/SELF_AUDIT_REPORT.md"

  accion_si_inconsistencia:
    - "Si es recuperable: Repair Engine"
    - "Si no es recuperable: notificar a SRIE Core + humanos"
```
