# Ω-0.5 — SRIE Mission Control

> **Version:** 1.0.0
> **Parent:** OPERATIONAL_MODEL.md
> **Depends on:** Ω-0.1, Ω-0.2, Ω-0.3, Ω-0.4

---

## Concept

Mission Control is SRIE's **Control Plane** — not a dashboard, not a shell. It is the graphical representation of the cognitive operating system. It consumes the same Runtime as the CLI and SDK, but presents it as an interrogable, navigable space.

## 4 Official Interfaces

```
CLI         → srie inspect, srie why, srie explain, srie history
SDK         → sdk.inspect(), sdk.why(), sdk.explain(), sdk.history()
REST API    → GET /inspect, GET /why, GET /explain, GET /history
Mission Control  → Visual representation of all three
```

## Entry Point

Mission Control opens showing the **Universe**, not a project. The universe is the top of the hierarchy:

```
Universe
  └── Organizations
        └── Workspaces
              └── Projects
                    └── Domains
                          └── Capabilities
                                └── Tasks
                                      └── Events
```

## 7 Operational Rooms

| Room | Content | Source |
|------|---------|--------|
| **Universe** | Global health, org map, workspace map, Universe Twin | universe.twin() |
| **Operations** | Runtime, Sessions, Activities, Tasks, Timeline, Checkpoints | runtime, session, task |
| **Intelligence** | Discovery, Indicators, Hypothesis, Diagnosis, Intent, Planner | cognitive, indicators, discovery |
| **Knowledge** | Graph, Ontology, Patterns, Cases, Memory, Journal, Reuse | graph, journal |
| **Engineering** | Capabilities, Sandbox, Repair, Deployment, Testing | (future) |
| **Resources** | Resource Broker, MCP, REST, SSH, SQL, SDK, Costs | (future) |
| **Governance** | Identity, Authority, Policies, Federation, Contracts | identity, registry |

## Question-Based Navigation

Instead of clicking buttons, the user asks questions:

| Question | Command | Response |
|----------|---------|----------|
| "What's happening?" | `inspect` | Full state tree |
| "Why is this degraded?" | `why` | Causal analysis |
| "What changed since yesterday?" | `history` | Event replay |
| "Explain this decision?" | `explain` | Hypothesis trace |
| "What would you do next?" | `recommend` | (future) |
| "What happens if we deploy?" | `predict` | (future) |

## Visual Grammar

Every entity (Universe, Organization, Workspace, Project, Domain, Capability, Task, Event) is rendered with:

1. **Identity card** — ID, name, type, state
2. **Inspect button** — show full state tree
3. **Explain button** — show reasoning context
4. **History button** — show event timeline
5. **Relations** — linked entities (parent, children, peers)
