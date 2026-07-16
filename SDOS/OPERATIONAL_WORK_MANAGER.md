# Ω-0.2 — Operational Work Manager

> **Version:** 1.0.0
> **Parent:** OPERATIONAL_MODEL.md (Rhythms, Presence)
> **Depends on:** Ω-0.1 (Persistent Cognitive Runtime)

---

## Work Hierarchy

```
WORKSPACE (months–years)
    │
    ├── Identity, Organization, Policies, Credentials
    │
    ├── PROJECT (weeks–months)
    │       │
    │       ├── Digital Twin, Knowledge, Timeline
    │       │
    │       ├── SESSION (minutes–hours)
    │       │       │
    │       │       ├── Start → End, Duration, User
    │       │       │
    │       │       ├── ACTIVITY (logical unit of work)
    │       │       │       │
    │       │       │       ├── Objective, Hypothesis, Confidence
    │       │       │       ├── Reasoning Context (cost, models, tools)
    │       │       │       │
    │       │       │       └── TASK (atomic unit)
    │       │       │               │
    │       │       │               ├── States: PENDING | RUNNING | COMPLETED
    │       │       │               │         | BLOCKED | DELEGATED
    │       │       │               │         | WAITING_INPUT | RESUMABLE | FAILED
    │       │       │               │
    │       │       │               └── EVENTS (from Ω-0.1 Event Store)
    │       │       │
    │       │       └── Session DNA (work pattern over time)
    │       │
    │       └── ...
    │
    └── ...
```

---

## Task States

| State | Meaning | Can transition to |
|-------|---------|-------------------|
| PENDING | Created, not started | RUNNING |
| RUNNING | Currently executing | COMPLETED, BLOCKED, FAILED |
| COMPLETED | Finished successfully | — (terminal) |
| FAILED | Finished with error | PENDING (retry) |
| BLOCKED | Waiting for resource or evidence | RUNNING, WAITING_INPUT |
| DELEGATED | Assigned to another Domain/SRIE | COMPLETED, FAILED |
| WAITING_INPUT | Requires human decision | RUNNING, COMPLETED |
| RESUMABLE | Interrupted, can continue | RUNNING |

---

## Filesystem layout

```
SDOS/
├── work/
│   ├── workspace.yaml          # Workspace identity and config
│   ├── sessions/
│   │   ├── ses-001.yaml        # Session metadata
│   │   ├── ses-002.yaml
│   │   └── INDEX.yaml
│   ├── activities/
│   │   ├── act-001.yaml        # Activity with reasoning context
│   │   ├── act-002.yaml
│   │   └── INDEX.yaml
│   ├── tasks/
│   │   ├── task-001.yaml       # Task with state transitions
│   │   ├── task-002.yaml
│   │   └── INDEX.yaml
│   └── dna/
│       └── session-dna.json    # Aggregated work patterns
└── ... (from Ω-0.1)
```

---

## Key operations

```python
# Session
session = Session.start(project_id, user="david")
session.end()
session.current()

# Activity
activity = session.begin_activity("discovery", objective="Inventory project")
activity.complete(hypotheses=5, confidence=0.87)
activity.replay()  # Returns event sequence

# Task
task = activity.create_task("Scan git history", depends_on=["task-001"])
task.transition("RUNNING")
task.transition("COMPLETED")

# Reasoning Context
activity.reasoning(
    hypothesis_count=14,
    confirmed=5,
    discarded=9,
    confidence=0.87,
    cost_usd=0.18,
    models=["GPT-5.5", "Claude"],
    tools=["Git", "Filesystem", "Docker"],
)

# Session DNA
dna = workspace.dna()
# Returns: {"discovery": 0.7, "planning": 0.2, "repair": 0.1}
```

---

## Activity Replay

```python
activity = Activity.load("act-081")
for event in activity.replay():
    print(f"{event.timestamp}: {event.message}")
# 08:10: Discovery started
# 08:11: Git found
# 08:12: Flask found
# 08:13: Hypothesis created
# ...
```

Replay reads from the Event Store (Ω-0.1), filtered by activity_id.

---

## Integration with Ω-0.1

The Journal, Cognitive State, and Checkpoint systems already record per-event data. The Work Manager adds:

1. **Session boundaries** (start/end) to the Journal
2. **Activity IDs** to each event for filtering/replay
3. **Task state transitions** to the Cognitive State
4. **Work consolidation** at session end → Session DNA
