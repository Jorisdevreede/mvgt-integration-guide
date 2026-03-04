# MVGT Integration Guide

**Wasteland wanted item:** [w-com-004](https://www.dolthub.com/repositories/steveyegge/wl-commons) — Write MVGT integration guide for non-Gas-Town systems

## What is this?

A guide documenting how external systems (agent frameworks, orchestrators, CI pipelines) can participate in the [Wasteland](https://steve-yegge.medium.com/welcome-to-the-wasteland-a-thousand-gas-towns-a5eb9bc8dc1f) federation using only [Dolt](https://github.com/dolthub/dolt) and the commons schema — **without running Gas Town**.

MVGT = **Minimum Viable Gas Town** — the smallest set of Dolt operations needed to participate.

## How this guide was created

This guide was planned using [Jeffrey Emanuel's](https://github.com/Dicklesworthstone) agent flywheel — a comprehensive orchestration system for managing fleets of AI coding agents at scale. The planning process behind this document is itself a demonstration of what a non-Gas-Town rig looks like in practice.

### Emanuel's Agent Flywheel

Jeffrey Emanuel built the agent flywheel as a production-grade orchestration layer for AI-driven software development. The system includes:

- **Multi-agent coordination** — Claude Code (Opus), Codex, and Gemini agents working in parallel across tmux panes, managed by NTM (Named Tmux Manager)
- **Structured planning** — Emanuel published his planning workflow on Twitter. From that, an open-source [plan-toolkit](https://github.com/Jorisdevreede/plan-toolkit) was derived that follows his planning process but points it at an existing codebase to plan new work in an existing project — the Jeffrey Emanuel way. The toolkit's `/shape` skill drives plans through 6 phases: prerequisites, drafting, multi-model critique (Claude + Codex reviewing each other's work), design integration, task creation with dependency graphs, and QA passes
- **Quality gates** — The flywheel comes with extensive quality gates built in and is easily extensible for a specific project or codebase. Example gates used in one project: TDD, unit tests, integration tests, E2E tests (Playwright), contract tests, SimpleCov coverage, RuboCop linting, Brakeman security scanning, and UBS static analysis
- **Agent Mail** — An MCP-based messaging system for inter-agent coordination with file reservation to prevent conflicts
- **Beads integration** — Integration with Steve Yegge's [Beads](https://github.com/steveyegge/beads) git-backed task tracking system for dependency wiring, priority management, and graph-scored triage (`bv --robot-triage`)
- **Respawn guardians** — Daemons that detect finished agents, spawn fresh ones with clean context, and let them self-assign work
- **Definition of Done** — Every deliverable passes through quality gates before it's considered complete

This guide went through 4 critique rounds with 30 findings integrated before a single word of the actual guide was written. Planning tokens are cheap — debugging tokens from bad plans are expensive. That's the flywheel philosophy.

### A product person, not an engineer

Worth noting: the person operating this flywheel ([Joris de Vreede](https://github.com/Jorisdevreede)) is a product person, not a software engineer. Using Emanuel's agent flywheel, he built a Rails 8 modular monolith (18 bounded context engines, 475 beads closed, 3400+ RSpec tests, full Playwright E2E coverage) with **0 SonarQube findings**. The flywheel's procedures and checkpoints make it possible for non-engineers to produce production-quality software — which is exactly the kind of capability the Wasteland is designed to recognize and reward through its stamp-based reputation system.

## Case Study

This guide uses Emanuel's agent flywheel as the primary case study — a production agent orchestrator that completed the full MVGT flow (Dolt install, DoltHub auth, commons fork, rig registration, wanted board claim, PR submission) on March 4, 2026, without any dependency on Gas Town.

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
6. Case Study — Emanuel's agent flywheel
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
