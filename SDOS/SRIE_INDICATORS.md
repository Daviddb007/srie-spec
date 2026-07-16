# SRIE INDICATORS — Modelo de Madurez, Métricas e Indicadores

---

## El modelo de madurez SRIE

SRIE define 6 niveles de madurez (L0 a L5) adaptados del modelo CMMI (Capability Maturity Model Integration). Cada nivel representa un estado cualitativamente distinto en la capacidad del proyecto para ser gobernado, medido y evolucionado por el sistema.

### Escala de niveles

```
L0 ●─────────────── Descubierto
   │
L1 ●─────────────── Documentado
   │
L2 ●─────────────── Reproducible
   │
L3 ●─────────────── Modular
   │
L4 ●─────────────── Gobernado
   │
L5 ●─────────────── Autoevolutivo
```

### Definición de cada nivel

| Nivel | Nombre | Significado |
|-------|--------|-------------|
| **L0** | Descubierto | El proyecto ha sido escaneado por Discovery Engine. Existe PROJECT_STATE.md. No hay más documentación. |
| **L1** | Documentado | El proyecto tiene README, Architecture, Design y Memory básicos. Las decisiones recientes tienen ADR. |
| **L2** | Reproducible | El proyecto se construye, prueba y despliega de forma reproducible. Existe CI/CD, tests, Docker. |
| **L3** | Modular | El proyecto está organizado en capacidades con contratos explícitos. Las capacidades son intercambiables. |
| **L4** | Gobernado | El proyecto cumple todos los indicadores de gobierno. Las decisiones requieren evidencia. El ciclo SDOS se ejecuta completo. |
| **L5** | Autoevolutivo | El sistema es capaz de proponer y ejecutar mejoras sobre sí mismo. Knowledge Engine alimenta decisiones futuras. |

### Regla de avance

Un proyecto no puede avanzar al siguiente nivel sin cumplir **todos** los indicadores del nivel actual. No existen avances parciales entre niveles.

---

## Los 8 dominios de indicadores

Cada dominio agrupa métricas relacionadas. La puntuación de cada dominio es el promedio ponderado de sus métricas.

### 1. Gobernanza

Evalúa la existencia y calidad de la documentación de gobierno del proyecto.

| # | Indicador | Peso | L0 | L1 | L2 | L3 | L4 | L5 |
|---|-----------|------|----|----|----|----|----|----|
| 1 | Existe README.md | 10% | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| 2 | Existe ARCHITECTURE.md | 10% | - | ✓ | ✓ | ✓ | ✓ | ✓ |
| 3 | Existe DESIGN.md | 10% | - | ✓ | ✓ | ✓ | ✓ | ✓ |
| 4 | Existe MEMORY.md o memory/ | 10% | - | ✓ | ✓ | ✓ | ✓ | ✓ |
| 5 | Existe COMMANDS.md | 10% | - | ✓ | ✓ | ✓ | ✓ | ✓ |
| 6 | Existe ROADMAP.md | 10% | - | - | ✓ | ✓ | ✓ | ✓ |
| 7 | Existe CHANGELOG.md | 10% | - | - | ✓ | ✓ | ✓ | ✓ |
| 8 | ADR para decisiones recientes | 10% | - | - | ✓ | ✓ | ✓ | ✓ |
| 9 | Los documentos están actualizados (<30 días) | 10% | - | - | - | ✓ | ✓ | ✓ |
| 10 | Los documentos son coherentes entre sí | 10% | - | - | - | - | ✓ | ✓ |

**Fórmula:** `Gobernanza = Σ(indicador × peso)`

---

### 2. Desarrollo

Evalúa la salud de las prácticas de desarrollo.

| # | Indicador | Peso | L0 | L1 | L2 | L3 | L4 | L5 |
|---|-----------|------|----|----|----|----|----|----|
| 1 | El proyecto compila/build sin errores | 15% | - | ✓ | ✓ | ✓ | ✓ | ✓ |
| 2 | Cobertura de tests ≥ 70% | 15% | - | - | ✓ | ✓ | ✓ | ✓ |
| 3 | Lint/pasan sin errores | 10% | - | - | ✓ | ✓ | ✓ | ✓ |
| 4 | No hay dependencias con vulnerabilidades conocidas | 10% | - | - | - | ✓ | ✓ | ✓ |
| 5 | Las dependencias están actualizadas | 10% | - | - | - | ✓ | ✓ | ✓ |
| 6 | Existe script de build reproducible | 10% | - | ✓ | ✓ | ✓ | ✓ | ✓ |
| 7 | Existe script de test ejecutable | 10% | - | ✓ | ✓ | ✓ | ✓ | ✓ |
| 8 | El proyecto sigue un estándar de formato de código | 10% | - | - | ✓ | ✓ | ✓ | ✓ |
| 9 | No hay código muerto o comentado | 5% | - | - | - | ✓ | ✓ | ✓ |
| 10 | La deuda técnica está documentada | 5% | - | - | - | - | ✓ | ✓ |

**Fórmula:** `Desarrollo = Σ(indicador × peso)`

---

### 3. Arquitectura

Evalúa la calidad y documentación de la arquitectura del proyecto.

| # | Indicador | Peso | L0 | L1 | L2 | L3 | L4 | L5 |
|---|-----------|------|----|----|----|----|----|----|
| 1 | La arquitectura está documentada | 15% | - | ✓ | ✓ | ✓ | ✓ | ✓ |
| 2 | Existe diagrama de arquitectura | 10% | - | ✓ | ✓ | ✓ | ✓ | ✓ |
| 3 | Hay separación clara de capas | 15% | - | - | ✓ | ✓ | ✓ | ✓ |
| 4 | No hay acoplamiento circular entre módulos | 15% | - | - | - | ✓ | ✓ | ✓ |
| 5 | Los contratos entre módulos están definidos | 15% | - | - | - | ✓ | ✓ | ✓ |
| 6 | El proyecto sigue un patrón arquitectónico definido | 10% | - | - | ✓ | ✓ | ✓ | ✓ |
| 7 | La deuda arquitectónica está documentada | 10% | - | - | - | - | ✓ | ✓ |
| 8 | Las decisiones arquitectónicas tienen ADR | 10% | - | - | - | ✓ | ✓ | ✓ |

**Fórmula:** `Arquitectura = Σ(indicador × peso)`

---

### 4. Memoria

Evalúa la calidad y cobertura del sistema de memoria del proyecto.

| # | Indicador | Peso | L0 | L1 | L2 | L3 | L4 | L5 |
|---|-----------|------|----|----|----|----|----|----|
| 1 | Existe memoria del proyecto (archivos) | 20% | - | ✓ | ✓ | ✓ | ✓ | ✓ |
| 2 | La memoria está actualizada | 15% | - | - | ✓ | ✓ | ✓ | ✓ |
| 3 | Existen ADR para decisiones de ingeniería | 15% | - | - | ✓ | ✓ | ✓ | ✓ |
| 4 | Las decisiones tienen contexto documentado | 10% | - | - | - | ✓ | ✓ | ✓ |
| 5 | La memoria es consultable por IA (formato estructurado) | 15% | - | - | - | ✓ | ✓ | ✓ |
| 6 | Las alternativas consideradas están documentadas | 10% | - | - | - | - | ✓ | ✓ |
| 7 | Existe un índice de memoria | 10% | - | - | - | ✓ | ✓ | ✓ |
| 8 | La memoria incluye decisiones descartadas | 5% | - | - | - | - | - | ✓ |

**Fórmula:** `Memoria = Σ(indicador × peso)`

---

### 5. Seguridad

Evalúa las prácticas de seguridad del proyecto.

| # | Indicador | Peso | L0 | L1 | L2 | L3 | L4 | L5 |
|---|-----------|------|----|----|----|----|----|----|
| 1 | No hay secretos hardcodeados | 20% | - | ✓ | ✓ | ✓ | ✓ | ✓ |
| 2 | Las variables de entorno están documentadas | 15% | - | ✓ | ✓ | ✓ | ✓ | ✓ |
| 3 | No hay dependencias con CVSs conocidas | 15% | - | - | ✓ | ✓ | ✓ | ✓ |
| 4 | Existe análisis de seguridad en CI/CD | 10% | - | - | - | ✓ | ✓ | ✓ |
| 5 | Headers de seguridad configurados | 10% | - | - | - | ✓ | ✓ | ✓ |
| 6 | Autenticación y autorización implementadas | 10% | - | - | - | ✓ | ✓ | ✓ |
| 7 | Validación de entrada en todas las APIs | 10% | - | - | ✓ | ✓ | ✓ | ✓ |
| 8 | Existe plan de respuesta a incidentes | 5% | - | - | - | - | ✓ | ✓ |
| 9 | Auditoría de seguridad periódica | 5% | - | - | - | - | - | ✓ |

**Fórmula:** `Seguridad = Σ(indicador × peso)`

---

### 6. Operación

Evalúa la madurez operativa del proyecto.

| # | Indicador | Peso | L0 | L1 | L2 | L3 | L4 | L5 |
|---|-----------|------|----|----|----|----|----|----|
| 1 | Existe logging estructurado | 15% | - | - | ✓ | ✓ | ✓ | ✓ |
| 2 | Existen health checks | 15% | - | - | ✓ | ✓ | ✓ | ✓ |
| 3 | Existe monitoreo de errores | 15% | - | - | - | ✓ | ✓ | ✓ |
| 4 | Existen backups automáticos | 10% | - | - | - | ✓ | ✓ | ✓ |
| 5 | Existe plan de restore | 10% | - | - | - | - | ✓ | ✓ |
| 6 | El deploy es automatizado (CI/CD) | 15% | - | ✓ | ✓ | ✓ | ✓ | ✓ |
| 7 | Existe estrategia de rollback | 10% | - | - | ✓ | ✓ | ✓ | ✓ |
| 8 | Hay alertas configuradas | 10% | - | - | - | ✓ | ✓ | ✓ |

**Fórmula:** `Operación = Σ(indicador × peso)`

---

### 7. UX

Evalúa la madurez de la experiencia de usuario.

| # | Indicador | Peso | L0 | L1 | L2 | L3 | L4 | L5 |
|---|-----------|------|----|----|----|----|----|----|
| 1 | El sitio/app es responsive | 20% | - | - | ✓ | ✓ | ✓ | ✓ |
| 2 | Cumple estándares de accesibilidad (WCAG) | 15% | - | - | - | ✓ | ✓ | ✓ |
| 3 | Tiene SEO básico (meta tags, sitemap) | 15% | - | - | ✓ | ✓ | ✓ | ✓ |
| 4 | Performance aceptable (Lighthouse ≥ 80) | 15% | - | - | ✓ | ✓ | ✓ | ✓ |
| 5 | Manejo de errores visible para el usuario | 15% | - | ✓ | ✓ | ✓ | ✓ | ✓ |
| 6 | Existe diseño de componentes reutilizables | 10% | - | - | ✓ | ✓ | ✓ | ✓ |
| 7 | Pruebas de usabilidad realizadas | 10% | - | - | - | - | ✓ | ✓ |

**Fórmula:** `UX = Σ(indicador × peso)`

---

### 8. IA

Evalúa la madurez del proyecto desde la perspectiva de la inteligencia artificial que lo gobierna.

| # | Indicador | Peso | L0 | L1 | L2 | L3 | L4 | L5 |
|---|-----------|------|----|----|----|----|----|----|
| 1 | Existe prompt coverage (todos los archivos relevantes son accesibles) | 15% | - | - | ✓ | ✓ | ✓ | ✓ |
| 2 | Las capabilities están definidas y catalogadas | 15% | - | - | ✓ | ✓ | ✓ | ✓ |
| 3 | El knowledge base tiene patrones documentados | 15% | - | - | - | ✓ | ✓ | ✓ |
| 4 | No hay loops de repetición en el ciclo SDOS | 10% | - | - | - | ✓ | ✓ | ✓ |
| 5 | Riesgo de alucinación bajo (contratos explícitos) | 15% | - | - | ✓ | ✓ | ✓ | ✓ |
| 6 | La memoria del proyecto es consultable por IA | 10% | - | - | ✓ | ✓ | ✓ | ✓ |
| 7 | El contexto necesario para decisiones está cubierto | 10% | - | - | - | ✓ | ✓ | ✓ |
| 8 | El sistema aprende de iteraciones anteriores | 10% | - | - | - | - | ✓ | ✓ |

**Fórmula:** `IA = Σ(indicador × peso)`

---

## SRIE_SCORE: La métrica compuesta

### Fórmula general

```
SRIE_SCORE = (Gobernanza × 0.20)
           + (Desarrollo × 0.20)
           + (Arquitectura × 0.15)
           + (Memoria × 0.15)
           + (Seguridad × 0.10)
           + (Operación × 0.10)
           + (UX × 0.05)
           + (IA × 0.05)
```

**Rango:** 0.0 — 100.0

### Correspondencia con niveles

| SRIE_SCORE | Nivel | Significado |
|------------|-------|-------------|
| 0.0 | L0 | Proyecto descubierto |
| 0.1 — 25.0 | L1 | Proyecto documentado |
| 25.1 — 50.0 | L2 | Proyecto reproducible |
| 50.1 — 70.0 | L3 | Proyecto modular |
| 70.1 — 90.0 | L4 | Proyecto gobernado |
| 90.1 — 100.0 | L5 | Proyecto autoevolutivo |

### Reglas de interpretación

1. **Un dominio en 0 anula el avance** — Si cualquier dominio puntúa 0, el proyecto no puede superar L1, independientemente del SRIE_SCORE.
2. **El nivel no es negociable** — No se puede declarar un nivel. Se calcula desde los indicadores.
3. **Progresión forzada** — Un proyecto en L2 debe cumplir todos los indicadores L2 de todos los dominios para alcanzar L3.
4. **Regresión posible** — Si un indicador deja de cumplirse (ej. se borra un archivo), el nivel puede descender.

---

## Formato del reporte de indicadores

El Indicators Engine genera `SDOS/SRIE_REPORT.md` con esta estructura:

```yaml
srie_score: 74.3
nivel_madurez: L4
fecha: "2026-07-15T10:00:00Z"
proyecto: mi-proyecto
dominios:
  gobernanza:
    puntuacion: 95
    nivel_dominio: L4
    indicadores:
      - nombre: Existe README.md
        cumplido: true
        peso: 10
      - nombre: Existe ARCHITECTURE.md
        cumplido: true
        peso: 10
  desarrollo:
    puntuacion: 82
    nivel_dominio: L3
    indicadores:
      - nombre: Cobertura de tests ≥ 70%
        cumplido: false
        peso: 15
        detalle: "Cobertura actual: 54%"
capacidades_requeridas_proximo_nivel:
  - Alcanzar cobertura de tests ≥ 70%
  - Documentar deuda técnica
```

---

## Archivo de configuración de indicadores

El sistema puede leer `SDOS/SRIE_INDICATORS_CONFIG.yaml` para personalizar pesos o agregar indicadores específicos del proyecto:

```yaml
personalizacion:
  dominios_adicionales: []
  pesos_personalizados:
    gobernanza: 0.25        # Peso mayor al estándar
    ia: 0.10                # Peso mayor al estándar
  indicadores_propios:
    - dominio: gobernanza
      nombre: Existe CONTRIBUTING.md
      peso: 5
      niveles: [L1, L2, L3, L4, L5]
```

Si no existe este archivo, se usan los pesos y métricas por defecto definidos en este documento.
