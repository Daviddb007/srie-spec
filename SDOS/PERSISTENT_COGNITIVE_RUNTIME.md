# Ω-0.1 — Persistent Cognitive Runtime (PCR)

> **Goal:** SRIE can pause/resume, checkpoint/restore, and maintain cognitive continuity across restarts.
> **Principle:** Never think the same thing twice.
> **Parent:** OPERATIONAL_MODEL.md (Rhythms, Presence, Recovery)

---

## What survives

| Domain | Contents | Format | Frequency |
|--------|----------|--------|-----------|
| **Runtime Snapshot** | State, active modules, loaded resources, policies, queues | YAML | Every state transition |
| **Timeline** | Ordered events with session context | JSONL | Every event |
| **Event Store** | Full event history with evidence + confidence | JSON | Every operation |
| **Cognitive State** | Active hypotheses, confidence scores, evidence counts | JSON | On change |
| **Intent State** | Current objective, priority, next milestone | YAML | On user set |
| **Digital Twin** | Project nodes, relationships, indicators | JSON | Every sync |
| **Checkpoint** | Full frozen state (all of the above) | tarball | Configurable interval |
| **Journal** | Human-readable cognitive history | Markdown | Append only |

---

## Filesystem layout

```
SDOS/
├── runtime.lock                 # Runtime manifest (always current)
├── timeline.journal.md          # Append-only cognitive history
├── state/
│   ├── runtime.yaml             # Runtime Snapshot
│   ├── cognitive.json           # Cognitive State
│   ├── intent.yaml              # Intent State
│   └── events/                  # Event Store
│       ├── evt-001.json
│       ├── evt-002.json
│       └── ...
├── checkpoints/
│   ├── cp-001.tar.gz
│   ├── cp-002.tar.gz
│   └── INDEX.yaml
└── SRIE_DIGITAL_TWIN.json       # Digital Twin
```

---

## Key operations

```python
# Runtime
runtime.pause()      → serialize all state, write checkpoint, state → SUSPENDED
runtime.resume()     → load last checkpoint, verify integrity, state → OPERATIONAL
runtime.checkpoint() → snapshot current state without pausing
runtime.restore(id)  → load specific checkpoint, replay events to catch up

# Journal
journal.append(event)      # Structured event
journal.narrative(text)    # Human-readable entry
journal.replay(since=...)  # Get events since timestamp

# Cognitive State
cognitive.hypothesis(id, confidence, evidence)
cognitive.validate(id, new_evidence)
cognitive.archive(id)

# Intent
intent.set(objective, priority, milestone)
intent.current()
```

---

## Checkpoint lifecycle

```
Every N operations OR every M minutes:
  1. Pause module execution
  2. Flush event queue
  3. Snapshot Runtime State
  4. Snapshot Cognitive State
  5. Snapshot Twin (current version)
  6. Compress → cp-NNN.tar.gz
  7. Update CHECKPOINT_INDEX.yaml
  8. Resume module execution

On restore:
  1. Load INDEX.yaml → find latest (or specific) checkpoint
  2. Extract cp-NNN.tar.gz
  3. Verify integrity (hash check)
  4. Restore Runtime State
  5. Restore Cognitive State
  6. Restore Twin
  7. Replay events from checkpoint timestamp to present
  8. Resume
```

---

## Operational Age

```yaml
operational_age:
  born: "2026-07-15T10:00:00Z"
  uptime_seconds: 1504800        # 418 hours
  operational_hours: 1177
  sessions: 124
  checkpoints: 892
  learnings: 81
  hypotheses_confirmed: 216
  projects_known: 14
  domains: 7
  capabilities: 52
```

Updated every checkpoint. Provides the system's biography.
