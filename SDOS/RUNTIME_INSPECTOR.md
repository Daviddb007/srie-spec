# Ω-0.4 — Runtime Inspector

> **Version:** 1.0.0
> **Parent:** OPERATIONAL_MODEL.md (Presence)
> **Depends on:** Ω-0.1, Ω-0.2

---

## Concept

The Runtime Inspector is not a dashboard. It is a **universal query interface** for SRIE. It answers four types of questions:

| Query | Command | Data source |
|-------|---------|-------------|
| **State** — What are you and how are you? | `inspect` | Runtime, Identity, Cognitive, Tasks, Modules |
| **History** — How did you get here? | `history` | Journal, Timeline, Checkpoints |
| **Explanation** — Why did you decide that? | `explain` | Hypotheses, Differential Diagnosis, Cases |
| **Projection** — What will happen next? | `predict` | Indicators, Simulation Engine (future) |

---

## Inspect tree

```
srie inspect                    → Full system tree (7 sections)
srie inspect identity           → Identity section
srie inspect runtime            → Runtime state
srie inspect cognitive          → Hypotheses + Intent
srie inspect tasks              → Task states (by state group)
srie inspect modules            → Loaded modules
srie inspect workspace          → Workspace identity
srie inspect health             → System health summary
```

---

## Why

```bash
srie why
```

Returns causal analysis of current runtime state:

- If FAILED: which modules failed, why
- If SUSPENDED: when it was paused, how to resume
- If RUNNING: all systems operational summary

---

## Explain

```bash
srie explain system             # System overview
srie explain score              # Current SRIE Score breakdown
srie explain decision           # Last decision with context
srie explain hypothesis <id>    # Full hypothesis trace
```

---

## History

```bash
srie history                    # Last 20 events
srie history -n 100             # Last 100 events
```

Events displayed in reverse chronological order with timestamp, source, and message.
