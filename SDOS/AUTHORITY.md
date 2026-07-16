# AUTHORITY — Niveles de Autoridad en el Ecosistema SRIE

---

## ¿Qué es este documento?

AUTHORITY.md define **los niveles de autoridad** dentro del ecosistema SRIE. No todos los nodos tienen los mismos poderes. La autoridad determina qué puede crear, modificar, aprobar o eliminar cada nivel.

---

## La jerarquía de autoridad

```
CORE ──────────── Autoridad raíz del sistema
  │
  ▼
FEDERATION ────── Autoridad sobre una red de Domains
  │
  ▼
ORGANIZATION ──── Autoridad sobre una organización
  │
  ▼
DOMAIN ────────── Autoridad sobre un dominio específico
  │
  ▼
CAPABILITY ────── Autoridad mínima (solo su propia existencia)
```

**La autoridad nunca fluye hacia arriba.** Un Domain no puede gobernar a su Organization. Una Organization no puede gobernar a su Federation. Un Core no es gobernado por nadie (es la raíz de confianza).

---

## Nivel 0: CORE

El kernel del sistema SRIE. No es un Domain. Es la **raíz de confianza** de todo el ecosistema.

```yaml
nivel: CORE
identidad: "SRIE-CORE-001"

poderes:
  - "Definir la ontología del sistema"
  - "Definir el contrato universal de Engines"
  - "Modificar el kernel SDOS"
  - "Crear y destruir Federations"
  - "Arbitrar conflictos entre Federations"
  - "Revocar cualquier identidad del sistema"
  - "Definir las reglas de gobernanza global"

restricciones:
  - "No opera sobre ningún dominio de negocio"
  - "No tiene capabilities propias"
  - "No tiene memoria de negocio"

relacion_con_domains:
  - "No es padre de ningún Domain"
  - "Es la autoridad máxima a la que cualquier Domain puede apelar"
```

---

## Nivel 1: FEDERATION

Una red de Domains que cooperan bajo reglas comunes.

```yaml
nivel: FEDERATION
identidad: "NET-<ORG>-<NUM>"

poderes:
  - "Crear y destruir Organizations dentro de la red"
  - "Aprobar Organizations para unirse a la red"
  - "Definir reglas de la red (NETWORK.md)"
  - "Gestionar el Federation Registry"
  - "Mediar conflictos entre Organizations"
  - "Suspender Organizations"

restricciones:
  - "No puede modificar la ontología"
  - "No puede modificar el kernel SDOS"
  - "No opera sobre dominios de negocio"

requisitos:
  nivel_minimo: "L4"
  necesita_aprobacion: "CORE"
```

---

## Nivel 2: ORGANIZATION

Una organización (empresa, unidad de negocio, cliente) que opera dentro de la federación.

```yaml
nivel: ORGANIZATION
identidad: "ORG-<NOMBRE>-<NUM>"

poderes:
  - "Crear y destruir Domains dentro de la organización"
  - "Aprobar charters de Domains"
  - "Gestionar el registro de Domains de la organización"
  - "Mediar conflictos entre Domains de la organización"
  - "Firmar contratos en nombre de la organización"

restricciones:
  - "No puede crear Domains fuera de su organización"
  - "No puede modificar reglas de la federación"
  - "No puede acceder a memoria de Domains sin contrato"

requisitos:
  nivel_minimo: "L3"
  necesita_aprobacion: "FEDERATION"
```

---

## Nivel 3: DOMAIN

Un dominio de negocio específico. Es la unidad operativa del sistema.

```yaml
nivel: DOMAIN
identidad: "DOM-<ORG>-<AREA>-<NUM>"

poderes:
  - "Crear y destruir capabilities dentro de su dominio"
  - "Gestionar su propia memoria local"
  - "Firmar contratos con otros Domains"
  - "Publicar eventos al BUS"
  - "Definir su charter y misión"

restricciones:
  - "No puede crear otros Domains (solo sub-Domains si el padre lo autoriza)"
  - "No puede modificar el registry federado"
  - "No puede modificar la ontología"
  - "No puede acceder a memoria de otros Domains sin contrato"
  - "No puede crear capabilities fuera de su dominio"

requisitos:
  nivel_minimo: "L1"
  necesita_aprobacion: "ORGANIZATION"
```

---

## Nivel 4: CAPABILITY

La unidad funcional más pequeña del sistema. No tiene autoridad sobre nada más que su propia existencia.

```yaml
nivel: CAPABILITY
identidad: "CAP-<DOMAIN>-<NOMBRE>-<NUM>"

poderes:
  - "Existir dentro de su Domain propietario"
  - "Ser consumida por otros Domains mediante contrato"
  - "Registrar su propia evidencia"

restricciones:
  - "No puede crear nada fuera de sí misma"
  - "No puede firmar contratos (lo hace su Domain propietario)"
  - "No puede modificar su Domain propietario"

requisitos:
  nivel_minimo: "L0"
  necesita_aprobacion: "DOMAIN"
```

---

## Matriz de autoridad

| Acción | CORE | FEDERATION | ORGANIZATION | DOMAIN | CAPABILITY |
|--------|------|------------|--------------|--------|------------|
| Definir ontología | ✓ | - | - | - | - |
| Modificar kernel SDOS | ✓ | - | - | - | - |
| Crear Federation | ✓ | - | - | - | - |
| Crear Organization | ✓ | ✓ | - | - | - |
| Crear Domain | - | - | ✓ | * | - |
| Crear Capability | - | - | - | ✓ | - |
| Firmar contrato | ✓ | ✓ | ✓ | ✓ | - |
| Arbitrar conflicto | ✓ | ✓ | ✓ | - | - |
| Suspender Domain | - | - | ✓ | - | - |
| Revocar identidad | ✓ | ✓ | ✓ | - | - |
| Acceder a registry global | ✓ | ✓ | ✓ | ✓ | - |
| Modificar registry local | - | - | - | ✓ | - |
| Leer memoria de otro Domain | * | * | * | * | - |

(* = solo con contrato explícito)

---

## Delegación de autoridad

Un nivel superior puede delegar autoridad a un nivel inferior:

```yaml
delegacion:
  ejemplo:
    nivel: "ORGANIZATION"
    delega_a: "DOMAIN"
    accion: "Crear sub-Domains"
    condiciones:
      - "Solo dentro del alcance definido en el charter"
      - "Máximo 3 sub-Domains sin aprobación adicional"
      - "Cada sub-Domain debe reportar al padre"

  reglas:
    - "La delegación es revocable en cualquier momento"
    - "La delegación debe registrarse en el Governance Registry"
    - "El delegante sigue siendo responsable de las acciones del delegado"
```

---

## Cadena de mando para resolución de conflictos

```
1. ¿El conflicto es entre dos capabilities del mismo Domain?
   → Resuelve: DOMAIN (dueño de ambas)

2. ¿El conflicto es entre dos Domains de la misma Organization?
   → Resuelve: ORGANIZATION (padre común)

3. ¿El conflicto es entre dos Organizations de la misma Federation?
   → Resuelve: FEDERATION

4. ¿El conflicto es entre dos Federations?
   → Resuelve: CORE

5. ¿El conflicto involucra al CORE?
   → Resuelve: Revisión humana (no hay autoridad superior)
```

---

## Pérdida de autoridad

```yaml
perdida_autoridad:
  causas:
    - "Nivel de madurez cae por debajo del mínimo requerido"
    - "Violación de gobernanza reiterada"
    - "Confianza promedio por debajo de 0.50 por 30 días"
    - "Decisión del nivel superior"

  efectos:
    - "El Domain degradado pierde capacidad de firmar contratos"
    - "Sus capabilities existentes siguen activas"
    - "Su memoria se preserva"
    - "Puede recuperar autoridad al cumplir los requisitos nuevamente"

  recuperacion:
    - "Ejecutar Repair Engine para corregir causas"
    - "Demostrar cumplimiento durante 30 días"
    - "Solicitar revisión al nivel superior"
```
