# Knowledge Graph — Stonelytics Studio
version: 1
ultima_actualizacion: "2026-07-15T14:00:00Z"

nodos:
  - id: "PROJECT"
    tipo: "raiz"
    estado: "SYNCED"
    archivos: ["SDOS/PROJECT.yaml", "SDOS/PROJECT_STATE.md"]

  - id: "BUSINESS"
    tipo: "dominio"
    estado: "DESYNCED"
    archivos: []
    nota: "No hay README.md. El propósito del proyecto no está documentado explícitamente."

  - id: "ARCHITECTURE"
    tipo: "dominio"
    estado: "DESYNCED"
    archivos: []
    nota: "No hay ARCHITECTURE.md. La arquitectura se infiere del código."

  - id: "CAPABILITIES"
    tipo: "dominio"
    estado: "SYNCED"
    archivos: ["modules/"]
    hijos:
      - "CAP:landing"
      - "CAP:contact"
      - "CAP:demo_start"
      - "CAP:demo_growth"
      - "CAP:demo_intelligence"
      - "CAP:admin"
      - "CAP:cotizador"
      - "CAP:studio"
      - "CAP:portal"
      - "CAP:marketplace"
      - "CAP:teams"
      - "CAP:analytics"

  - id: "CAP:studio"
    tipo: "capability"
    estado: "SYNCED"
    archivos: ["modules/studio/"]
    depende_de: ["CAP:admin"]
    nota: "Corazón del sistema. Contiene AI prompts, twin engine, SRIE mapper, code generator."

  - id: "CAP:admin"
    tipo: "capability"
    estado: "SYNCED"
    archivos: ["modules/admin/"]
    hijos:
      - "CAP:admin_auth"
      - "CAP:admin_clients"
      - "CAP:admin_projects"
      - "CAP:admin_quotations"
      - "CAP:admin_catalog"

  - id: "SECURITY"
    tipo: "dominio"
    estado: "DESYNCED"
    archivos: [".env.example"]
    nota: "Secretos hardcodeados en deploy.bat y config.py. CSRF exempt en studio blueprint."

  - id: "INFRASTRUCTURE"
    tipo: "dominio"
    estado: "PENDING"
    archivos: ["deploy.bat", "start.bat"]
    nota: "Sin Docker, sin CI/CD. Deploy manual por SSH."

  - id: "DOCUMENTATION"
    tipo: "dominio"
    estado: "PENDING"
    archivos: ["SDOS/", "docs/"]
    nota: "67 documentos SDOS de especificación, pero sin README.md ni ARCHITECTURE.md."

  - id: "MEMORY"
    tipo: "dominio"
    estado: "SYNCED"
    archivos: ["SDOS/"]
    nota: "La especificación SDOS funciona como memoria del sistema."

  - id: "INDICATORS"
    tipo: "dominio"
    estado: "PENDING"
    archivos: []
    nota: "Requiere ejecución del Indicators Engine."

  - id: "DEPLOYMENT"
    tipo: "dominio"
    estado: "PENDING"
    archivos: ["deploy.bat"]
    nota: "Deploy manual sin automatización."

  - id: "KNOWLEDGE"
    tipo: "dominio"
    estado: "SYNCED"
    archivos: ["modules/studio/industry_packs.py", "modules/studio/ai_prompts.py", "modules/studio/srie_mapper.py"]
    nota: "Industry packs, AI prompts y solution mapper son conocimiento estructurado."

  - id: "TESTS"
    tipo: "dominio"
    estado: "SYNCED"
    archivos: ["tests/"]
    nota: "92 tests, pytest, sin cobertura configurada."

  - id: "DESIGN"
    tipo: "dominio"
    estado: "DESYNCED"
    archivos: ["docs/superpowers/specs/"]
    nota: "Solo existe design doc del sprint 1 de SDOS."

relaciones:
  - origen: "PROJECT" destino: "BUSINESS" tipo: "contiene" peso: 10
  - origen: "PROJECT" destino: "ARCHITECTURE" tipo: "contiene" peso: 10
  - origen: "PROJECT" destino: "CAPABILITIES" tipo: "contiene" peso: 10
  - origen: "PROJECT" destino: "SECURITY" tipo: "contiene" peso: 10
  - origen: "PROJECT" destino: "INFRASTRUCTURE" tipo: "contiene" peso: 10
  - origen: "PROJECT" destino: "DOCUMENTATION" tipo: "contiene" peso: 10
  - origen: "PROJECT" destino: "MEMORY" tipo: "contiene" peso: 10
  - origen: "PROJECT" destino: "DEPLOYMENT" tipo: "contiene" peso: 10
  - origen: "PROJECT" destino: "KNOWLEDGE" tipo: "contiene" peso: 10
  - origen: "PROJECT" destino: "TESTS" tipo: "contiene" peso: 10
  - origen: "PROJECT" destino: "DESIGN" tipo: "contiene" peso: 10
  - origen: "BUSINESS" destino: "ARCHITECTURE" tipo: "alimenta" peso: 7
  - origen: "ARCHITECTURE" destino: "CAPABILITIES" tipo: "afecta" peso: 8
  - origen: "ARCHITECTURE" destino: "DESIGN" tipo: "afecta" peso: 6
  - origen: "ARCHITECTURE" destino: "SECURITY" tipo: "afecta" peso: 5
  - origen: "CAPABILITIES" destino: "CAP:studio" tipo: "contiene" peso: 9
  - origen: "CAP:studio" destino: "CAP:admin" tipo: "depende_de" peso: 8
  - origen: "CAPABILITIES" destino: "MEMORY" tipo: "alimenta" peso: 7
  - origen: "MEMORY" destino: "KNOWLEDGE" tipo: "alimenta" peso: 8
  - origen: "MEMORY" destino: "INDICATORS" tipo: "alimenta" peso: 6
  - origen: "TESTS" destino: "CAP:studio" tipo: "valida" peso: 9
  - origen: "DEPLOYMENT" destino: "INFRASTRUCTURE" tipo: "depende_de" peso: 7

eventos:
  - origen: "BUSINESS" tipo: "modificacion" destino: "ARCHITECTURE" fecha: "2026-07-15"
    mensaje: "BUSINESS no documentado (falta README). Architecture no puede alinearse con propósito."
  - origen: "SECURITY" tipo: "alerta" destino: "PROJECT" fecha: "2026-07-15"
    mensaje: "Secretos hardcodeados detectados en deploy.bat y config.py."
