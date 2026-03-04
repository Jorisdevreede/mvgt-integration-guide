# Plan — MVGT Integration Guide (w-com-004)

## 1. Goals / Non-goals

### Goals
- Write a comprehensive markdown guide documenting how non-Gas-Town systems can participate in the Wasteland federation using only Dolt and the commons schema
- Use Emmanuel's agent flywheel as the primary case study — a real, production-grade orchestrator that completed the MVGT flow
- Document the full commons schema (7 tables) with field-level explanations
- Provide copy-paste-ready Dolt CLI commands for every lifecycle step
- Cover the complete participation lifecycle: install → auth → fork → register → browse → claim → work → submit → get stamped
- Deliver as a PR to `steveyegge/gastown` (likely `docs/guides/mvgt-integration.md`)

### Non-goals
- Not a Gas Town installation or usage guide (that exists separately)
- Not a Dolt tutorial — assume basic SQL knowledge, link to Dolt docs for deep dives
- Not a protocol specification — that's `w-hop-001`, a separate wanted item
- Not covering Gas City, the declarative orchestrator builder
- Not documenting the Spider Protocol fraud detection or chain constitution internals
- Not building any software — this is a documentation-only deliverable

## 2. User Stories + Success Criteria

### US-1: Framework Author Integration
As a developer who built their own agent orchestrator (like a flywheel, custom tmux rig, or CI pipeline), I can read this guide and participate in the Wasteland within 30 minutes, so that my rig earns stamps and builds reputation without switching to Gas Town.

**Acceptance criteria:**
- Guide includes a "Quick Start" fast path (register rig + claim one open item + submit evidence) achievable in ~30 minutes, plus the full lifecycle walkthrough for complete integration
- All Dolt CLI commands are copy-paste ready
- No Gas Town dependency in any step
- Quick Start path explicitly defers sandbox compliance, chain integrity, and federation concerns to later sections

### US-2: Schema Understanding
As a developer integrating with the Wasteland, I can understand every table and column in the commons schema, so that I can write correct SQL to register, claim, submit, and query the board.

**Acceptance criteria:**
- Every table documented with column descriptions, types, and example values
- Relationships between tables explained (wanted → completions → stamps)
- Sample queries for common operations

### US-3: Case Study Learning
As a developer evaluating whether to integrate, I can read a real case study of a non-Gas-Town system that completed the full MVGT flow, so that I understand the practical effort and value.

**Acceptance criteria:**
- Case study covers a real system (Emmanuel's agent flywheel)
- Shows actual Dolt commands run, actual output received
- Includes timing/effort estimates
- Honest about rough edges encountered

### US-4: Automation Reference
As a developer who wants to script Wasteland participation into their CI/CD or orchestrator, I can find automation patterns and API examples, so that my rig can participate programmatically.

**Acceptance criteria:**
- DoltHub API examples for fork and PR creation
- Shell script snippets for common operations
- Pattern for periodic sync with upstream

## 3. Architecture Changes

Not applicable — this is a documentation deliverable. No code changes to any codebase.

The guide itself will be structured as a single markdown file with these sections:
1. Introduction — what MVGT is, why it exists
2. Prerequisites — what you need before starting
3. Quick Start — minimal 10-step path from zero to first claimed item (register, browse, claim, submit). Copy-paste commands, no deep explanations. Links forward to full schema and lifecycle sections for details.
4. The Commons Schema — full reference for all 7 tables, including:
   - The sandbox model (sandbox_required, sandbox_scope, sandbox_min_tier on wanted items)
   - The chain integrity model (block_hash, prev_stamp_hash for tamper evidence)
   - The HOP federation pattern (hop_uri across rigs, completions, stamps, chain_meta)
5. MVGT Lifecycle — step-by-step walkthrough (detailed, covers edge cases)
6. Case Study — Emmanuel's agent flywheel
7. Automation Patterns — scripting participation
8. Troubleshooting — common issues and fixes
9. FAQ

## 4. Data Model Changes

Not applicable — no migrations. However, the guide must document the existing commons schema accurately:

### Important Defaults (must be documented in the guide)
The following defaults affect integrator behavior — INSERT statements that omit these columns get these values:

| Table.Column | Default | Implication |
|-------------|---------|-------------|
| rigs.trust_level | 0 | New rigs start untrusted — higher levels granted by validators |
| rigs.rig_type | 'human' | Override to 'agent' or 'ci' for automated rigs |
| wanted.status | 'open' | New items are immediately visible on the board |
| wanted.effort_level | 'medium' | Callers should set explicitly: small/medium/large/epic |
| wanted.priority | 2 | Range 1-3 observed in data; 2 is default |
| wanted.sandbox_required | 0 (false) | Set to 1 for items requiring sandboxed execution |
| stamps.confidence | 1.0 | Full confidence; lower values indicate uncertainty |
| stamps.severity | 'leaf' | Leaf stamps are the base level |

### Tables

| Table | Purpose | All Columns |
|-------|---------|------------|
| `rigs` | Registry of all participants | handle (PK), display_name, dolthub_org, hop_uri, owner_email, gt_version, trust_level, registered_at, last_seen, rig_type, parent_rig |
| `wanted` | The wanted board — open work items | id (PK), title, description, project, type, priority, tags, posted_by, claimed_by, status, effort_level, evidence_url, sandbox_required, sandbox_scope, sandbox_min_tier, created_at, updated_at |
| `completions` | Evidence of completed work | id (PK), wanted_id (FK→wanted), completed_by, evidence, validated_by, stamp_id, parent_completion_id, block_hash, hop_uri, completed_at, validated_at |
| `stamps` | Multi-dimensional attestations | id (PK), author, subject, valence (JSON), confidence, severity, context_id, context_type, skill_tags, message, prev_stamp_hash, block_hash, hop_uri, created_at |
| `badges` | Achievement-style awards | id (PK), rig_handle, badge_type, awarded_at, evidence |
| `chain_meta` | Federation chain metadata | chain_id (PK), chain_type, parent_chain_id, hop_uri, dolt_database, created_at |
| `_meta` | Commons metadata | key (PK), value — known keys: schema_version, wasteland_name, upstream_org, created_at |

### Key Relationships
```
# Core lifecycle chain
wanted.posted_by        → rigs.handle       (who posted the item)
wanted.claimed_by       → rigs.handle       (who claimed the item)
wanted.id               ← completions.wanted_id
completions.completed_by → rigs.handle      (who did the work)
completions.validated_by → rigs.handle      (who validated)
completions.stamp_id    → stamps.id         (reverse pointer to stamp)
stamps.context_id       → completions.id    (polymorphic, when context_type = 'completion')
stamps.author           → rigs.handle       (yearbook rule: author ≠ subject)
stamps.subject          → rigs.handle
badges.rig_handle       → rigs.handle

# Federation & provenance
completions.parent_completion_id → completions.id   (chained completions across forks)
chain_meta.parent_chain_id      → chain_meta.chain_id (chain hierarchy)

# Cross-cutting patterns
hop_uri — present on rigs, completions, stamps, chain_meta (HOP protocol federation URI)
block_hash / prev_stamp_hash — integrity chain (tamper-evident linking)
```

### Status Lifecycle
```
wanted.status: open → claimed → in_review → validated

  open        — Posted on the wanted board, available for claiming
  claimed     — A rig has claimed the item (claimed_by set)
  in_review   — Work submitted, completion record created, awaiting validation
  validated   — Validator reviewed, stamp issued (completions.validated_by + stamp_id set)
```

### Stamp Valence (multi-dimensional, open-ended)
Valence dimensions are NOT fixed — each stamp can use any combination of dimensions.
Common dimensions observed in practice: quality, reliability, speed, thoroughness.

Examples from actual stamps:
```json
// Minimal (2 dimensions)
{"quality": 0.72, "reliability": 0.68}

// Common (3 dimensions)
{"quality": 0.85, "reliability": 0.80, "speed": 0.90}

// With thoroughness
{"quality": 0.78, "reliability": 0.82, "thoroughness": 0.75}
```

## 5. Cross-Module Dependencies

Not applicable — no software modules. However, the guide depends on:
- Stable commons schema (currently v1.1 per `_meta`)
- DoltHub API stability for fork/PR operations
- Dolt CLI compatibility (tested on v1.83.1)

## 6. Security + Privacy Model

The guide must document:
- **DoltHub token handling** — tokens should never be committed to repos. Use env vars (`DOLTHUB_TOKEN`).
- **Credential management** — `dolt login` creates JWK files in `~/.dolt/creds/`. Document the `dolt creds use` flow for headless servers.
- **Credential revocation** — document how to revoke a compromised DoltHub credential (delete from DoltHub account settings, then `dolt creds rm <id>` locally).
- **`.dolt/` directory safety** — warn readers to add `.dolt/` to their `.gitignore` immediately after cloning the commons fork into any project directory.
- **CI/CD credential hygiene** — for ephemeral CI runners, recommend exporting `DOLT_CREDS_FILE` pointing to a secrets-managed path rather than relying on `~/.dolt/creds/`.
- **Fork permissions** — your fork is public by default on DoltHub. Don't put sensitive data in commons tables.
- **Data permanence warning** — Dolt retains full commit history. Even deleted rows are visible in `dolt log` and `dolt diff`. Any data written to the commons (including `rigs.owner_email`) is permanently public. Advise using a role/team email rather than personal email.
- **Concurrent claim races** — two rigs can claim the same wanted item simultaneously since each works on their own fork. Mitigation: always `dolt pull` upstream immediately before pushing a claim, check that `wanted.status` is still `'open'`. The PR merge acts as the serialization point.
- **Yearbook rule** — you can't stamp your own work. The guide should explain this security property.
- **Sandbox requirements** — some wanted items set `sandbox_required = 1` with a `sandbox_scope` and `sandbox_min_tier`. The guide should explain what these fields mean and how a rig should honor them (or skip items it cannot sandbox).
- **Chain integrity** — `block_hash` and `prev_stamp_hash` fields provide tamper-evident chaining. The guide should explain that integrators should not fabricate these values — they are computed by the validation process.

## 7. Performance Targets

Not applicable for the guide itself. However, the guide should document:
- `dolt clone` time expectations for the commons repo (~30s for current 208-chunk size)
- `dolt push` time expectations (~20s per commit)
- `dolt pull` for sync operations
- Note: Dolt is single-node, ~100GB practical limit — not a concern for commons

## 8. Testing Plan

### Guide Quality Testing
- **Technical accuracy**: Every Dolt command in the guide must be verified against an actual Dolt installation (v1.83.1+)
- **Completeness check**: Walk through the guide from scratch on a clean environment to verify nothing is missing
- **Schema accuracy**: Cross-reference every table/column description against the actual `DESCRIBE` output
- **Link validation**: All URLs (DoltHub, GitHub, docs) must resolve
- **Copy-paste verification**: Every code block must work when pasted directly into a terminal

### Review Process
- Self-review against the Great Plans checklist
- Critique rounds (4 rounds with Claude + Codex)
- Submit as PR to steveyegge/gastown for maintainer review

## 9. Migration / Backward-Compat Plan

Not applicable — documentation only. However:
- The guide should note the current schema version (1.1) and warn that the schema may evolve
- Reference `gt wl sync` and `dolt pull` as the mechanism for staying current
- Note Yegge's statement: "Using Dolt makes schema migration a dream"

## 10. Error Handling + Operational Plan

### Troubleshooting Section Coverage
The guide must document these common failure modes:
1. **`dolt login` retry loop** — headless servers can't open browsers. Solution: open URL manually, use `dolt creds use` to select the right key.
2. **`gt wl join` fork API error (HTTP 400)** — the join command's fork API call may fail on day-zero software. Solution: fork manually on DoltHub website, then clone/push.
3. **Permission denied on push** — fork doesn't exist on DoltHub yet. Solution: create fork first.
4. **Multiple credentials created** — each `dolt login` run creates a new key. Solution: `dolt creds ls` to see all, `dolt creds use <id>` to select the authorized one.
5. **Merge conflicts on sync** — if upstream commons has diverged. Solution: `dolt pull`, resolve conflicts at cell level.
6. **Stale wanted board after clone** — another rig may have already claimed an item. Solution: always `dolt pull` and re-check `wanted.status = 'open'` before writing a claim.
7. **Commits on wrong branch** — Dolt defaults to `main`. Verify `dolt branch` output before committing. Recovery: `dolt checkout main`.
8. **Schema version mismatch** — upstream commons may have migrated. Solution: check `SELECT value FROM _meta WHERE key = 'schema_version'` after each `dolt pull`.

### Delivery Plan
1. Write guide in local repo
2. Clone steveyegge/gastown
3. Add guide as `docs/guides/mvgt-integration.md`
4. Create PR with descriptive title and summary
5. Submit completion evidence to Wasteland commons (link to PR)

## 11. Risk Register

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Commons schema changes before guide is published | Medium | Medium | Pin to schema v1.1, add version note |
| Wasteland onboarding flow changes rapidly | High | Low | Focus on Dolt primitives, not UI/tooling |
| Gas Town maintainers have different vision for MVGT guide | Low | Medium | Keep guide factual, avoid opinionated architecture advice |
| DoltHub API undocumented or unstable | Medium | Low | Document CLI-first approach, API as optional enhancement |
| Our case study is too specific to one orchestrator | Low | Medium | Abstract patterns, keep implementation details in a clearly labeled section |

## 12. Task Breakdown

### T1: Clone gastown repo and identify guide location (P0, 15 min)
- Clone `steveyegge/gastown` to a working directory
- Identify where docs live (likely `docs/` or `docs/guides/`)
- Check for existing integration guides or templates
- **Acceptance**: Working clone, confirmed target path for the guide

### T2: Write guide — Introduction, Prerequisites, and Quick Start (P1, 40 min)
- What MVGT is and why it exists (3 paragraphs)
- Prerequisites section with:
  - Dolt CLI >= v1.83.1 — include install commands for Linux (`curl | bash`), macOS (`brew install dolt`), link to Windows docs
  - DoltHub account — link to signup page, note free account is sufficient
  - `dolt login` completed — link forward to Troubleshooting for headless server workaround
  - Agent framework (any) — clarify this means "you already have something that does work"
  - Basic SQL + Git familiarity
- Quick Start: minimal 10-step path from zero to first claimed item. Copy-paste commands only, link forward to detailed sections.
- Tone: practical, developer-focused, no marketing
- **Acceptance**: Clear intro, complete prereqs list with version-pinned install commands for 2+ platforms, quick start achievable in ~30 minutes

### T3a: Write guide — Commons Schema Table Reference (P1, 45 min)
- **First**: validate plan's schema documentation against live commons database — run `DESCRIBE` for all 7 tables, compare output against plan Section 4, update to match reality if discrepancies found
- Document all 7 tables with column-by-column reference
- Consistent format per table: `| Column | Type | Nullable | Default | Description |`
- 78 columns total across 7 tables — each gets a one-line description with example value
- **Acceptance**: Every column in every table has a row in the reference. Types match DESCRIBE output exactly. At least one example value per column.

### T3b: Write guide — Schema Relationships, Patterns, and ERD (P1, 30 min)
- Depends on: T3a
- Entity relationship diagram in Mermaid showing all 7 tables and FK/logical relationships
- Key relationships section (wanted→completions→stamps lifecycle chain, rigs as central reference)
- Status lifecycle diagram for `wanted.status` (open → claimed → in_review → validated)
- Stamp valence structure explanation with 3 real JSON examples
- Cross-cutting patterns: hop_uri federation, block_hash/prev_stamp_hash integrity chain, sandbox model
- **Acceptance**: ERD renders correctly in GitHub markdown. All relationships represented. No orphan tables.

### T4: Write guide — MVGT Lifecycle Walkthrough (P1, 45 min)
- Depends on: T3a
- Step-by-step: install → auth → fork → clone → register → browse → claim → work → submit evidence → push → create PR
- Copy-paste Dolt CLI commands for each step
- Expected output for each command
- After each mutation step (register, claim, submit), include a verification query to confirm success
- Include `dolt diff` after each `dolt commit` so developer sees what changed
- Explicitly note which steps require `dolt add .` and `dolt commit` (Dolt working set vs committed state is a common confusion)
- **Acceptance**: A developer can follow steps from scratch and complete the lifecycle. Every mutation has a verification query. Every commit shows the preceding add+diff sequence.

### T5: Write guide — Case Study: Emmanuel's Agent Flywheel (P1, 30 min)
- Depends on: T3a
- What the flywheel is (brief architecture)
- How it maps to Wasteland concepts (rig, wanted board ↔ beads, validators ↔ DoD gates)
- Reconstruct actual Dolt commands from the commons fork history (`dolt log` on jorisdevreede/wl-commons)
- Include at minimum: the `INSERT INTO rigs` statement, the `SELECT` that browsed the wanted board, the `UPDATE wanted SET claimed_by`, and the DoltHub PR URL
- Timing and rough edges encountered
- **Acceptance**: Honest, specific. At least 3 copy-pasted terminal blocks showing actual command + output pairs.

### T6a: Write guide — Automation Patterns (P2, 20 min)
- Depends on: T4
- Shell script snippets for CI/CD: fork, clone, register, claim, submit, push, create PR
- DoltHub API examples: `curl` examples with `DOLTHUB_TOKEN` header for fork and PR creation
- Periodic sync pattern: cron-style `dolt pull` + conflict detection script
- **Acceptance**: Every shell snippet runs without modification (given valid credentials). API examples include expected HTTP status codes.

### T6b: Write guide — Troubleshooting (P1, 20 min)
- Depends on: T4
- Document all 8 failure modes from plan Section 10
- Each follows consistent format: **Symptom** (what developer sees), **Cause** (why), **Solution** (exact commands)
- Include diagnostic commands: `dolt status`, `dolt branch`, `dolt log --oneline -n 5`, `SELECT * FROM _meta`
- **Acceptance**: Each entry has symptom/cause/solution format. Diagnostic commands are copy-paste ready.

### T6c: Write guide — FAQ (P2, 15 min)
- Depends on: T2, T3b, T4, T5, T6a, T6b
- 5-8 common questions: "Do I need Gas Town?", "How do I get stamps?", "What if the schema changes?", "Can multiple rigs share a fork?", "How does trust_level increase?", "What if I disagree with a stamp?"
- Each answer: 2-4 sentences with a link to the relevant guide section
- **Acceptance**: Every FAQ answer links to at least one other section. No answer requires external knowledge beyond the guide.

### T7: Review, polish, and submit PR (P1, 30 min)
- Depends on: all above
- Self-review checklist:
  1. Run every `dolt sql` command against an actual commons checkout — verify no SQL errors
  2. Run every non-destructive shell command and confirm output matches
  3. Validate all URLs with `curl -sI <url> | head -1` — must return 200 or 301
  4. Render markdown in GitHub preview — verify Mermaid ERD, tables, and code blocks render
  5. Verify no Gas Town CLI commands appear in "you don't need Gas Town" sections without noting they're Gas Town commands shown for comparison
- Create branch in gastown clone, commit guide
- Push and create PR — description must include schema version, Dolt version, and case study link
- Update Wasteland commons: submit completion evidence for w-com-004
- **Acceptance**: PR created on steveyegge/gastown, completion evidence pushed to fork

### Dependencies
```
T1 (clone repo)
├── T2  (intro + prereqs + quick start) — no deps beyond T1
├── T3a (schema tables)   — no deps beyond T1
│   └── T3b (ERD + patterns) — depends on T3a
├── T4  (lifecycle walkthrough) — depends on T3a
├── T5  (case study) — depends on T3a
├── T6a (automation) — depends on T4
├── T6b (troubleshooting) — depends on T4
└── T6c (FAQ) — depends on T2, T3b, T4, T5, T6a, T6b
    └── T7 (review + PR) — depends on all above
```

Parallel tracks after T1:
- Track A: T2 (intro)
- Track B: T3a → T3b (schema)
- Track C: T4 → T6a, T6b (lifecycle → automation, troubleshooting)
- Track D: T5 (case study, after T3a)
- Final: T6c (FAQ) → T7 (review + PR)

Total estimated effort: ~4.5 hours sequential, ~3 hours with 2 parallel agents (medium effort, matches wanted item's effort_level)
