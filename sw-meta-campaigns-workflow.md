---
name: sw-meta-campaigns-workflow
description: >
  Orchestrates Meta Ads campaign builds end-to-end at Streetwise Consultancy
  through the local Meta Draft Builder MCP. Forces preflight, asset
  validation, API compatibility check, and Cowork-vs-External-vs-Cowork
  session splitting BEFORE any draft creation. Triggers on: build Meta
  campaign, Meta draft, Messages campaign, Leads campaign, Sales campaign,
  Traffic campaign, Awareness campaign, App install campaign,
  meta-draft-builder, OUTCOME_ENGAGEMENT, OUTCOME_LEADS, OUTCOME_SALES,
  OUTCOME_TRAFFIC, OUTCOME_AWARENESS, ad creative production, Meta video cut,
  audience segmentation, Cold and Warm audiences, "let's work on a Meta
  campaign", "draft an ad campaign", "edit targeting", "increase budget",
  "pause campaign", "duplicate campaign", "swap creative". Companion skills:
  sw-meta-api-compatibility-matrix (objective-specific rules + error lookup),
  sw-meta-business-manager-protocol (access), sw-ad-variant-generator
  (creative variants), sw-paid-campaigns-data-normalization-schema
  (reporting shape).
---

# Streetwise Meta Campaign Workflow
**Risk level if violated:** HIGH — late-stage blockers (Meta API rejections, schema mismatches, budget multiplication, mid-word video cuts) burn tokens and produce orphan campaigns in client accounts. This skill enforces the discovery of blockers BEFORE creative work begins.

**Version:** 2.0 (2026-05-13). Replaces v1 which was a generic checklist; v2 adds the API matrix reference, the Cowork-A/External/Cowork-B production split, the smoke-test phase, and Streetwise-specific MCP context from the 2026-05-13 DP Gates Messages build session.

**Companion:** always cross-check `sw-meta-api-compatibility-matrix.md` for the chosen objective.

---

## Core rule

Do **not** attempt to create a Meta draft immediately. Always complete the workflow phases in order. If any gate fails, stop and surface the blocker.

The job is to create the **right campaign structure with the fewest hidden failures** — not "build something quickly."

---

## File-output rule (CRITICAL — apply from the FIRST artifact, not at cleanup)

**Every artifact produced during a client campaign build goes to that client's working folder**, NOT to the session outputs scratchpad. This includes intermediate work — transcripts, cut spec sheets, audio chunks, build pack, copy variations, anything reusable.

Canonical path:
```
/Users/Antonio/Documents/streetwise-consultancy/clients/[Client Name]/04-working/[YYYY-MM]-[campaign-name]/
```

Sub-structure inside the campaign folder:
- `build_pack.md` — the spec doc (audience, structure, copy variations, risk flags)
- `transcripts/` — full video transcripts named by AD theme (not source filename, to avoid the filename↔content trap)
- `cuts/` or final mp4 files at root — final ad-ready creatives
- `segments/` (optional) — intermediate segment cuts if work was done in-session
- Drop the date prefix on the campaign folder so multiple campaigns per client are sortable

**Never** write client artifacts to `/sessions/.../outputs/` — that's the session scratchpad and is invisible to the user across sessions. It's fine to use it for temporary scratch (e.g., extracted .wav files for whisper) but anything reusable must land in the client folder by end of phase.

The session outputs folder is only for files that are NOT client-specific (e.g., a generic tool script you wrote for one-time use). For Streetwise client work, default-to-client-folder always. This rule comes from CLAUDE.md §9 rule 13 and §14.

---

## Phase 1 — Intake

Restate the request into a structured build brief BEFORE touching tools:

- Client name (resolves to `account_scope` in MCP)
- Campaign objective (resolves to `OUTCOME_*`)
- Destination (Messenger / IG Direct / WhatsApp / Website / App / On-Ad form / Phone)
- Geographies + age band + gender
- Audience split: warm (custom audiences) vs cold (interests / lookalikes)
- Total budget + duration + budget type (daily vs lifetime)
- Number of creatives + creative type (single image, video, carousel)
- Source assets status (raw video / already-cut / image / TBD)
- Delivery state: PAUSED for review (default) / pre-set to launch on date / live now

If anything critical is missing, ask once in a compact checklist. Don't ask one question at a time.

---

## Phase 2 — Asset preflight

**Never assume assets are present because the user said they're attached.** This is the #1 cause of wasted setup time.

Checklist:
- Source video/image files: verify they exist locally (in `/uploads` or a granted cowork directory). List the actual filenames found.
- Are files raw long-form or already ad-length cuts? Check duration via `ffprobe`.
- Meta Asset Library status: are uploaded creatives already in the client's Meta library with IDs? If not, plan for upload step.
- Carousel assets: if a carousel is in scope but no design files exist, downgrade to a carousel brief unless the user accepts external production.
- Welcome message (Messages campaigns only): not in scope of the ad itself — must be set on the Page Inbox > Greetings + Auto-reply BEFORE launch.

If source media is missing, STOP and surface that before any copywriting.

---

## Phase 2.5 — MCP smoke test (NEW — saves the most tokens)

**Before writing copy, fire a minimal valid draft for the target objective + scope.** This catches schema mismatches and Meta API rejections in 30 seconds instead of after 45 copy variations.

1. Construct the smallest possible valid draft (1 ad, generic copy "Test", minimal targeting).
2. Call `meta_create_draft`. If schema validation fails:
   - Read the error. Compare against `sw-meta-api-compatibility-matrix.md` §1 to identify the field.
   - Patch `backend/docs/schemas/normalized-draft-schema.json` if a value is missing from an enum.
   - Patch `backend/lib/meta.js` if the creative shape is wrong for the objective (e.g., video_data vs link_data, MESSAGE CTA without value.link).
   - Restart backend via:
     ```
     launchctl unload ~/Library/LaunchAgents/com.carusomartech.meta-draft-builder.plist
     launchctl load   ~/Library/LaunchAgents/com.carusomartech.meta-draft-builder.plist
     curl -s http://localhost:4040/healthz | jq
     ```
3. Once validation passes, call `meta_submit_draft_to_meta` on the smoke-test draft.
4. If submit returns Meta error 100: look up the subcode in `sw-meta-api-compatibility-matrix.md` §4 and apply the fix. Common: Advantage Audience + interests incompatibility for Messages (set AA to 0).
5. Once submit returns `meta_object_ids` populated for all 4 (campaign, ad_set, creative, ad): **archive the smoke-test campaign** via `meta_archive_campaign`, log the path that worked, proceed to Phase 3.

This phase is 1–3 tool calls if everything works, 5–10 if a patch is needed. Either way it's cheaper than discovering the problem after copywriting.

---

## Phase 3 — Map creatives to actual content

**Never map ad copy to filenames alone.** This was the bug that caused a Day-N filename to contain Day-M content in the 2026-05-13 DP Gates session.

For every source video that will be used:
1. Transcribe enough content (5-min chunks if videos are long) to identify the actual theme.
2. Build a content map:
   - filename
   - asset ID (in Meta Library, if uploaded)
   - actual theme — confirmed by transcript, not assumed from filename
   - best hook timestamps (provocative claim, contrarian setup, story start)
   - best close timestamps (declarative punchline, "X = Y" reframe, definitive statement)
   - risks: silent opening, intro references like "welcome back to day 2", self-corrections, mid-sentence pauses
3. If actual content does NOT match the assumed angle, rewrite the copy plan BEFORE drafting.

---

## Phase 4 — Decide creative production path

Honest assessment: AI-driven video editing in this environment is expensive and error-prone. Word-level whisper data + ffmpeg cuts can work, but the iteration cycle (cut → playback → re-cut at correct boundary) is mechanical work that burns tokens without AI leverage.

**Default rule:** if creative production is needed AND source clips are >2 minutes total, split the workflow:

| Session | What runs in it |
|---|---|
| **Cowork A — Strategy & Spec** (this session) | Transcribe videos, identify hook timestamps, write copy variations, produce a **cut spec sheet** with exact timestamps + captions copy + transitions + style notes. Output a build pack. **STOP here.** |
| **External — Production** (Canva, CapCut, Descript, Premiere) | Human applies the cut spec sheet, eyeballs each clip on playback (catches mid-word cuts instantly), burns captions, exports 9:16 1080×1920, uploads to Meta Asset Library, copies the new video IDs |
| **Cowork B — Build** (separate session) | Take video IDs + build pack from above → run Phase 2.5 smoke test if not done → fire production drafts → submit to Meta → handle extension via Chrome MCP or manual Ads Manager |

If source clips are <2 minutes total AND user explicitly accepts in-session cuts, you can do it all in one session. Use these guardrails:
- Always use word-level timestamps from whisper for cut points (`word_timestamps=True`).
- After every cut, verify with `ffmpeg silencedetect` on first 1s + last 1s.
- Verify final duration matches spec ±2%.
- Re-transcribe last 2s of the final cut to confirm clean sentence ending.

**Never** trust whisper SEGMENT-level timestamps for word-precise cuts — they include the start of the next phrase. This caused the "tox..." / "so" mid-word cut bug in the 2026-05-13 session.

---

## Phase 5 — Copy generation

Generate copy AFTER the content map is confirmed and the production path is chosen. Per ad:
- 5 primary text variations (max 125 chars for mobile-visible portion)
- 5 headlines (≤40 chars)
- 5 descriptions (≤30 chars where possible)
- 1 CTA from the valid set per objective (look up in `sw-meta-api-compatibility-matrix.md` §1)

Apply client voice (via `sw-content-voice-antonio` or `sw-content-voice-sean` or client-specific voice skill if it exists). For DP Gates specifically, apply the voice rules in `dp_gates_context.md`.

**Cap copy variations at 5 unless dynamic creative testing is explicitly requested.** Meta's dynamic creative limits: 5 primary text + 5 headline + 5 description per ad.

---

## Phase 6 — Campaign structure design

Document the intended structure BEFORE any draft is fired:

- Campaign count (almost always 1)
- Ad set count (typically 2: AS1 Warm, AS2 Cold)
- Ads per ad set (typically 3-6 if creative variants exist)
- Budget split per ad set (sum = total budget)
- Targeting per ad set:
  - AS1 Warm: custom audiences (include) + minimal age/geo filtering
  - AS2 Cold: interests (flexible_spec) or lookalikes + age + geo + EXCLUDE the warm audiences
- Optimization goal per ad set (look up in matrix)
- Destination type per ad set (look up in matrix)
- Placement strategy (default: Advantage+ Placements ON unless specific reason to manual-place)
- Bid strategy (default: LOWEST_COST_WITHOUT_CAP)
- Attribution spec (default: 7d-click)
- Launch state (default: PAUSED)

**Cross-check against `sw-meta-api-compatibility-matrix.md` §1** for the chosen objective to confirm every field value is in the valid set.

---

## Phase 7 — MCP architecture awareness

The `meta-draft-builder` MCP creates **1 campaign per `meta_create_draft` call**. There is no batch-create-ad endpoint and no add-ad-to-existing-campaign endpoint.

**Consequences:**
- 6 ads × 2 ad sets ≠ 1 draft. The MCP cannot represent this in one call.
- N drafts submitted = N campaigns = N × per-campaign spend.

**Decision tree:**

If user wants 1 campaign with multiple ads/ad-sets:
- **Path A** — Submit 1 foundation draft (the most important variant), then extend in Ads Manager. Either you (via Chrome MCP) or the user manually duplicates the ad and ad set.
- **Path B** — Patch the MCP backend + schema to support multi-ad/multi-adset arrays per draft. ~100 LOC change. Permanent fix but adds another restart cycle and testing burden.
- **Path C** — Manual build pack only. User builds entirely in Ads Manager from the spec doc.

If user accepts N separate campaigns (e.g. for parallel A/B testing):
- **Path D** — Submit N drafts. Each creates its own campaign. Budget divides by N (or user accepts higher total spend).

**Always confirm the path before firing more than 1 draft.** Saying "submitting all 3 drafts" creates 3 campaigns unless explicitly stated otherwise.

---

## Phase 8 — Execution and submission

Fire drafts one at a time. After each `meta_create_draft`:
1. Verify state = `draft_ready` and `missing_items: []`.
2. Call `meta_submit_draft_to_meta` only after the user confirms readiness.
3. If submit fails with Meta error: look up in matrix §4, fix root cause (NOT just the symptom), retry. **Archive any orphan campaigns** the failed submit created via `meta_archive_campaign`.
4. If multiple drafts are queued, submit the FOUNDATION draft first, verify the campaign appears correctly in Ads Manager, THEN proceed with remaining work.

---

## Phase 9 — Post-submission extension

After foundation campaign exists in Meta:
- Via Chrome MCP (requires extension enabled on `business.facebook.com` + `adsmanager.facebook.com`):
  - Duplicate the ad with creative swap (use new video IDs from matrix)
  - Duplicate the ad set with audience swap (warm → cold or vice versa)
  - Adjust budgets per ad set to match the spec
- Or manually by the user, using the build pack as the spec.

End state should match the Phase 6 design exactly. Verify with `meta_list_campaigns`, `meta_list_adsets`, `meta_list_ads` against the chosen `scope`.

---

## Phase 10 — Cleanup and audit

Before declaring done:
- Archive any orphan/test campaigns created during smoke tests or failed submits.
- Save the build pack to `clients/[Name]/04-working/[YYYY-MM]-[campaign-name]/` for future reference.
- Update the client's project memory with: draft IDs, Meta object IDs, audience IDs used, learnings.
- Save a copy of the spec doc + final ad copy to `03-Deliverables/` ONLY after client sign-off.

---

## Mandatory warning patterns

Explicitly warn the user when ANY of these is true:
- Uploaded source files not actually present (per Phase 2 check).
- Meta video IDs not in Asset Library yet.
- Filenames don't match transcribed content (per Phase 3 check).
- Schema validation passes but Meta API will reject — known incompatibility (per matrix §1).
- One draft equals one campaign (per Phase 7).
- Current path would multiply spend beyond brief budget.
- Raw videos being used as temporary creatives pending real edits.
- Advantage Audience setting will conflict with explicit interests.
- Special Ad Category constraints active for the client (e.g., CareerCoin = EMPLOYMENT).

---

## Response style

Keep updates operational and concise. Use this structure:
1. **What is confirmed** (asset, scope, schema state)
2. **What is blocked** (missing assets, schema mismatch, ambiguous user intent)
3. **What decision is needed** (1-2 questions max, with options)
4. **Recommended next step** (path A/B/C/D with reason)

Avoid long speculative builds before blockers are removed. Each unblocking step should produce a state change visible in 1 reply, not 5.

---

## Reusable output templates

### Template: asset status
- Source media present locally: yes/no — `<filepath>`
- Already cut to ad length: yes/no — durations: `<list>`
- Meta video/image IDs available: yes/no — `<id list>`
- Browser access enabled (business.facebook.com): yes/no
- Production path: in-session / external Canva / external editor / N/A

### Template: backend smoke-test result
- Schema validation: pass/fail — error list
- Submit to Meta: pass/fail — error code + subcode
- Patches required: `<list of files + line refs>`
- Restart status: backend uptime + version

### Template: campaign structure summary
- Campaigns intended: 1
- Ad sets intended: warm + cold (or N parallel)
- Ads per ad set: N
- MCP architecture limit: 1 campaign per draft
- Path chosen: A (Chrome extend) / B (backend patch) / C (manual) / D (N campaigns parallel)
- Why: one sentence
- Risk if ignored: one sentence

### Template: execution recommendation
- Recommended: MCP / Chrome / manual
- Estimated tool calls: N
- Estimated session split: single / Cowork-A + External + Cowork-B

---

## Streetwise-specific MCP context

- Backend location: `/Users/Antonio/Documents/streetwise-consultancy/ai-automations/meta-ads/`
- Schema file: `docs/schemas/normalized-draft-schema.json`
- Backend Meta logic: `backend/lib/meta.js` (function `createPausedDraft` — branches on `creative.video_id` for video_data vs link_data shape)
- Restart command: `launchctl unload && launchctl load ~/Library/LaunchAgents/com.carusomartech.meta-draft-builder.plist`
- Health check: `curl -s http://localhost:4040/healthz | jq`
- Logs: `/tmp/meta-draft-builder.log` and `/tmp/meta-draft-builder.err` (on Antonio's host, not in sandbox)
- Backend reads `destination_type`, `bid_strategy`, `schedule` from `ad_set._extras` (NOT top-level on the draft, despite the schema having them as top-level). This is the canonical pattern; do not deviate.
- `toCents()` heuristic: amount ≥ 1000 treated as already-in-cents; amount < 1000 multiplied by 100. So `daily_budget: 1430` = $14.30/day and `daily_budget: 14.30` also = $14.30/day.
- The MCP scope name MUST match a filename (minus .json) in `backend/configs/`. As of 2026-05-13 the registered scopes are: `caruso_martech_internal`, `client_careercoin`, `client_dp_gates`, `client_nineteenth_golf`, `streetwise_dm_internal`.

---

## Final rule

If a blocker appears, surface it early, change path fast, preserve user confidence. **The right campaign structure with the fewest hidden failures wins over a fast-but-wrong delivery every time.**
