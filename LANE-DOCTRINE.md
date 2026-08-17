# Estate Lane Doctrine — Pharoah First, Live Data Only

**Owner directive (Phil, 17 Aug 2026).** Applies to every Claude lane working this estate: the hourly coordinator, every worker lane it spawns, the overseer, and any local Claude Code session doing estate work.

## Command structure

| Role | Where | Job |
|---|---|---|
| **Owner** | Phil | Sets direction. Only he approves spend or scope changes. |
| **Overseer** | session `claude/lane-live-data-updates-jkb448` (this repo) | Audits hourly, keeps this doctrine + `LOCAL-CLAUDE-PASTE.md` current, reports blockers to Phil. |
| **Coordinator** | "Pharoah product estate sweep" session, woken hourly by trigger `pharoah-estate-completion-autopilot` (:20 UTC) | Drives the queue, supervises lanes via `lane_handoffs` + poke triggers, maintains the LEDGER row. |
| **Worker lanes** | `pharoah-lane-*` sessions | Execute queue items, post evidence to the bus. |
| **Bus** | `lane_handoffs` table, Supabase project `ofcjggvruhwhaqthdwxa` | The only channel of record between lanes. |

## Priority

**Pharoah venture to 100% first** — the register slice `cluster = 'pharoah'` plus Pharoah-owned rows (costa, forges, property/sourcing, pharoah-b2b-intelligence). AMAYA/attoh work happens only when the entire Pharoah queue is blocked, never at its expense. GhostWire is never touched by estate lanes.

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
