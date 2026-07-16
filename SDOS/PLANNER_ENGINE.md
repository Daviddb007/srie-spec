# Ω-4 — Planner Engine

> **Version:** 1.0.0
> **Depends on:** Ω-1 (Execution), Ω-2 (PDL), Ω-3 (Workbench)

---

## Concept

The Planner Engine does not invent steps. It **optimizes workflows** by evaluating constraints, resolving dependencies, and estimating cost/time.

## Capabilities

- **Goal inference**: "Build MVP" → 8-step workflow (discover → indicators → diagnose → plan → implement → test → verify → deploy)
- **Dependency resolution**: Steps are ordered respecting action dependencies
- **Cost/time estimation**: Each action has base cost and time
- **Parallel simulation**: Groups parallelizable steps for efficiency calculation
- **Orchestration**: Plan + Execute in one command

## Commands

```bash
srie plan --goal "Build MVP" --priority high      # Generate optimized plan
srie simulate --goal "Build MVP"                    # Simulation with efficiency
srie orchestrate --goal "Quick demo"                # Plan + Execute
```

## Action costs

| Action | Time (min) | Cost | Depends on |
|--------|-----------|------|------------|
| discover | 2 | 1 | — |
| indicators | 1 | 1 | discover |
| diagnose | 3 | 2 | indicators |
| plan | 2 | 2 | diagnose |
| implement | 10 | 5 | plan |
| test | 3 | 2 | implement |
| verify | 2 | 1 | test |
| deploy | 5 | 3 | verify |
| repair | 5 | 3 | diagnose |
