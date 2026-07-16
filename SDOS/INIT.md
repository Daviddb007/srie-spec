# INIT — Especificación Funcional del Proceso `srie init`

---

## Propósito

`INIT` es el punto de entrada del sistema SRIE. Es el primer proceso que se ejecuta cuando un proyecto aún no está gobernado por SDOS.

INIT no genera código. No construye funcionalidad. No modifica el contenido del proyecto. **INIT descubre, diagnostica y prepara el terreno** para que el ciclo SDOS pueda ejecutarse.

---

## Ámbito

### Lo que INIT HACE

- Establece la **identidad** del nodo (ID, tipo, autoridad, cadena de firmas)
- Descubre si el directorio actual es un proyecto existente o está vacío
- Escanea la estructura del proyecto (si existe)
- Genera la estructura mínima de SDOS
- Crea el archivo `PROJECT_STATE.md` con el inventario inicial
- Crea `STATE.md` con el estado inicial del ciclo
- Establece el nivel de madurez base (L0 o superior si detecta documentación)

### Lo que INIT NUNCA HACE

- No modifica archivos del proyecto existente
- No instala dependencias
- No ejecuta scripts de build, test o deploy
- No modifica configuraciones existentes
- No migra bases de datos
- No toca `.gitignore`, `README.md` ni ningún archivo del proyecto
- No genera código de implementación
- No borra archivos

---

## Diagrama de flujo

```
┌──────────────────────────────────────────────────────────┐
│                  srie init (4 niveles)                    │
└────────────────────────┬─────────────────────────────────┘
                         │
                         ▼
              ┌──────────────────────┐
              │    NIVEL 0: IDENTITY  │
              │  ¿Existe IDENTITY?    │
              └──────────┬───────────┘
                         │
           ┌─────────────┼─────────────┐
           │ NO          │ SÍ          │
           ▼             ▼             │
   ┌──────────────┐  ┌──────────────┐ │
   │ Crear        │  │ Validar      │ │
   │ IDENTITY     │  │ cadena       │ │
   └──────┬───────┘  │ de firmas    │ │
          │          └──────┬───────┘ │
          │                 │         │
          └──────┬──────────┘         │
                 ▼                    │
              ┌─────────────────────┐ │
              │  ¿Existe SDOS/ ?    │ │
              └─────────┬───────────┘ │
                        │             │
          ┌─────────────┼─────────────┘
          │ NO          │ SÍ
          ▼             ▼
   ┌──────────────┐  ┌──────────────┐
   │ Proyecto     │  │ Proyecto     │
   │ no iniciado  │  │ gobernado    │
   └──────┬───────┘  └──────┬───────┘
          │                 │
          ▼                 ▼
   ┌──────────────┐  ┌──────────────┐
   │ Discovery    │  │ Discovery    │
   │ completo     │  │ rápido       │
   └──────┬───────┘  └──────┬───────┘
          │                 │
          ▼                 ▼
   ┌──────────────┐  ┌──────────────┐
   │ Crear SDOS/  │  │ Actualizar   │
   │ + STATE.md   │  │ STATE.md     │
   └──────┬───────┘  └──────┬───────┘
          │                 │
          └──────┬──────────┘
                 ▼
          ┌──────────────┐
          │ PROJECT_     │
          │ STATE.md     │
          └──────┬───────┘
                 │
                 ▼
          ┌──────────────┐
          │  INIT listo  │
          │  Fin FASE 0  │
          └──────┬───────┘
                 │
                 ▼
          ┌──────────────┐
          │  DISCOVERY   │
          │  ENGINE      │
          └──────────────┘
```

---

## Nivel 0: Identity

Antes de cualquier discovery, INIT establece la identidad del nodo.

```
IDENTITY
  │
  ├── ¿Existe SDOS/IDENTITY.yaml?
  │     │
  │     ├── NO → Crear identidad:
  │     │         1. Generar domain_id único (DOM-<ORG>-<AREA>-<NUM>)
  │     │         2. Determinar tipo (standalone | domain)
  │     │         3. Crear cadena de firmas
  │     │         4. Si hay federación, registrar en federation registry
  │     │
  │     └── SÍ → Validar identidad:
  │               1. Verificar que el domain_id no esté revocado
  │               2. Validar cadena de firmas
  │               3. Verificar que la identidad no haya expirado
  │               4. Si expirada, solicitar renovación
  │
  ▼
Continuar a DISCOVERY (Fase 0)
```

### Salida de IDENTITY

```yaml
# SDOS/IDENTITY.yaml
identity:
  domain_id: "DOM-001"
  nombre: "SRIE Domain"
  tipo: "standalone"
  version: "1.0.0"
  estado: "ACTIVE"
  organization: null
  parent: null
  authority:
    nivel: "Domain"
    gobernado_por: null
  federation: null
  created: "2026-07-15T10:00:00Z"
  updated: "2026-07-15T10:00:00Z"
  fingerprint: "sha256:..."
```

### Reglas de IDENTITY en INIT

1. **Nunca reemplaza una identidad existente.** Si IDENTITY.yaml existe, la valida. No la regenera.
2. **Un Domain sin identidad no puede continuar.** Si no hay identidad y no se puede crear, INIT aborta.
3. **La identidad se crea antes que SDOS.** Primero el nodo sabe quién es, luego decide qué estructura crear.
4. **La identidad de un proyecto standalone es temporal.** Puede actualizarse cuando se una a una federación.

---

## Fase 0: Discovery

INIT ejecuta un discovery superficial para determinar qué existe.

### Si el proyecto NO existe (directorio vacío o nuevo)

```
Estado: PROYECTO_NUEVO

Acciones:
1. Crear directorio SDOS/
2. Crear SDOS/STATE.md con:
   - ciclo_actual: 0
   - fase_actual: INIT
   - nivel_madurez: L0
   - proyecto_descubierto: false
3. Crear SDOS/PROJECT_STATE.md con:
   - estado: NUEVO
   - lenguaje: no detectado
   - framework: no detectado
   - documentacion: ninguna
4. Mostrar mensaje al usuario:
   "Proyecto nuevo detectado. SDOS inicializado en L0.
    Ejecuta 'srie discover' para comenzar el ciclo."
```

### Si el proyecto SÍ existe (detecta archivos)

```
Estado: PROYECTO_EXISTENTE

Acciones:
1. Crear directorio SDOS/
2. Ejecutar discovery superficial (solo lectura):
   a. Detectar lenguaje(s) por extensiones de archivo
   b. Detectar framework por archivos de configuración
   c. Detectar documentación existente
   d. Detectar git (ramas, últimos commits)
   e. Detectar tests (directorios, archivos de configuración)
   f. Detectar CI/CD
   g. Detectar Docker
3. Generar SDOS/PROJECT_STATE.md con hallazgos
4. Generar SDOS/STATE.md con estado inicial
5. Mostrar mensaje al usuario:
   "Proyecto existente detectado. SDOS inicializado.
    Nivel de madurez base: [L0 | L1].
    Discovery Engine disponible para análisis completo."
```

---

## Estructura que genera INIT

```
<proyecto>/
└── SDOS/
    ├── IDENTITY.yaml              # Identidad del nodo
    ├── STATE.md                   # Estado del ciclo SDOS
    ├── PROJECT_STATE.md           # Inventario del proyecto
    └── SRIE_ENGINE.md             # Copia de la constitución
```

### SDOS/STATE.md

```yaml
# SDOS State
ciclo_actual: 0
fase_actual: INIT
fase_anterior: null
ultima_fase_completada: null
srie_score: null
nivel_madurez: L0
proyecto_descubierto: false
proyecto_tipo: "nuevo | existente"
creado_en: "2026-07-15T10:00:00Z"
ultima_actualizacion: "2026-07-15T10:00:00Z"
```

### SDOS/PROJECT_STATE.md (proyecto vacío)

```yaml
# Project State
estado: NUEVO
creado_en: "2026-07-15T10:00:00Z"
lenguaje_detectado: null
framework_detectado: null
documentacion_detectada: []
tests_detectados: false
docker_detectado: false
ci_cd_detectado: false
git_detectado: false
ramas: []
ultimo_commit: null
archivos_totales: 0
estructura_descubierta: []
```

### SDOS/PROJECT_STATE.md (proyecto existente)

```yaml
# Project State
estado: EXISTENTE
creado_en: "2026-07-15T10:00:00Z"
lenguaje_detectado: "Python"
framework_detectado: "Flask"
documentacion_detectada:
  - README.md
  - ARCHITECTURE.md (no)
  - MEMORY.md (no)
  - DESIGN.md (no)
  - ROADMAP.md (no)
tests_detectados: true
framework_tests: "pytest"
docker_detectado: true
ci_cd_detectado: false
git_detectado: true
ramas:
  - main
  - develop
ultimo_commit:
  hash: "abc123"
  mensaje: "feat: initial commit"
  autor: "usuario"
  fecha: "2026-07-14"
archivos_totales: 47
estructura_descubierta:
  - src/
  - tests/
  - docs/
  - docker/
dependencias_principales:
  - flask==3.1.0
  - sqlalchemy==2.0.0
```

---

## Detección de lenguajes y frameworks

INIT detecta la tecnología del proyecto mediante archivos característicos:

| Lenguaje | Detectado por |
|----------|---------------|
| Python | `*.py`, `pyproject.toml`, `requirements.txt`, `setup.py`, `Pipfile` |
| JavaScript | `*.js`, `package.json`, `node_modules/` |
| TypeScript | `*.ts`, `tsconfig.json` |
| Go | `*.go`, `go.mod` |
| Rust | `*.rs`, `Cargo.toml` |
| Java | `*.java`, `pom.xml`, `build.gradle` |
| Ruby | `*.rb`, `Gemfile` |
| PHP | `*.php`, `composer.json` |
| C# | `*.cs`, `*.csproj`, `*.sln` |
| Swift | `*.swift`, `Package.swift` |

| Framework | Detectado por |
|-----------|---------------|
| Flask | `flask` en dependencias |
| Django | `django` en dependencias, `manage.py` |
| React | `react` en package.json |
| Next.js | `next.config.js` |
| Vue | `vue` en package.json |
| Angular | `angular.json` |
| Express | `express` en package.json |
| FastAPI | `fastapi` en dependencias |
| Rails | `Gemfile` con `rails` |
| Spring | `pom.xml` con `spring-boot` |

---

## Validación post-ejecución

Después de ejecutar INIT, se debe validar que:

1. `SDOS/` existe en la raíz del proyecto
2. `SDOS/STATE.md` existe y tiene todos los campos requeridos
3. `SDOS/PROJECT_STATE.md` existe con la información del discovery
4. Ningún archivo fuera de `SDOS/` fue creado, modificado o eliminado
5. Si el proyecto es nuevo, PROJECT_STATE.md refleja estado NUEVO
6. Si el proyecto existe, PROJECT_STATE.md contiene detección de lenguaje y framework

---

## Códigos de salida

| Código | Significado |
|--------|-------------|
| 0 | INIT completado exitosamente |
| 1 | Error: no se puede crear SDOS/ (permisos) |
| 2 | Error: proyecto corrupto o ilegible |
| 3 | Advertencia: INIT ya se había ejecutado (SDOS/ existe) |

---

## Transición al ciclo SDOS

Una vez INIT completa su ejecución, el sistema debe:

```
Si STATE.proyecto_tipo == "nuevo":
    → Mostrar mensaje: "Proyecto listo. SRIE inicializado en L0."
    → Esperar instrucciones del usuario

Si STATE.proyecto_tipo == "existente":
    → STATE.fase_actual = "DISCOVERY"
    → Invocar Discovery Engine
    → Iniciar ciclo SDOS completo
```

---

## Consideraciones de seguridad

1. INIT nunca debe leer archivos binarios o ejecutables (solo extensiones de código y texto)
2. INIT nunca debe leer archivos de secretos (`.env`, `*.key`, `*.pem`, `credentials.*`)
3. INIT nunca debe enviar información del proyecto a servicios externos
4. INIT nunca debe ejecutar código del proyecto descubierto
5. INIT debe respetar `.gitignore` para determinar qué archivos escanear
