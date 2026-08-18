# Pharoah Estate — Cloud Fleet Stood Down, Handover to Local Claude Code

**18 Aug 2026, 01:35Z.** Owner order: too many blockers need a human and there is too much prompting — the cloud lanes are closed and the remainder moves local. All eight recurring triggers are **disabled** (autopilot v2, five poke-* lane triggers, the GhostWire website tracker, the PR #894 check-in). Nothing will prompt you again. Lanes were told to finish the unit of work in hand, push everything, post a `FINAL STATE:` row, and stop.

## Where the estate actually stands

**Active estate: 85 products** (`cluster='pharoah' AND stage NOT IN ('excluded','spec')`) — **73% avg readiness, 44 at 100%.** 56 lack a final test (41 of those are live and serving 2xx), 16 have no deploy URL.

Measured properly, the estate is **~95% built and under-recorded**, not half-built. Of the parts not marked `live_wired`: 115 built-but-unattested, 97 spec names the prober physically cannot HTTP-probe (it was probing `.py` and config files), 46 with no evidence line, and only **14 parts across 11 products genuinely unbuilt**. `readiness_pct` is a generated column that counts only the exact string `live_wired`, so every real-but-unattested part reads as zero.

**Done today, verified:** service_role exposure closed (legacy keys disabled 13:26:13Z, exposed JWT 200→401, all 24 pg_cron jobs pre-migrated) · 74 parts attested from source with `CODE@sha file:line` evidence · 14 empty rows de-scoped on your decision · sol-ai + pharoah-access deployed and probe-proven live (both were **broken as merged** and fixed before deploy) · hollow-twin inflation found and corrected *downward* · waitlist mis-route self-reported with a fix in PR #894.

## Part 1 — Things only you can do (~15 minutes, in this order)

1. **Merge PR #148 (safety, do this first).** Your daughter's device serial `93LAY0CJ0K` (3a) is **not** in `protected-serials.sh` on main. A lane flagged it critical at 00:30Z and it is still open.
2. **Merge Attohdigital PR #894.** A lane's earlier PR #891 mis-routed waitlist product keys on ten of your products (augur, bulwark, dragnet, tidewatch, oculus, get-paid-os, app-builder, stocklane, pocket-business-os). Any signup between 12:51 and now landed under the wrong key. #894 restores every key and app-builder's "Live now" badge. **Also:** check `free_tool_grant` / waitlist tables for that window and re-key anything found.
3. **Rotate the LiteLLM gateway master key** — plaintext in `brg-mac-desktop-intake` (`_ops/litellm-gateway/.env.example`, `test-routing.sh`, `AI-PROXY-GATEWAY-PACKET-2026-05-18.md`, `CREDENTIAL-BATCH-BINDING-PACKET-*`). Authenticated live when found **24 June** — 8 weeks open, your spend exposure. Rotate, then purge the values from those files.
4. **Connect Cloudflare Workers Builds** (dashboard; GitHub App already installed, id 132149322 — no estate token can authorise `builds/repos`). This is the structural fix: merges start deploying themselves, and "merged but not live" stops recurring permanently.
5. **Set permissions once so lanes never silently stall again** — approve with "don't ask again", or pre-allow via `/permissions`, in any session you run: **Supabase MCP writes, out-of-cwd file writes, and network/wrangler calls.** On 17 Aug all five lanes sat frozen on unanswered permission prompts for ~3 hours; that, not the workload, is why this felt endless.

**Decisions the fleet will not make for you:** vyral-studio #4 (DEPLOY.md Supabase choice + `VITE_SUPABASE_*` in the Pages build; note the project is DIRECT-upload so env alone won't ship it) · kid-safe-companion AADC-vs-revenue call on its buy-path spec · propvault-deals (Vercel Pro upgrade **or** approval to cut 5 crons to 2 — production is ~37 days stale) · **`pip`'s real repo** — no repo named `pip` exists in ATTOH-DIGITAL or philpereira371-svg, only you know where it lives · prices for 19 products with `price_gbp` NULL · `theresonance.cloud` needs an IONOS NS change · sol-ai-app **PR #8 is deliberately owner-gated** — it deletes a working WhatsApp/voucher money path for a Checkout that 500s until three worker secrets exist; do not merge it until those secrets are set. **Stripe remains PARKED** until you say unpark.

## Part 2 — The paste for local Claude Code

Run on a machine with `gh`/git authenticated, `wrangler` logged into the Cloudflare account, and Supabase access to `ofcjggvruhwhaqthdwxa`.

```text
You are finishing the Pharoah estate. Register: Supabase project ofcjggvruhwhaqthdwxa. Owner: Phil,
present with you. The cloud fleet has been STOOD DOWN — you are now the only worker, so nothing else
is racing you and nothing will overwrite your work.

DOCTRINE (binding, this is why the estate is trustworthy — do not relax it):
- LIVE DATA ONLY. Session memory is stale. SELECT fresh immediately before every read that feeds an
  action and before every write. git fetch before reasoning about any repo. Probe URLs at the moment
  of verification, never from an earlier result.
- EVIDENCE OR IT DIDN'T HAPPEN. Every done/verified/live you record carries same-step evidence:
  HTTP code + timestamp, commit SHA, or CODE@sha file:line. No exceptions.
- NO GUESSING, NO ASSUMING. Anything you cannot verify live is recorded as unknown and queued, never
  asserted. Never invent a URL. Never mark a part live because a sibling product is live.
- NEVER touch GhostWire (clusters ghostwire / ghostwire-suite, ghostmaps) — separate programme.
- NEVER spend money without asking Phil. Stripe work is PARKED unless he says "unpark Stripe".
- The active estate is: cluster='pharoah' AND stage NOT IN ('excluded','spec') — 85 products. 14 empty
  rows were de-scoped by owner decision on 17 Aug; do not resurrect them or count them.
- ⛔ DO NOT MASS-AUTOMATE ATTESTATION. Some rows are parity-inherited SHELLS that list built parts
  they do not have (pharoah-oculus is the known case: 8 "built" parts, actually a shell). Blanket
  backfill re-inflates exactly what we just cleaned up. Attest per-part, from source you have read.

START HERE — read the fleet's own handover rows first, they contain resume state you must not
re-derive:
  SELECT created_at, topic, body, body_evidence FROM lane_handoffs
  WHERE created_at > '2026-08-17T20:00:00Z'
    AND (topic LIKE 'FINAL STATE:%' OR topic LIKE 'STAND DOWN%' OR topic LIKE 'LEDGER%'
         OR topic LIKE 'RESUME STATE%' OR topic LIKE 'CHECKPOINT%')
  ORDER BY created_at DESC;

TASK 1 — THE BIG UNBLOCKED WIN: 41 live products missing a final test.
These are already serving 2xx. No deploys, no credentials, no owner input needed — this is the
single fastest movement available and the cloud fleet never managed it (it was frozen on permission
prompts). Pull them live:
  SELECT product_key, display_name, deploy_url, readiness_pct FROM products
  WHERE cluster='pharoah' AND stage NOT IN ('excluded','spec')
    AND coalesce(final_test_passed,false)=false AND deploy_probe_code BETWEEN 200 AND 299
  ORDER BY coalesce(readiness_pct,0) DESC;
As of 01:35Z that is: sol-ai-os, wa-invoice-recovery, pharoah-kids-world, wa-company-report, wa-clinic,
sentinel-pro, sol-ai-app, forge-hub, pharaoh-vision-site, wa-assistant, resonance, pharoah-suite, aegis,
pharoah-access, wa-property-concierge, glasses-recruiter, glasses-costa, rizz-ai, pharoah-surgicalsight,
wa-recruit-screener, fieldassist, malaga-alerta, glasses-visionplus, solfuego, pharaoh-vision-glasses,
pharoah-security-core, costashop, pharoah-wire, spanish-property-os, glasses-estate, propvault-deals,
pharoah-website, sol-ai-gateway, pharoah-whatsapp-order-menu, sol-ai, propvault-espana, pharoah-stocklane,
vyral-studio, glasses-sentinel, gym-health-glasses, pharoah-fusion.
For each: probe the URL live NOW; run whatever tests the repo has; exercise the product's ONE core
promise end-to-end (not just a 200 — a 200 can be a password gate: pharoah-fusion's "live" status is
exactly that mistake). Then:
  UPDATE products SET final_test_passed=true, final_test_at=now(), updated_at=now(),
    notes = coalesce(notes,'') || E'\n[<date> local] FINAL TEST: <what you exercised> -> <result>, HTTP <code> at <ts>'
  WHERE product_key=$1;
If a product fails its own core promise, do NOT pass it — record the defect in notes, set stage
appropriately, and move on. A failed test recorded honestly is worth more than a false pass.

TASK 2 — 16 products with no deploy_url. Most should NEVER have one; do not wire them.
  SELECT product_key, source_repo, source_path, built_pct FROM products
  WHERE cluster='pharoah' AND stage NOT IN ('excluded','spec') AND deploy_url IS NULL;
Classification already established — respect it:
  CORRECT AS-IS (ops/engine/CLI/local, no web surface by design — a NULL deploy_url is RIGHT):
    pharoah-control, pharoah-lane-autopilot, pharoah-ad-factory, pharoah-gtm-daily-engine,
    pharoah-social-publisher (a local python script under ~/Library/Application Support/PharoahOutreach).
    Do not deploy these. Mark them measured-by-attestation so they stop reading as gaps.
  REAL DEPLOY CANDIDATES (built, have repos): costa-alerta, costa-code, costa-refs, pharoah-comms-hub,
    pharoah-vision-pro, sol-ai-os, retailbot (source pharoah-core), pharoah-app-builder + 
    pharoah-pocket-business-os (both source brg-operations), pharoah-tech.
    For each: fetch the repo fresh, find wrangler/Pages config, deploy, PROBE, then register the URL
    with same-step evidence and INSERT the probe into product_live_probe. workers.dev is acceptable as
    first wiring ONLY for internal use — the estate rule bars workers.dev in customer-facing content,
    so anything customer-facing needs a branded domain.
  NOT BUILT: pharoah-ai-builds, vault-messenger (both 0% built — they need building, not wiring).

TASK 3 — publish what is already merged but not live.
listingforge (#6 merged, tests 8/8) still serves the OLD page with a disabled buy button; proposalforge
(#9 merged) likewise. wrangler deploy each from main, then smoke-test: listingforge must show
"Go Pro — £39/month" and the button must open a real Stripe Checkout session. This is the shortest
path to revenue on the estate.

TASK 4 — settle two things the cloud fleet left open.
(a) P0 RESIDUE: grep every live product's source AND deployed worker config/env for
    'gateway-production-951c' and any '*.up.railway.app'. Context: pharoah-core's register notes say it
    is a PARKED SCAFFOLD ("platform build, not product gap; resume after estate 100%"), so its Railway
    404 means never-deployed, not went-down — but a 6 Aug critical row claims Stripe webhooks /
    Telegram bot / agent runtime depend on that host. If NOTHING references it: resolve rows
    39b1f4da-affb-4833-9fb3-590306494621 and 206487db-642a-4899-9da4-03f124de6c62 citing the grep, and
    record "portfolio has no shared-backend dependency". If ANYTHING references it: that product is
    calling a dead host — name the call sites and tell Phil what breaks.
(b) CRITICAL-CHANNEL HYGIENE: `SELECT ... WHERE severity='critical' AND status='open'` returns ~30 rows,
    mostly stale — topics that literally say "(CLOSED)", repeated INVARIANT GATE FAIL, June Craig-lane
    STAND DOWN orders. That noise hid a real P0 for 11 days. Sweep everything older than 48h: resolve
    with evidence, downgrade to 'high' with a reason, or prefix the topic 'UNVERIFIED-STALE'. From now
    on critical means verified-true-right-now-and-blocking.

TASK 5 — register defects already found, fix while you are in there:
  - pharoah-comms-hub has a product_parts row with part_name IS NULL, status=staged — name it or delete it.
  - pharoah-fusion: its "live" part is NOT probe-proven (the 200 is a password gate), and its deploy_url
    is a workers.dev host, breaching the branded-domain rule.
  - pharoah-vision-pro part "ColorLens colour-assist backend" is misnamed — the source is a SwiftUI
    client; no backend exists.
  - pip: source_repo is NULL and no repo named pip exists in ATTOH-DIGITAL or philpereira371-svg. Ask
    Phil to name the real repo; do not guess.
  - costa-code (55MB) and costa-refs (18MB) were skipped by a size-limited session — attest them here.

TASK 6 — genuinely unbuilt work (only 14 parts across 11 products). Query it live rather than trusting
this list, then build in readiness order:
  SELECT p.product_key, pp.part_name, pp.status, pp.gap_type FROM product_parts pp
  JOIN products p ON p.product_key=pp.product_key
  WHERE p.cluster='pharoah' AND p.stage NOT IN ('excluded','spec')
    AND pp.status IN ('spec','staged') AND coalesce(pp.gap_type,'') <> ''
  ORDER BY p.readiness_pct DESC;

WHEN YOU FINISH A CHUNK — commit and push immediately (never leave work only in a container), and
record it:
  INSERT INTO lane_handoffs (from_lane,to_lane,topic,body,body_evidence,severity,status,related_session)
  VALUES ('mac_code','pharoah','LOCAL: <what you finished>','<detail + what remains>',
          '<jsonb: counts, SHAs, probe codes, timestamps>','normal','open','local-claude-code');
Finish by re-running the Task 1 and Task 2 queries and reporting the fresh counts to Phil, so the true
state is visible without anyone re-deriving it.
```

## If you want the cloud fleet back

Everything is disabled, not deleted. Re-enable `pharoah-estate-autopilot-v2` (trigger `trig_01EJYesq3We36wD5hQL3crJx`) and it resumes hourly with its full Pharoah-first standing orders; the poke triggers and the GhostWire website tracker re-enable the same way. Do the permissions fix in Part 1 item 5 first, or the lanes will silently stall again. `LANE-DOCTRINE.md` in this repo holds the operating rules the whole fleet ran on.
