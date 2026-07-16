# Project State — Stonelytics Studio
ultima_actualizacion: "2026-07-15T14:00:00Z"
modo: "full"
discovery_id: "disc_stonelytics_001"

proyecto:
  nombre: "SRIE_Engine / Stonelytics Studio"
  estado: "ANALYZED"
  nivel_madurez: "L1"

lenguajes:
  - nombre: "Python"
    version: "3.14"
    confianza: 0.99

frameworks:
  - nombre: "Flask"
    version: "3.1.1"
    fuente: "requirements.txt"
    confianza: 0.97

arquitectura:
  patrones:
    - "Service Layer"
    - "Blueprint-based modular routing"
    - "Application Factory"
  confianza: 0.85
  nota: "Service Layer detectado. No se confirma DDD completo. Hay separación routes→services→models."

documentacion:
  - archivo: "README.md"
    existe: false
    dominio: "business"
    criticidad: "CRITICAL"
  - archivo: "ARCHITECTURE.md"
    existe: false
    dominio: "technical"
    criticidad: "CRITICAL"
  - archivo: "CHANGELOG.md"
    existe: false
    dominio: "documentation"
    criticidad: "MEDIUM"
  - archivo: "DESIGN.md"
    existe: false
    dominio: "technical"
    criticidad: "HIGH"
  - archivo: "SDOS/"
    existe: true
    dominio: "governance"
    documentos: 67
    criticidad: "HIGH"
  - archivo: "docs/"
    existe: true
    dominio: "documentation"
    documentos: 2
    criticidad: "MEDIUM"

git:
  detectado: true
  ramas:
    - "master"
  commits_totales: 5
  autores: ["David DB"]
  ultimo_commit:
    hash: "1e88326"
    mensaje: "fix: handle_login_form returns None on invalid credentials"
    fecha: "2026-07-08"

tests:
  detectado: true
  framework: "pytest"
  config: "ninguno (pytest por defecto)"
  archivos_test: 10
  funciones_test: 92
  cobertura: "No detectada (sin herramienta de cobertura)"

docker:
  detectado: false

ci_cd:
  detectado: false

variables_entorno:
  archivos:
    - ".env.example"
  variables_detectadas:
    - "DATABASE_URL"
    - "OPENAI_API_KEY"

seguridad:
  secretos_hardcodeados:
    detectado: true
    archivos:
      - "deploy.bat"
      - "config.py"
      - "start.bat"
    severidad: "HIGH"

base_de_datos:
  motor: "PostgreSQL (producción) / SQLite (desarrollo)"
  modelos: 21
  migraciones: "No detectadas (db.create_all())"

capabilities:
  total: 12
  lista:
    - nombre: "Landing"
      path: "modules/landing/"
      estado: "ACTIVE"
    - nombre: "Contact"
      path: "modules/contact/"
      estado: "ACTIVE"
    - nombre: "Demo Start"
      path: "modules/demos/start.py"
      estado: "ACTIVE"
    - nombre: "Demo Growth"
      path: "modules/demos/growth.py"
      estado: "ACTIVE"
    - nombre: "Demo Intelligence"
      path: "modules/demos/intelligence.py"
      estado: "ACTIVE"
    - nombre: "Admin Panel"
      path: "modules/admin/"
      estado: "ACTIVE"
      submódulos: 7
    - nombre: "Cotizador"
      path: "modules/cotizador/"
      estado: "ACTIVE"
    - nombre: "Studio (SRIE Core)"
      path: "modules/studio/"
      estado: "ACTIVE"
    - nombre: "Portal Cliente"
      path: "modules/portal/"
      estado: "ACTIVE"
    - nombre: "Marketplace"
      path: "modules/marketplace/"
      estado: "ACTIVE"
    - nombre: "Teams"
      path: "modules/teams/"
      estado: "ACTIVE"
    - nombre: "Analytics"
      path: "modules/analytics/"
      estado: "ACTIVE"

archivos_totales: 312
archivos_py: 81
archivos_html: 46
archivos_css: 8
archivos_js: 8
archivos_md: 71

endpoints_api: 74
blueprints: 18
servicios: 9
modelos: 21

memoria:
  sdos_documentos: 67
  industry_packs: 4
  ai_prompts: "10 capas de entrevista"
  solution_mapper: "12 reglas"
