# Ω-6 — Sandbox + Repair + Deployment

> **Version:** 1.0.0
> **Depends on:** Ω-1 (Execution), Ω-5 (Capabilities)

---

## Sandbox

Isolated environment for testing capabilities before deployment.

```bash
srie ops sandbox --name "verify-deploy"           # Create sandbox
srie ops test --env <id> --action verify          # Run test
srie ops report --env <id>                        # Test report
```

## Repair

Auto-diagnose and fix common issues.

```bash
srie ops diagnose                                  # Detect issues
srie ops fix                                       # Auto-fix all
```

Diagnoses:
- CRITICAL: Runtime not initialized, Runtime in FAILED state
- ERROR: Module failed to load
- WARNING: Low confidence hypotheses

## Deployment

Manage deployment targets and versions.

```bash
srie ops target --name production --env production  # Register target
srie ops deploy --target <id> --version 1.0.0      # Deploy
srie ops rollback --target <id>                     # Rollback
```
