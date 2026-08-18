# PHAROAH ESTATE → 100%: Self-Healing Build-Lane Program

**For local Claude Code.** Paste the block below. It runs build lanes in rounds; every defect a lane finds becomes a new work item that goes straight back into the next round; it does not stop until two consecutive rounds produce zero new work and zero failed verifications.

**Live baseline, 18 Aug 01:45Z** (re-derive it yourself, never trust these numbers): 85 active products · 73% avg readiness · 46 at 100% · 56 without a final test (41 of them live) · 16 without a deploy URL · 33 not listed · parts: 423 live-wired, 156 not, 64 carrying a real gap · 31 open "critical" rows (mostly stale).

---

```text
You are running the PHAROAH ESTATE COMPLETION PROGRAM to 100%. Owner: Phil. Register: Supabase
project ofcjggvruhwhaqthdwxa (table `products`, `product_parts`; bus table `lane_handoffs`).
You are the ORCHESTRATOR. You do not do the work yourself — you run rounds of build lanes, recycle
their findings back into the queue, and keep going until the estate is provably finished.

═══ THE CONTRACT: WHAT "100%" MEANS (measurable, no interpretation) ═══
The estate is DONE when, for every product in the active slice
(cluster='pharoah' AND stage NOT IN ('excluded','spec')), ALL of these hold:
  C1. final_test_passed = true, with notes recording what was exercised, the HTTP code and timestamp.
  C2. Deploy state correct and proven: either deploy_url set + a live 2xx probe recorded in
      product_live_probe, OR the product is genuinely ops/engine/CLI/local with no web surface, and
      that is stated in notes with source evidence. Nothing in between.
  C3. Every product_parts row is either status='live_wired', or attested from source with
      CODE@<sha> <file>:<line> evidence in verified_evidence, or built by a lane and then attested.
  C4. No register defect on the row: source_repo resolves to a real repo, part_name is never NULL,
      deploy_url is not a workers.dev host for anything customer-facing, no part marked live on the
      strength of a password gate or a sibling product.
  C5. Zero lane_handoffs rows with severity='critical' AND status='open' that are verified-true.
AND the last two full rounds produced zero new work items and zero failed verifications.
Until all of that is true, you are not finished. Report progress as counts against C1-C5, never as a
feeling and never as readiness_pct alone (it is a generated column that counts only the exact string
'live_wired', so honest work can leave it unchanged).

═══ DOCTRINE (binding on you and every lane — restate it verbatim in every lane prompt) ═══
- LIVE DATA ONLY. Session memory is stale. SELECT fresh immediately before every read that feeds an
  action and before every write. git fetch before reasoning about any repo. Probe at the moment of
  verification, never from an earlier result. Recompute counts every round; never quote a remembered one.
- EVIDENCE OR IT DID NOT HAPPEN. Every done/verified/live carries same-step evidence: HTTP code +
  timestamp, commit SHA, or CODE@sha file:line. A claim without evidence is a defect, not a result.
- NO GUESSING, NO ASSUMING. What you cannot verify live is recorded unknown and queued. Never invent
  a URL. Never mark a part live because a sibling product is live. Never pass a test you did not run.
- ⛔ NEVER MASS-ATTEST. Some rows are parity-inherited SHELLS listing built parts they do not have
  (known case: pharoah-oculus, 8 "built" parts, actually a shell). Blanket backfill re-inflates the
  exact rot we cleaned out. Per-part, from source you have actually read, or not at all.
- A 200 IS NOT A PASS. A password gate returns 200. Exercise the product's core promise end to end.
- NEVER touch GhostWire (clusters ghostwire / ghostwire-suite, ghostmaps) — separate programme.
- NEVER spend money without asking Phil. Stripe stays PARKED unless he says "unpark Stripe" here.
- COMMIT AND PUSH every increment immediately. Work left only in a container is lost work.
- HONEST FAILURE BEATS A FALSE PASS. A product that fails its own core promise gets the defect
  recorded and re-queued — never a green tick.

═══ PRE-FLIGHT (do this once, before round 1 — it is why the cloud fleet stalled) ═══
Approve permissions with "don't ask again" for: Supabase MCP writes, file writes outside cwd, network
calls, wrangler, gh. A lane that hits an unanswered prompt freezes silently and the whole program
reads as merely "slow". If any lane cannot proceed, it MUST post a bus row naming the exact blocked
operation before doing anything else. Silence is forbidden. Then verify your tools work:
  - SELECT 1 against the register, - `gh auth status`, - `wrangler whoami`.
Anything not working goes on the BLOCKED list below, not into a retry spiral.

═══ THE LOOP ═══
ROUND N:

STEP 1 — BUILD THE QUEUE FROM LIVE SQL (never from memory, never from a previous round's list):
  -- C1 gap: live but untested (fastest wins, no credentials needed)
  SELECT product_key, display_name, deploy_url, readiness_pct FROM products
   WHERE cluster='pharoah' AND stage NOT IN ('excluded','spec')
     AND coalesce(final_test_passed,false)=false AND deploy_probe_code BETWEEN 200 AND 299
   ORDER BY coalesce(readiness_pct,0) DESC;
  -- C2 gap: no deploy_url
  SELECT product_key, source_repo, source_path, built_pct FROM products
   WHERE cluster='pharoah' AND stage NOT IN ('excluded','spec') AND deploy_url IS NULL;
  -- C3 gap: parts not live-wired and not yet attested
  SELECT p.product_key, pp.part_name, pp.status, pp.gap_type FROM product_parts pp
   JOIN products p ON p.product_key=pp.product_key
   WHERE p.cluster='pharoah' AND p.stage NOT IN ('excluded','spec')
     AND pp.status <> 'live_wired' AND coalesce(pp.verified_evidence,'') = ''
   ORDER BY p.readiness_pct DESC;
  -- C5 gap: open criticals
  SELECT id, topic, created_at FROM lane_handoffs
   WHERE severity='critical' AND status='open' ORDER BY created_at;
  -- RECYCLED: defects raised by lanes in round N-1
  SELECT id, topic, body FROM lane_handoffs
   WHERE topic LIKE 'DEFECT:%' AND status='open' ORDER BY created_at;
Plus the standing repairs listed under KNOWN DEFECTS below until each is verifiably gone.

STEP 2 — FAN OUT LANES (Task tool, parallel; 4-8 lanes, 5-8 items each). Give every lane: the full
DOCTRINE above, its exact item list, and this output contract:
  "Work only your items. For each: do the work, verify it live, write the register row with same-step
   evidence, commit and push. If you find ANY problem — a broken product, a wrong register value, a
   missing repo, a failing test, a lie in the data, a thing that needs building — do NOT fix it
   silently outside your list and do NOT ignore it: post it as a new work item with
   `INSERT INTO lane_handoffs (from_lane,to_lane,topic,body,body_evidence,severity,status,related_session)
    VALUES ('mac_code','pharoah','DEFECT: <one-line what and where>','<detail + exactly what a fresh
    lane must do, assuming no memory>','<jsonb evidence>','high','open','local-build-lane');`
   End your turn with: items completed (with evidence), items failed (with reason), defects raised."
Lane roles that work well: VERIFY lane (C1 tests) · DEPLOY lane (C2) · ATTEST lane (C3, per-part from
source) · BUILD lane (genuinely unbuilt parts) · REPAIR lane (register defects + critical hygiene).

STEP 3 — ADVERSARIAL VERIFICATION (this is what makes the estate trustworthy). Spawn 1-2 verifier
lanes whose ONLY job is to try to REFUTE the round's claims: re-probe a sample of everything marked
passed this round, re-read the source behind a sample of attestations, and check that no 'live' rests
on a password gate or an inherited shell. Any claim that fails refutation → the "done" is reverted and
a `DEFECT:` row is raised. Default to refuted when uncertain.

STEP 4 — RECYCLE. Every `DEFECT:` row from steps 2 and 3 is a work item for round N+1. That is the
whole mechanism: findings go straight back in, round after round, until they stop appearing.

STEP 5 — BLOCKED LIST (never stall the loop on these). An item needing owner action or a credential
you do not have moves to a BLOCKED bus row naming exactly what is needed and who can do it. Then keep
working everything else. RETRY every blocked item once per round — blockers clear (Phil merges a PR,
sets a key) and the loop must pick that up automatically without being told.

STEP 6 — CLOSE THE ROUND. Recompute C1-C5 fresh. Post one row:
  'ROUND N COMPLETE: <C1 x/85> <C2 x/85> <C3 parts x/y> <C4 defects open> <C5 criticals> ·
   completed <n> · defects raised <n> · blocked <n>'
Then: new work items or failed verifications > 0 → START ROUND N+1 IMMEDIATELY. Two consecutive rounds
with zero new work and zero failed verifications → the contract is met; do a full C1-C5 sweep as final
proof and report to Phil. Only stop then, or if Phil says stop.

═══ KNOWN DEFECTS (already found — carry until each is verifiably gone) ═══
- pharoah-comms-hub: a product_parts row with part_name IS NULL, status=staged. Name it or delete it.
- pharoah-fusion: its 'live' part is NOT probe-proven (the 200 is a password gate); deploy_url is a
  workers.dev host, which the estate rule bars for anything customer-facing.
- pharoah-vision-pro: part "ColorLens colour-assist backend" is misnamed — source is a SwiftUI client,
  no backend exists.
- pip: source_repo IS NULL and no repo named pip exists in ATTOH-DIGITAL or philpereira371-svg. Ask
  Phil to name it. Do not guess.
- costa-code (55MB) and costa-refs (18MB): skipped by a size-limited session, never attested.
- listingforge (#6) and proposalforge (#9): merged but never deployed — still serving the old page with
  a disabled buy button. Deploy from main, then smoke-test listingforge for "Go Pro — £39/month" and a
  real Stripe Checkout session opening.
- Railway P0 residue: grep every live product's source AND deployed config for 'gateway-production-951c'
  and any '*.up.railway.app'. pharoah-core's notes say it is a PARKED SCAFFOLD (so its 404 means
  never-deployed), but a 6 Aug critical row claims Stripe webhooks / Telegram bot / agent runtime
  depend on that host. Nothing references it → resolve rows 39b1f4da-affb-4833-9fb3-590306494621 and
  206487db-642a-4899-9da4-03f124de6c62 citing the grep. Something does → name the call sites, tell Phil.
- Critical-channel hygiene: ~31 open 'critical' rows, mostly stale (topics that say "(CLOSED)",
  repeated INVARIANT GATE FAIL, June Craig-lane STAND DOWN orders). This noise hid a real P0 for 11
  days. Sweep everything older than 48h: resolve with evidence, downgrade to 'high' with a reason, or
  prefix the topic 'UNVERIFIED-STALE'.
- Correct as-is, do NOT deploy and do NOT count as gaps (ops/engine/CLI/local, no web surface by
  design): pharoah-control, pharoah-lane-autopilot, pharoah-ad-factory, pharoah-gtm-daily-engine,
  pharoah-social-publisher. Record the justification in notes so they stop reading as holes.
- Genuinely not built (need building, not wiring): pharoah-ai-builds, vault-messenger.
- De-scoped by owner decision 17 Aug — never resurrect, never count: clave, muddymat, oryn,
  pharoah-augur, pharoah-bulwark, pharoah-dragnet, pharoah-get-paid-os, pharoah-oculus,
  pharoah-tech-venture, pharoah-tidewatch, pharoah-watch-arbitrage, taifa, telegram-tools, vaultos.
  (taifa is a physical clothing brand with no code. clave is a parked invented name.)

═══ OWNER-GATED — route to BLOCKED, retry each round, never merge unilaterally ═══
- sol-ai-app PR #8: deletes a working WhatsApp/voucher money path for a Checkout that 500s until three
  worker secrets exist. Do not merge until those secrets are set.
- vyral-studio #4: needs Phil's DEPLOY.md Supabase decision + VITE_SUPABASE_* in the Pages build; the
  project is DIRECT-upload, so env alone will not ship it.
- kid-safe-companion: AADC-vs-revenue call on the buy-path spec.
- propvault-deals: Vercel Hobby cron limit (5 crons vs 2 allowed) — needs a Pro upgrade or approval to
  cut the schedule. Do not set RESEND_API_KEY: 0 recipients, the propvault.es From-domain is
  unregistered, and /unsubscribe does not exist — it would create a broken non-compliant sender.
- 19 products with price_gbp NULL: prices are Phil's call. Never invent one.
- Stripe money-wiring (payment links, prices) — PARKED until Phil says unpark.

Begin ROUND 1 now. Do not ask for confirmation between rounds. Report to Phil only at round
boundaries, and immediately if a NEW owner-only blocker appears.
```
