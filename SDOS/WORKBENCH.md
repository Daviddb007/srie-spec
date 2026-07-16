# Ω-3 — Workbench

> **Version:** 1.0.0
> **Depends on:** Ω-1 (Execution Orchestrator), Ω-2 (PDL), Ω-0.5 (Mission Control)

---

## Concept

The Workbench is the **operational workspace** where SRIE executes and supervises work.
It is a room within Mission Control that replaces static dashboards with live process management.

## What it shows

- Active executions with progress bars
- Workflow steps and their states (PENDING, RUNNING, COMPLETED, BLOCKED, FAILED)
- Step evidence (what each step produced)
- Execution history (created, started, completed timestamps)

## API

```bash
GET /api/workbench              # Execution status + list
GET /api/workbench/execution/X  # Execution detail + workflows + steps
```

## Mission Control room

The Workbench room is accessible from the sidebar in Mission Control, alongside Universe, Operations, Intelligence, Knowledge, and Governance.
