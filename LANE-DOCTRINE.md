# Estate Lane Doctrine — Pharoah First, Live Data Only

**Owner directive (Phil, 17 Aug 2026).** Applies to every Claude lane working this estate: the hourly coordinator, every worker lane it spawns, the overseer, and any local Claude Code session doing estate work.

## Command structure

| Role | Where | Job |
|---|---|---|
| **Owner** | Phil | Sets direction. Only he approves spend or scope changes. |
| **Overseer** | session `claude/lane-live-data-updates-jkb448` (this repo) | Audits hourly, keeps this doctrine + `LOCAL-CLAUDE-PASTE.md` current, reports blockers to Phil. |
| **Coordinator** | "Pharoah product estate sweep" session, woken hourly by trigger `pharoah-estate-autopilot-v2` (:20 UTC) | Drives the queue, supervises lanes via `lane_handoffs` + poke triggers, maintains the LEDGER row. |
| **Worker lanes** | `pharoah-lane-*` sessions | Execute queue items, post evidence to the bus. |
| **Bus** | `lane_handoffs` table, Supabase project `ofcjggvruhwhaqthdwxa` | The only channel of record between lanes. |

## Priority

**Pharoah venture to 100% first** — the register slice `cluster = 'pharoah'` plus Pharoah-owned rows (costa, forges, property/sourcing, pharoah-b2b-intelligence). AMAYA/attoh work happens only when the entire Pharoah queue is blocked, never at its expense. GhostWire is never touched by estate lanes.

### The active denominator (owner decision, 17 Aug 2026)

Estate progress is measured over **`cluster = 'pharoah' AND stage NOT IN ('excluded','spec')`** — 85 active products as of 13:32Z. Phil de-scoped 14 rows that had no `source_repo`, no `source_path` and no `deploy_url`: nothing existed to wire, and counting them made 100% mathematically unreachable. Rows are preserved with a dated note and revive by setting `stage` back and naming a source repo.

Lanes must not touch de-scoped rows, must not reintroduce them into counts, and must **flag** (never guess at) any future row that is empty on all three fields.

### Order of attack

Work the piles in this order — biggest unblocked first, never trickle across all three:

1. **Live but untested** (40 at 13:32Z) — already serving 2xx, only missing `final_test_passed`. No owner, no deploy, no credentials. Fan out wide; this is always the fastest available movement.
2. **Built but undeployed** (15 built ≥80% with no `deploy_url`) — one deploy each. Prepare them deploy-ready; never attempt credentialed or paid deploys. Connecting Cloudflare Workers Builds converts this pile from manual to automatic and is the highest-leverage owner action on the estate.
3. **Everything else** — partial builds, register defects, listing prep.

**Products with no web surface by design** (ops, engine, CLI, local scripts) are not URL gaps. A NULL `deploy_url` is correct for them and they must never be counted as missing wiring or have a URL invented. Measure them by source attestation instead.

**Attestation is per-part with `CODE@sha file:line` evidence — never mass-automated.** Some rows are parity-inherited shells that show built parts they do not have; blanket backfill re-inflates them. Attest what you have read in source, nothing more.

**A blocked lane must say so, within one wake.** If an operation cannot proceed — permission prompt, missing credential, refused tool — the lane posts a bus row naming the exact blocked operation before doing anything else. Silence is forbidden: on 17 Aug all five lanes stalled on permission prompts for ~3 hours while the queue read as merely "not moving", and re-issuing orders could never have fixed it. The overseer treats two hours of lane silence as blocked and escalates. Operations with no route-around become owner config items, not repeated retries.

**`critical` severity means verified-true-right-now and blocking.** Anything else is `high`. Stale critical rows hide real P0s — a real one sat unseen for 11 days behind ~27 stale rows.

Report movement as counts of `final_test_passed` and verified `product_parts`, not `readiness_pct` alone.

## Live-data rules

Session memory is stale the moment a sibling lane pushes, probes, or writes. Therefore:

1. **Git**: `git fetch origin` and read origin state before reasoning about any repo's contents; pull/rebase before push.
2. **Register**: `SELECT` the current rows immediately before every read that feeds an action, and again immediately before every `UPDATE`/`INSERT`. Never act on remembered row states.
3. **Probes**: verify URLs/deploys live at the moment of the claim. An earlier probe result is history, not evidence.
4. **Evidence**: every `done`/`verified`/`live` written to the register or the bus carries same-step evidence — origin commit SHA, HTTP code + timestamp, run URL.
5. **Counts**: always recomputed, never quoted from memory. (This file's own numbers are reference snapshots, timestamped, and yield to a fresh query.)

**No guessing. No assuming.** A fact that cannot be verified live is recorded as `unknown` and queued for verification — it is never asserted, and work is never marked complete on top of it.

## Blockers

Anything a lane cannot complete remotely (billing, credentials, platform access, connector auth) goes on the bus as a `severity = 'critical'` row and into the coordinator's LEDGER row. The overseer renders those into Phil's report and `LOCAL-CLAUDE-PASTE.md` so local Claude Code can execute the remainder. Blocked ≠ parked: the moment the blocker clears, the item re-enters the queue automatically.
