# Ω-0.3 — Universe Model

> **Version:** 1.0.0
> **Parent:** OPERATIONAL_MODEL.md
> **Depends on:** Ω-0.1, Ω-0.2, Ω-0.4

---

## Hierarchy

```
UNIVERSE (months–years)
   │
   ├── ID, Name, State, Policies
   │
   ├── ORGANIZATION (months–years)
   │       │
   │       ├── ID, Name, State
   │       │
   │       ├── WORKSPACE (months–years)
   │       │       │
   │       │       ├── ID, Name, Context, State
   │       │       │
   │       │       ├── PROJECT (weeks–months)
   │       │       │       │
   │       │       │       ├── Sessions, Activities, Tasks
   │       │       │       │
   │       │       │       └── DOMAIN (specialized SRIE)
   │       │       │               │
   │       │       │               └── Capabilities
   │       │       │
   │       │       └── ...
   │       │
   │       └── ...
   │
   └── ...
```

## Ownership Chain

Every entity can answer "who do I belong to?":

```
entity → workspace → organization → universe
```

## Commands

```bash
srie universe init --name "Stonelytics"     # Create universe
srie universe org "SRIE Lab"                 # Register org
srie universe workspace Studio --org ORG-xxx # Register workspace
srie universe project "MVP" --workspace WS   # Register project
srie universe status                         # Universe health
srie universe chain                          # Ownership tree
srie universe twin                           # Universe Digital Twin
srie inspect universe                        # Through Inspector
```

## Universe Digital Twin

```json
{
  "total_organizations": 3,
  "total_workspaces": 7,
  "total_projects": 14,
  "total_domains": 6,
  "state": "ACTIVE"
}
```
