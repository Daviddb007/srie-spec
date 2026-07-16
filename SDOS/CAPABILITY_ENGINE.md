# Ω-5 — Capability Engine

> **Version:** 1.0.0
> **Depends on:** Ω-1 (Execution), Ω-4 (Planner)

---

## Concept

The Capability Engine matches execution steps to available capabilities and their required resources. It answers:

- "Can we perform this action?" → capability matching
- "What resources do we need?" → resource aggregation
- "What's the cost?" → cost estimation
- "What's missing?" → resource gap analysis

## Built-in capabilities (8)

| Capability | Actions | Resources | Cost |
|------------|---------|-----------|------|
| Project Discovery | discover | filesystem, git | 1 |
| Maturity Indicators | indicators | digital_twin | 1 |
| Project Diagnosis | diagnose | digital_twin, indicators | 2 |
| Workflow Planner | plan | planner | 2 |
| Code Implementation | implement | ai, filesystem | 5 |
| Test Execution | test, verify | sandbox, filesystem | 2 |
| Deployment | deploy | deployment, credentials | 3 |
| Auto Repair | repair | digital_twin, ai | 3 |

## Commands

```bash
srie cap list                                # List all capabilities
srie cap match deploy                        # Find capabilities for an action
srie cap plan --goal "Build MVP"              # Full capability plan
```
