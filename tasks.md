# Tasks — non-gastown-integration

Wasteland wanted item: **w-com-004** — Write MVGT integration guide for non-Gas-Town systems

## Summary

| ID | Title | Priority | Dependencies | Status |
|----|-------|----------|--------------|--------|
| gp-x8o | Clone gastown repo and identify guide location | P0 | — | open |
| gp-d97 | Write guide — Introduction, Prerequisites, and Quick Start | P1 | x8o | open |
| gp-kih | Write guide — Commons Schema Table Reference | P1 | x8o | open |
| gp-ay9 | Write guide — Schema Relationships, Patterns, and ERD | P1 | kih | open |
| gp-kod | Write guide — MVGT Lifecycle Walkthrough | P1 | kih | open |
| gp-14c | Write guide — Case Study: globalPlatform2 Flywheel | P1 | kih | open |
| gp-24i | Write guide — Automation Patterns | P2 | kod | open |
| gp-9o4 | Write guide — Troubleshooting | P1 | kod | open |
| gp-87m | Write guide — FAQ | P2 | d97, ay9, kod, 14c, 24i, 9o4 | open |
| gp-347 | Review, polish, and submit PR | P1 | 87m | open |

## Parallel Execution Tracks

```
T1 (clone repo)
├── Track A: T2 (intro + prereqs + quick start)
├── Track B: T3a → T3b (schema tables → ERD + patterns)
├── Track C: T4 → T6a, T6b (lifecycle → automation, troubleshooting)
├── Track D: T5 (case study, after T3a)
└── Final: T6c (FAQ) → T7 (review + PR)
```

Estimated: ~4.5h sequential, ~3h with 2 parallel agents.
