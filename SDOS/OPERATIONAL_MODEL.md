# SRIE Operational Model

> **Version:** 1.0.0
> **Status:** Ratified
> **Domain:** OPERATION

---

## Preamble

This document does not describe architecture. It describes **behavior**. It answers a single question:

> **How does SRIE behave when nobody is watching?**

SRIE is not an engine that executes commands. SRIE is an operating system for engineering that **lives**. It has rhythms, states, memory, and the ability to recover. This document defines the rules by which it remains alive, useful, and trustworthy over months and years.

---

## Operational Principles

Five rules that govern every Runtime instance, every session, every module, and every decision. These are not guidelines. They are inviolable.

### 1. Never lose context

Every action, every decision, every result must be traceable to the session, project, identity, and evidence that produced it. A Runtime without context is not SRIE — it is a calculator.

### 2. Never execute without identity

Every operation must be associated with a verified identity. No anonymous execution. No background task without a known domain_id. Identity is the root of trust.

### 3. Never assert without evidence

Every claim a module makes (language detected, maturity score, recommendation) must include confidence, source, and supporting evidence. Unexplained output is a bug.

### 4. Never forget without consolidating knowledge

Before archiving any data, extract patterns, anti-patterns, decisions, and lessons. Knowledge consolidation is mandatory before any data lifecycle transition.

### 5. Always recover before restarting

When a Runtime session fails, the first response is recovery — not restart. Replay the event store, reconstruct the twin, verify module states, and continue. Restart is the last resort.

---

## Operational States

SRIE is never truly "off". It transitions between these states:

```
DORMANT ──── System installed, never booted. No identity, no session.
     │
     ▼
BOOTING ──── Identity verified, Runtime loading, modules discovering.
     │
     ▼
OPERATIONAL ── Normal operation. User may or may not be present.
     │
     ├──► BUSY ──── Processing a user request or scheduled task.
     │       │
     │       ▼
     │   OPERATIONAL
     │
     ├──► LEARNING ── Consolidating knowledge, reindexing, recalibrating.
     │       │
     │       ▼
     │   OPERATIONAL
     │
     ├──► MAINTENANCE ── Housekeeping: log rotation, cache cleanup, archiving.
     │       │
     │       ▼
     │   OPERATIONAL
     │
     ├──► DEGRADED ── Non-critical error. Partial functionality.
     │       │
     │       ▼
     │   OPERATIONAL or RECOVERING
     │
     └──► RECOVERING ── Session replay, twin reconstruction, module verification.
             │
             ▼
         OPERATIONAL or DORMANT

SUSPENDED ──── Long-term inactivity. Session archived, twin frozen.
     │
     ▼
ARCHIVED ──── Project closed. Data preserved but Runtime terminated.
```

**State transitions are logged.** Every transition produces an event in the Event Store with session_id, timestamp, previous state, new state, and reason.

---

## Biological Clock: The Rhythms of SRIE

SRIE does not have a scheduler. It has a **Biological Clock** — a hierarchy of rhythms that govern when and how work happens.

### Rhythm 1: Startup (milliseconds to seconds)

Triggered by: `srie init`, `SDK.init()`, Studio opening a workspace.

```
Power On
    │
    ├── 1. Load Runtime Manifest (runtime.lock)
    ├── 2. Verify Identity (IDENTITY.yaml)
    ├── 3. Open or Create Session
    ├── 4. Load Workspace (workspace.yaml)
    ├── 5. Load Project Registry
    ├── 6. Module Loader → Discover + Resolve DAG + Load
    ├── 7. Synchronize Twin (compare disk state with twin)
    ├── 8. Verify Resource Health
    ├── 9. Emit: SESSION_STARTED
    └── 10. State → OPERATIONAL
```

**If any step fails:** transition to RECOVERING, replay last known good session.

### Rhythm 2: Alive (continuous,无人值守)

Executed automatically while the Runtime is in OPERATIONAL state. No user interaction required.

| Interval | Action | Responsibility |
|----------|--------|----------------|
| Every 30s | Heartbeat: update runtime.lock uptime, module states | Kernel |
| Every 60s | Health Check: verify module responsiveness | Lifecycle |
| Every 5min | Resource Health: check disk, memory, connectivity | Resource Broker |
| Every 1hr | Twin Sync: re-scan project for changes, update twin | Discovery Module |
| Daily (03:00) | Reindex: full discovery scan, recalibrate indicators | Discovery + Indicators |
| Weekly (Sun 03:00) | Knowledge Consolidation: extract patterns from event store | Knowledge Module |
| Monthly (1st, 03:00) | Evolution Review: compare twin history, detect drift | Indicators + Memory |
| Quarterly | Health Report: generate operational summary | Observability |

**These are defaults.** Each interval is configurable per-workspace in `workspace.yaml`.

### Rhythm 3: Interaction (user-driven)

Triggered by any user action via CLI, SDK, or Studio.

```
User Action
    │
    ├── 1. Identify: verify caller identity and permissions
    ├── 2. Intent: classify the action (query, command, request)
    ├── 3. Route: dispatch to appropriate Runtime service or Module
    ├── 4. Execute: run the operation
    ├── 5. Evidence: capture all output with confidence and sources
    ├── 6. Knowledge: check if result should be persisted as pattern
    ├── 7. Response: return result to caller
    └── 8. Log: record session_id, action, duration, result
```

### Rhythm 4: Sleep (graceful shutdown)

Triggered by: Studio closing, `srie shutdown`, system idle timeout.

```
Sleep
    │
    ├── 1. Pause Module Loader (no new discoveries)
    ├── 2. Complete pending operations (timeout: 30s)
    ├── 3. Flush Event Store (pending events → disk)
    ├── 4. Save Session state (session_id, timeline, decisions)
    ├── 5. Save Digital Twin (version + 1, last_sync = now)
    ├── 6. Save Timeline (session events)
    ├── 7. Update Runtime Manifest (state → SUSPENDED)
    ├── 8. Close connections (resources, vault, network)
    ├── 9. Emit: SESSION_SUSPENDED
    └── 10. State → SUSPENDED
```

**Timeout:** If sleep takes longer than 60 seconds, force state to SUSPENDED and log warning.

### Rhythm 5: Recovery (crash recovery)

Triggered by: unexpected shutdown, Runtime error, module failure.

```
Crash Detected
    │
    ├── 1. Load last known Runtime Manifest
    ├── 2. Identify last known good Session
    ├── 3. Replay Event Store from last checkpoint
    ├── 4. Reconstruct Digital Twin from persisted state
    ├── 5. Verify Module states (check each module.yaml)
    ├── 6. Re-synchronize with project filesystem
    ├── 7. Compare twin version with expected (detect drift)
    ├── 8. Emit: SESSION_RECOVERED or SESSION_RECOVERY_FAILED
    ├── 9. If recovered → State → OPERATIONAL
    └── 10. If failed → State → DORMANT (manual intervention required)
```

---

## Presence

SRIE has three levels of presence:

| Level | User visible | Runtime active | Modules loaded | Persistence |
|-------|-------------|----------------|----------------|-------------|
| **Present** | Yes (Studio open) | Full | All | Real-time |
| **Observing** | No (Studio closed) | Alive rhythm only | Minimal (identity, registry, discovery) | Periodic sync |
| **Dormant** | No | None | None | Frozen (last state preserved) |

**Observing mode** is the default when Studio closes. The Alive rhythm continues:
- Heartbeat, health checks, and twin sync still execute
- Knowledge consolidation runs on schedule
- The system remains recoverable within milliseconds

**Dormant mode** activates after 7 days of inactivity:
- All rhythms pause
- Session is compressed and archived
- Runtime manifest marked SUSPENDED
- Full recovery requires a Startup cycle

---

## Event Store

Every operation in every session produces events. The Event Store is the single source of truth for reconstructing state.

### Event schema

```yaml
event:
  id: "evt-<uuid>"
  session_id: "ses-<uuid>"
  timestamp: "2026-07-16T10:00:00Z"
  type: "MODULE_LOADED" | "DISCOVERY_COMPLETE" | "INDICATOR_CALCULATED" | ...
  source: "kernel" | "module:discovery" | "module:indicators" | ...
  data: { ... }
  confidence: 0.95
  evidence: [ ... ]
```

### Event retention

| Age | Storage | Queryable |
|-----|---------|-----------|
| < 7 days | Hot (memory + fast storage) | Yes, full |
| 7-90 days | Warm (compressed JSON) | Yes, indexed |
| 90-365 days | Cold (archived per month) | Yes, with delay |
| > 365 days | Knowledge-extracted only | No raw events |

**Before archiving cold:** extract patterns, decisions, and metrics → Knowledge.

---

## Operational Model vs Runtime Model

| Dimension | Runtime Model (today) | Operational Model (this document) |
|-----------|----------------------|-----------------------------------|
| Scope | Single execution | Continuous existence |
| States | RUNNING, STOPPED, FAILED | DORMANT, BOOTING, OPERATIONAL, SUSPENDED, RECOVERING |
| Rhythm | Command → Execute → Return | Startup → Alive → Interaction → Sleep → Recovery |
| Persistence | File-based (YAML) | Session + Event Store + Timeline + Twin |
| Presence | Active only when invoked | Present, Observing, or Dormant |
| Failure | Raise error | Recover, replay, continue |
| Time horizon | Seconds to minutes | Months to years |

---

## Operational Model Compliance

A Runtime instance is **Operationally Compliant** if and only if:

- [ ] It logs every state transition
- [ ] It runs Alive rhythms on schedule
- [ ] It persists session state before shutdown
- [ ] It can recover from a crash by replaying the Event Store
- [ ] It never executes without a verified identity
- [ ] It never produces output without confidence and evidence
- [ ] It consolidates knowledge before archiving any data
- [ ] It operates in Observing mode when no user is present
- [ ] It can transition to Dormant after 7 days of inactivity
- [ ] It can reconstruct the Digital Twin from persisted state

These ten criteria define whether SRIE is truly **alive** or merely running.
