# KERNEL — El Núcleo del Runtime

---

## ¿Qué es este documento?

KERNEL.md define el **kernel del Runtime SRIE**. Es el núcleo mínimo que debe estar vivo para que el sistema funcione. El kernel arranca, carga la configuración, inicia el Event Loop, arranca el Scheduler, carga los Engines y mantiene el sistema operando.

---

## Responsabilidades del Kernel

```yaml
kernel:
  responsabilidades:
    - "Arrancar y detener el Runtime"
    - "Inicializar el Event Loop"
    - "Inicializar el Scheduler"
    - "Cargar y verificar Engines"
    - "Gestionar el ciclo de vida del sistema"
    - "Mantener el heartbeat del Runtime"
    - "Propagar eventos críticos"
    - "Coordinar el apagado seguro"
```

---

## Arranque del Kernel

```yaml
kernel_boot:
  secuencia:
    1. "Cargar SDOS/RUNTIME.yaml"
    2. "Verificar Foundation (versión, documentos mínimos)"
    3. "Verificar identidad (SDOS/IDENTITY.yaml)"
    4. "Inicializar Memory Manager"
    5. "Inicializar Knowledge Manager"
    6. "Inicializar Digital Twin Manager"
    7. "Inicializar Resource Broker"
    8. "Inicializar Execution Layer"
    9. "Inicializar Event Loop"
    10. "Inicializar Scheduler"
    11. "Cargar Engines registrados"
    12. "Iniciar Observability"
    13. "Emitir evento RUNTIME_STARTED"
    14. "Cambiar estado a RUNNING"

  si_falla:
    paso_3: "No hay identidad → ejecutar srie identity"
    paso_6: "No hay Twin → ejecutar Discovery"
    paso_11: "Engine falla al cargar → marcar como ERROR, continuar"
```

---

## Apagado del Kernel

```yaml
kernel_shutdown:
  secuencia:
    1. "Detener Scheduler (esperar tareas en curso)"
    2. "Detener Event Loop (procesar eventos pendientes)"
    3. "Guardar estado del Runtime"
    4. "Cerrar conexiones de recursos"
    5. "Detener Engines (cada uno guarda su estado)"
    6. "Emitir evento RUNTIME_STOPPED"
    7. "Cambiar estado a STOPPED"

  forcado:
    timeout_ms: 10000
    accion: "Kill -9, reconstructir desde última evidencia"
```

---

## Engine Loader

```yaml
engine_loader:
  carga:
    proceso:
      - "Leer engine.yaml del Engine"
      - "Verificar nivel SRIE requerido"
      - "Verificar dependencias"
      - "Registrar en el Runtime"
      - "Cambiar estado a IDLE"

    si_falla:
      - "Engine no encontrado → registrar, continuar"
      - "Dependencia faltante → registrar, continuar"
      - "Nivel insuficiente → registrar, esperar"
```

---

## Heartbeat del Runtime

```yaml
runtime_heartbeat:
  intervalo_segundos: 30
  contenido:
    timestamp: string
    estado: string
    engines_activos: integer
    resources_disponibles: integer
    memoria_usada_mb: integer
    uptime_segundos: integer
  archivo: "SDOS/RUNTIME_HEARTBEAT.log"
```
