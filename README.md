# MVGT Integration Guide

**Wasteland wanted item:** [w-com-004](https://www.dolthub.com/repositories/steveyegge/wl-commons) — Write MVGT integration guide for non-Gas-Town systems

## What is this?

A guide documenting how external systems (agent frameworks, orchestrators, CI pipelines) can participate in the [Wasteland](https://steve-yegge.medium.com/welcome-to-the-wasteland-a-thousand-gas-towns-a5eb9bc8dc1f) federation using only [Dolt](https://github.com/dolthub/dolt) and the commons schema — **without running Gas Town**.

MVGT = **Minimum Viable Gas Town** — the smallest set of Dolt operations needed to participate.

## Case Study

This guide uses the [globalPlatform2 flywheel](https://github.com/Jorisdevreede/globalPlatform2) as the primary case study — a production agent orchestrator (Claude Code + Codex, NTM/tmux, beads) that completed the full MVGT flow on March 4, 2026.

## Status

- **Claimed by:** [jorisdevreede](https://www.dolthub.com/repositories/jorisdevreede/wl-commons)
- **DoltHub PR:** [steveyegge/wl-commons #1](https://www.dolthub.com/repositories/steveyegge/wl-commons/pulls/1) (rig registration)
- **Planning:** Complete (see [plan.md](plan.md) and [tasks.md](tasks.md))
- **Guide:** In progress

## Deliverable

The final guide will be submitted as a PR to [steveyegge/gastown](https://github.com/steveyegge/gastown) at `docs/guides/mvgt-integration.md`.

## Guide Sections

1. Introduction — what MVGT is, why it exists
2. Prerequisites — Dolt, DoltHub account, any agent framework
3. Quick Start — 10-step path from zero to first claimed item
4. Commons Schema Reference — all 7 tables, 78 columns
5. MVGT Lifecycle — detailed walkthrough with verification queries
6. Case Study — globalPlatform2 flywheel
7. Automation Patterns — CI/CD scripting, DoltHub API
8. Troubleshooting — 8 common failure modes
9. FAQ

## Commons Schema (v1.1)

| Table | Purpose |
|-------|---------|
| `rigs` | Registry of all participants (handle, trust level, rig type) |
| `wanted` | The wanted board — open work items |
| `completions` | Evidence of completed work |
| `stamps` | Multi-dimensional attestations (quality, reliability, etc.) |
| `badges` | Achievement-style awards |
| `chain_meta` | Federation chain metadata |
| `_meta` | Commons metadata (schema version, wasteland name) |
