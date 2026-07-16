# Ω-2 — Process Definition Language (PDL)

> **Version:** 1.0.0
> **Depends on:** Ω-1 (Execution Orchestrator)

---

## Concept

PDL is a declarative YAML language for describing work in SRIE.
Instead of running commands manually, you define a process and let SRIE execute it.

## Format

```yaml
id: exec-build-mvp
goal: Build MVP
priority: high

workflows:
  - name: Analysis
    steps:
      - name: Discover project
        action: discover
      - name: Calculate indicators
        action: indicators

  - name: Build
    steps:
      - name: Implement
        action: implement
      - name: Test
        action: test

success:
  conditions:
    - metric: score
      operator: ">"
      value: 85
```

## Known actions

`discover`, `indicators`, `diagnose`, `plan`, `implement`, `test`, `verify`, `deploy`, `repair`

## Commands

```bash
srie pdl template --name build-mvp    # Generate template
srie pdl validate --file build-mvp.yaml # Validate syntax
srie pdl run --file build-mvp.yaml     # Execute
srie pdl list                          # Saved definitions
```
