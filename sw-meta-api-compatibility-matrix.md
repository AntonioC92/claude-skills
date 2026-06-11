---
name: sw-meta-api-compatibility-matrix
description: >
  Per-objective Meta API compatibility matrix. Use this skill any time a Meta
  draft is being planned or built — BEFORE writing copy, cutting creatives, or
  firing a meta_create_draft call. Triggers on: Meta campaign, Meta objective,
  OUTCOME_LEADS, OUTCOME_SALES, OUTCOME_ENGAGEMENT, OUTCOME_TRAFFIC,
  OUTCOME_AWARENESS, OUTCOME_APP_PROMOTION, Messages campaign, leads campaign,
  sales campaign, conversions campaign, traffic campaign, awareness campaign,
  app install campaign, meta_create_draft, ad_set rejection, Meta error 100,
  destination_type, optimization_goal, promoted_object, advantage audience,
  attribution spec, special ad categories, lead form, pixel event, conversion
  domain. Prevents wasted tokens by surfacing objective-specific blockers
  BEFORE creative work begins. Companion to sw-meta-campaigns-workflow.
---

# Streetwise Meta API Compatibility Matrix
**Risk level if violated:** HIGH — wrong objective × optimization_goal × CTA × creative shape combination causes silent draft acceptance + Meta-side submission rejection mid-build, burning tokens and creating orphan campaigns in the client account.

**How to use this skill:** look up the user's intended objective in §1, run the relevant preflight in §3, cross-check the cross-cutting rules in §2. Use the known-error lookup in §4 when a submit fails.

---

## 1. Objective compatibility matrix

Each row is the **only known-good** combination shape per objective. If the user wants a combination NOT in this matrix, stop and surface the constraint before drafting.

### OUTCOME_ENGAGEMENT — Messages campaigns (DM-driven)

| Field | Valid values |
|---|---|
| `campaign.objective` | `OUTCOME_ENGAGEMENT` |
| `ad_set.optimization_goal` | `CONVERSATIONS` (messaging conversions) · `POST_ENGAGEMENT` · `REACH` · `IMPRESSIONS` |
| `ad_set.destination_type` (in _extras) | `MESSENGER` · `INSTAGRAM_DIRECT` · `WHATSAPP` · `ON_AD` |
| `ad_set.promoted_object` | `{ "page_id": "<numeric>" }` — page_id required |
| `creative.cta` | `SEND_MESSAGE` · `MESSAGE_PAGE` · `WHATSAPP_MESSAGE` |
| `creative.landing_page_url` | NOT required (Meta resolves destination to Page DM) |
| Creative shape | `video_data` with `call_to_action: { type: <cta> }` — NO `value.link` field |
| Welcome message | Must be set on the FB Page Inbox > Automated Responses BEFORE launch — not in the ad |
| Daily budget minimum (CONVERSATIONS) | $5+/day to escape learning phase; lifetime accepted |

**Known Meta-side rejection patterns:**
- **Error 100 / subcode 1870189** = "targeting cannot be empty" — Advantage Audience + explicit interests INCOMPATIBLE for Messages CONVERSATIONS. Set `targeting_automation.advantage_audience: 0` if explicit interests are used.
- `smart_pse_enabled` flag NOT in current schema enum — backend may auto-add for MESSENGER if needed; do not pass it.
- destination_type cannot be `WEBSITE` for Messages.

---

### OUTCOME_LEADS — Instant Forms / Lead Gen

| Field | Valid values |
|---|---|
| `campaign.objective` | `OUTCOME_LEADS` |
| `ad_set.optimization_goal` | `LEAD_GENERATION` · `QUALITY_LEAD` (Meta-curated, may inflate CPL) · `OFFSITE_CONVERSIONS` (for website-form lead campaigns only) |
| `ad_set.destination_type` (in _extras) | `ON_AD` for Instant Forms · `WEBSITE` for off-Meta lead pages |
| `ad_set.promoted_object` | `{ "page_id": "<numeric>" }` for Instant Forms · `{ "pixel_id": ..., "custom_event_type": "LEAD" }` for off-Meta |
| `creative.cta` | `SIGN_UP` · `GET_QUOTE` · `LEARN_MORE` · `SUBSCRIBE` · `APPLY_NOW` · `GET_OFFER` · `DOWNLOAD` |
| Lead form ID | Required for Instant Forms — must be set on the ad creative (Cowork MCP schema may not expose this yet — verify before drafting) |
| `creative.landing_page_url` | NOT required for Instant Forms (form is on Meta); required for off-Meta lead pages |
| Creative shape | `video_data` or `link_data` |

**Prerequisites:**
- Lead Form must be CREATED in Meta UI: Page > Forms Library > Create New, BEFORE the draft. Capture the form_id.
- For QUALITY_LEAD optimization: needs ≥10 leads/week historical baseline; new accounts use LEAD_GENERATION instead.
- For QUALITY_LEAD with CRM integration: Meta API for Leads CAPI required.

---

### OUTCOME_SALES — Conversions (purchases, signups, etc.)

| Field | Valid values |
|---|---|
| `campaign.objective` | `OUTCOME_SALES` |
| `ad_set.optimization_goal` | `OFFSITE_CONVERSIONS` · `VALUE` (requires ROAS bid strategy + revenue data) |
| `ad_set.destination_type` (in _extras) | `WEBSITE` (default) · `APP` for app conversion |
| `ad_set.promoted_object` | `{ "pixel_id": "<numeric>", "custom_event_type": "PURCHASE" }` — pixel_id + event required. Other event types: `ADD_TO_CART` · `INITIATE_CHECKOUT` · `LEAD` · `COMPLETE_REGISTRATION` · `SUBSCRIBE` · `OTHER` |
| `creative.cta` | `SHOP_NOW` · `LEARN_MORE` · `GET_OFFER` · `SUBSCRIBE` · `BOOK_TRAVEL` · `DOWNLOAD` |
| `creative.landing_page_url` | REQUIRED — must be valid URL, must match conversion_domain |
| Creative shape | `link_data` with valid URL; `video_data` for video creative with link spec |
| `ad_set.attribution_spec` | REQUIRED in iOS 14.5+ era. Format: `[{ "event_type": "CLICK_THROUGH", "window_days": 7 }]` (also 1d/CLICK_THROUGH, 1d/VIEW_THROUGH allowed combinations) |
| `ad_set.conversion_domain` | REQUIRED for OFFSITE_CONVERSIONS — domain must be verified in BM > Brand Safety > Domains |

**Prerequisites (verify BEFORE drafting):**
- Domain verified in Business Manager > Brand Safety > Domains (DNS or meta-tag).
- Pixel installed AND firing the target event (check Events Manager Test Events).
- Aggregated Event Measurement (AEM) configured: target event prioritized in top 8 events for the domain.
- For VALUE optimization: pixel must include `value` and `currency` params on Purchase events.

**Known rejection patterns:**
- Pixel not linked to ad account → error 100, subcode varies. Fix: BM > Pixels > Add Asset > Ad Account.
- Domain not verified → submit succeeds but Meta downranks delivery silently.
- attribution_spec mismatch with optimization_goal → error 100.

---

### OUTCOME_TRAFFIC — clicks and landing page views

| Field | Valid values |
|---|---|
| `campaign.objective` | `OUTCOME_TRAFFIC` |
| `ad_set.optimization_goal` | `LINK_CLICKS` · `LANDING_PAGE_VIEWS` · `REACH` · `IMPRESSIONS` |
| `ad_set.destination_type` (in _extras) | `WEBSITE` · `MESSENGER` (click-to-message) · `APP` · `PHONE_CALL` |
| `ad_set.promoted_object` | Not required for WEBSITE traffic; `{ page_id }` for MESSENGER click-to-message |
| `creative.cta` | `LEARN_MORE` · `SHOP_NOW` · `SIGN_UP` · `BOOK_TRAVEL` · `SUBSCRIBE` · `CONTACT_US` · `DOWNLOAD` · `GET_QUOTE` · `LISTEN_NOW` |
| `creative.landing_page_url` | REQUIRED for WEBSITE; NOT for MESSENGER |
| Creative shape | `link_data` or `video_data` |

**Notes:**
- Simplest objective. Fewest Meta-side gotchas.
- `LANDING_PAGE_VIEWS` requires the page to fire the standard pixel `PageView` event reliably. Slow-loading pages get fewer attributed views.
- For click-to-message traffic, this is functionally similar to OUTCOME_ENGAGEMENT/Messages but with click-optimization instead of conversation-optimization.

---

### OUTCOME_AWARENESS — reach and brand lift

| Field | Valid values |
|---|---|
| `campaign.objective` | `OUTCOME_AWARENESS` |
| `ad_set.optimization_goal` | `REACH` · `IMPRESSIONS` · `THRUPLAY` (video creatives only) · `AD_RECALL_LIFT` · `TWO_SECOND_CONTINUOUS_VIDEO_VIEWS` |
| `ad_set.destination_type` (in _extras) | `WEBSITE` typically; can be `ON_AD` for video-only views |
| `ad_set.promoted_object` | Not required |
| `creative.cta` | Any |
| Creative shape | `link_data` or `video_data` |

**Notes:**
- Configure frequency cap on the ad set: `frequency_control_specs: [{ "event": "IMPRESSIONS", "interval_days": 7, "max_frequency": 2 }]`.
- For AD_RECALL_LIFT optimization, Meta requires a baseline study; not available on all accounts.
- Brand Lift Studies are a separate Meta product opt-in (Meta sales contact required).

---

### OUTCOME_APP_PROMOTION — app installs and in-app events

| Field | Valid values |
|---|---|
| `campaign.objective` | `OUTCOME_APP_PROMOTION` |
| `ad_set.optimization_goal` | `APP_INSTALLS` · `OFFSITE_CONVERSIONS` (for in-app events) · `VALUE` · `LINK_CLICKS` (retargeting installs) |
| `ad_set.destination_type` (in _extras) | `APP` |
| `ad_set.promoted_object` | `{ "application_id": "<numeric>", "object_store_url": "<store URL>" }` — both required |
| `creative.cta` | `INSTALL_NOW` · `USE_APP` · `PLAY_GAME` · `LEARN_MORE` |
| Creative shape | `link_data` or `video_data`; deep links via `app_link_spec` |

**Prerequisites:**
- App registered in Meta Developer dashboard + linked to ad account.
- SDK installed for in-app event optimization.
- Universal links / deep links configured for retargeting flows.

---

## 2. Cross-cutting rules (apply to ALL objectives)

### 2.1 Special Ad Categories
Categories: `EMPLOYMENT` · `HOUSING` · `CREDIT` · `ISSUES_ELECTIONS_POLITICS`

If the client's config has `default_special_ad_categories` set (check `backend/configs/client_<name>.json`), the backend forces it onto every campaign. Streetwise's CareerCoin client is `EMPLOYMENT`. Impact:
- No age limits (must be 18+ to 65+ broad)
- No gender targeting
- No zip-code-level targeting (city radius only)
- No interest-based detailed targeting in some cases
- Lookalike audiences allowed but country-restricted

If a campaign violates these constraints, Meta either rejects at submit or silently strips the offending targeting.

### 2.2 Advantage Audience expansion
- Field: `targeting.targeting_automation.advantage_audience` — required to be present (0 or 1) in current Meta API.
- Backend default: `1` (on) if not specified.
- **INCOMPATIBLE with explicit `flexible_spec.interests` for OUTCOME_ENGAGEMENT/CONVERSATIONS** → error 100/1870189.
- For most other objectives: AA + interests works (AA expands AROUND the explicit interests).
- Default rule: if explicit interests are critical → set AA to 0. If broad reach is desired → set AA to 1, remove explicit interests.

### 2.3 Lookalike audiences
- Source audience must have ≥1000 people.
- LAL must be in the same country as the targeting.
- LAL 1% is most similar; 10% is largest (and most diluted).

### 2.4 Custom audiences
- Must EXIST before draft creation. Can be referenced by ID in `targeting.custom_audiences` (include) or `targeting.excluded_custom_audiences` (exclude).
- Pixel-based custom audiences need ≥100 hits in lookback window to be eligible.

### 2.5 Placements
- Reels: 9:16 only (1080×1920).
- Stories: 9:16 only.
- Feed: 1:1 (1080×1080), 4:5 (1080×1350), or 16:9 (1080×608).
- Right Column: image only, 1.91:1 minimum.
- If `publisher_platforms` or `placements` arrays not specified, backend defaults to `['facebook', 'instagram']` (no Audience Network or Messenger inbox).

### 2.6 Budget minimums and learning phase
- Daily minimum: $1/day for most countries; $5/day for CONVERSATIONS optimization to escape learning.
- Lifetime minimum: 24h flight duration; recommended ≥7 days.
- Learning phase exits when ad set hits ~50 optimization events/week. For CONVERSATIONS at $14/day generating 1-2 conversations/day, learning phase never exits in a 21-day flight — accept this and don't make mid-flight edits.

### 2.7 Attribution spec (iOS 14.5+ reality)
For any conversion-optimized objective (OUTCOME_SALES, OUTCOME_LEADS off-Meta, OUTCOME_ENGAGEMENT/CONVERSATIONS), the ad set typically needs:
```
"attribution_spec": [
  { "event_type": "CLICK_THROUGH", "window_days": 7 }
]
```
Valid combinations: `1d-click`, `7d-click`, `1d-click-and-1d-view`, `7d-click-and-1d-view`. Pick based on the client's funnel; default 7d-click.

### 2.8 Budget multiplication risk (Streetwise MCP-specific)
The `meta-draft-builder` MCP creates ONE campaign per `meta_create_draft` submission. Submitting N drafts = N campaigns = Nx the intended spend if all are launched. Always confirm with the user BEFORE firing multiple drafts whether they want:
- N separate campaigns (parallel A/B; budget divided by N)
- 1 campaign with N ads sharing budget (fire 1 draft, extend manually in Ads Manager)

---

## 3. Objective-specific preflight checklists

Run the relevant checklist BEFORE any creative or copy work. If a check fails, surface it to the user immediately.

### 3.1 Messages preflight
- [ ] Page has Inbox enabled (Messenger active)
- [ ] Welcome message draft prepared (will be set on Page > Automated Responses before launch)
- [ ] Page_id confirmed numeric
- [ ] Custom audiences (if any) exist
- [ ] Advantage Audience setting consciously chosen (default: OFF if using interests)
- [ ] destination_type: MESSENGER chosen (covers Messenger + IG Direct via Meta auto-routing)

### 3.2 Leads preflight
- [ ] Lead Form already exists in Page > Forms Library
- [ ] Form ID captured (numeric)
- [ ] Lead routing tested: does Meta CAPI for Leads feed the client's CRM?
- [ ] Form fields match the data the sales team actually needs
- [ ] Privacy policy URL set on the form (Meta-required)
- [ ] Thank-you screen + redirect URL set

### 3.3 Sales preflight
- [ ] Pixel installed AND firing target event (verify in Events Manager > Test Events)
- [ ] Domain verified in BM > Brand Safety > Domains
- [ ] AEM event priority set (target event in top 8)
- [ ] attribution_spec decided (default 7d-click)
- [ ] conversion_domain matches landing_page_url root domain
- [ ] For VALUE optimization: pixel includes `value` and `currency` params
- [ ] Pixel has ≥50 events/week historical baseline (or accept long learning phase)

### 3.4 Traffic preflight
- [ ] Landing page URL valid and tested (HTTPS, loads fast, mobile-ready)
- [ ] UTM parameters defined and consistent across ad copy
- [ ] Pixel PageView event firing on landing page (for LANDING_PAGE_VIEWS opt)

### 3.5 Awareness preflight
- [ ] Frequency cap discussed with client (default 2/7d unless they want higher)
- [ ] Creative quality verified — awareness campaigns punish low-quality creative more than other objectives

### 3.6 App Promotion preflight
- [ ] App linked to ad account in Meta Developer dashboard
- [ ] SDK installed for event optimization
- [ ] Object store URL valid (App Store / Play Store)
- [ ] Deep links configured if retargeting installed users

---

## 4. Known Meta error code lookup

| Code / Subcode | Meaning | Likely fix |
|---|---|---|
| 100 / 1870189 | Targeting cannot be empty | Advantage Audience + explicit interests incompatibility → set AA to 0 |
| 100 / 1815216 | Custom audience expired | Refresh source data in BM > Audiences |
| 100 / 1487006 | Promotion contains expired ads | Recreate ad with active creative |
| 100 / 2491 | Invalid placement combination | Specify `publisher_platforms` explicitly |
| 100 / 1885133 | Lookalike audience expired | Refresh source in BM > Audiences |
| 100 / 1487749 | Account does not have permission for that targeting | Check Special Ad Category restrictions |
| 100 / 1487390 | Pixel does not exist or not accessible | Verify pixel linked to ad account |
| 100 / 1815183 | Audiences haven't been refreshed | Wait 24h or trigger refresh |
| 200 | Permission denied | Token scope insufficient — needs `ads_management` |
| 102 | Session expired | Refresh system user token |
| 17 | User request limit reached | Throttle, wait 1-2 minutes |
| 1487060 | Ad approved but failed delivery | Review against Meta ad policies |
| 1487079 | Asset not found | Video/image ID invalid or asset deleted |

If a Meta error appears and isn't in this table: surface the raw error to the user, look up at `developers.facebook.com/docs/marketing-api/error-reference`, and add the new mapping to this matrix.

---

## 5. Decision rules

**Before ANY meta_create_draft call:**
1. Confirm objective with user (don't infer from ambiguous phrasing).
2. Look up the objective in §1 — copy the valid fields onto a working spec.
3. Run the §3 preflight checklist for that objective.
4. Check §2 cross-cutting rules (Special Ad Categories, AA, attribution spec, budget multiplication).
5. If all pass: fire the draft.
6. If submit fails: look up the error in §4, fix, retry.

**If user asks for something this matrix doesn't cover:** stop and say "this combination isn't in the known-good matrix — let me verify with Meta API docs before drafting." Then verify, then add the result to this matrix.
