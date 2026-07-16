# Ω-7 — Knowledge + Evolution

> **Version:** 1.0.0
> **Depends on:** Ω-2 (Work Log), Ω-6 (Ops)

---

## Knowledge Engine

Extracts patterns, stores cases, and enables cross-project knowledge reuse.

- **Patterns**: Reusable knowledge (success, failure, optimization, antipattern)
- **Cases**: Execution outcomes with context for similarity matching
- **Reuse**: Suggests past solutions based on context similarity

## Evolution Engine

Detects drift and suggests improvements based on system history.

- **Drift detection**: Score drops, confidence degradation, work imbalance
- **Evolution suggestions**: Prioritized improvement recommendations
- **Auto-evolve**: Extracts patterns from detected issues

## Commands

```bash
srie pattern --type success --title "Flask detected"     # Extract pattern
srie case --action discover --outcome success             # Store case
srie knowledge                                             # Knowledge base status
srie drift                                                 # Detect drift
srie evolve                                                # Get suggestions
srie auto                                                  # Auto-evolve
```
