# Ω-1 — Execution Orchestrator

> **Version:** 1.0.0
> **Depends on:** Ω-0.1, Ω-0.2 (Activity, Task), Ω-0.4 (Inspector)

---

## Concept

SRIE knows how to discover, measure, reason, persist, remember, explain, and live.
It does NOT yet know how to **execute long-running work**.

The Execution Orchestrator adds:

```
EXECUTION (weeks–months, goal-oriented)
    │
    ├── Goal, Priority, State, Progress
    │
    ├── WORKFLOW (ordered sequence)
    │       │
    │       ├── Steps with actions
    │       │
    │       └── STEP (atomic operation)
    │               │
    │               ├── Action, State, Evidence
    │               │
    │               └── EVIDENCE (output artifact)
    │
    └── Leverages Ω-0.2 Activity + Task hierarchy
```

## States

### Execution
- PENDING → RUNNING → COMPLETED
- RUNNING → BLOCKED → RUNNING
- RUNNING → FAILED

### Step
- PENDING → RUNNING → COMPLETED
- RUNNING → SKIPPED
- RUNNING → FAILED

## Commands

```bash
srie exec create --goal "Build MVP" --priority high
srie exec start <exec-id>
srie exec workflow <exec-id> --name "Build Pipeline"
srie exec complete <exec-id>
srie exec status
srie exec list
```
