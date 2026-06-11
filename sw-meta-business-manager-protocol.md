---
name: sw-meta-business-manager-protocol
description: >
  Defines how Streetwise Consultancy handles Meta Business Manager access,
  ad account ownership, system user tokens, and safe campaign creation
  patterns. Apply this skill any time a client engagement involves Meta Ads,
  Facebook Ads, Instagram Ads, Meta API, ad account permissions, business
  manager setup, or pixel/CAPI work. Triggers on: Meta Business Manager,
  BM access, Facebook Business Manager, ad account access, system user
  token, Meta API access, ads_management permission, partner access, client
  ad account, pixel access, CAPI setup, Meta app review, ads_read,
  Marketing API authorization. This protocol prevents account loss,
  compliance issues, and ban risk.
---

# Streetwise Meta Business Manager Protocol
**Risk level if violated:** HIGH — incorrect setup can lead to client account loss, ban risk, or unrecoverable assets.

---

## Rule 1 — Asset Ownership

**Always request partner access to the client's Business Manager. Never house client ad accounts under Streetwise's BM.**

Reasoning: if you house a client's ad account under your BM, the client can't take it with them when the engagement ends. Worse, if Meta flags Streetwise's BM, every client account goes with it. Partner access keeps assets in the client's name and protects both sides.

### Exception (rare)
If the client genuinely has no Business Manager:
1. Walk them through creating one **under their own credentials** (their email, their domain)
2. Once their BM exists, ask them to add Streetwise as a Partner
3. Never use a Streetwise email to create a client BM
4. Document the BM ID in `clients/[Name]/received/access-credentials.md`

---

## Rule 2 — Permission Levels Required

Request the minimum role that gets the job done. Always.

| Service in scope | Minimum role on ad account | Minimum role on pages |
|---|---|---|
| Reporting only (read-only) | Analyst | Analyst |
| Active campaign management | Advertiser | Advertiser |
| Pixel / CAPI / event setup | Advertiser + Pixel Admin | Editor |
| Full account management | Admin (only if client insists) | Editor |

Never accept lower than required — it blocks delivery.
Never accept higher than required — it expands liability.

---

## Rule 3 — Safe Campaign Creation via API (write operations)

When using the Meta Marketing API to create campaigns programmatically (e.g. through n8n's Campaign Optimization Engine), follow this exact pattern:

1. **Always create campaigns as `status: PAUSED`** — never `ACTIVE` straight from API
2. Human reviews the paused campaign in Ads Manager
3. Human manually flips to `ACTIVE` after review
4. Log every API write: timestamp, campaign ID, action, trigger reason
5. Pair with `sw-approval-gate-pattern` (when built) for Slack approve/reject before any API write

### Why this matters
Meta bans accounts for: sustained rate-limit abuse, mass ad creation to evade review, stolen tokens, or policy-violating creatives at scale. Creating paused campaigns via the official API is exactly what the API is designed for and carries no ban risk.

### What's NOT safe
- Browser automation in Ads Manager UI
- MCP servers writing to Ads Manager via DOM manipulation
- Bulk imports designed to bypass policy review
- Re-using a token tied to a banned account

---

## Rule 4 — Token Setup (one-time per client)

Required steps when setting up Meta API access for a new client:

1. Go to `developers.facebook.com` → **Create App** (Business type)
2. App should be created under the **client's** developer account if possible (so they keep the asset). If not, document why.
3. Inside the app: **Business Settings → System Users → Generate Token**
4. Permissions:
   - `ads_read` — required for reporting
   - `ads_management` — required ONLY if write operations are in scope
   - `business_management` — required for managing assets
5. Token expiry: **None** (system user tokens are long-lived)
6. Store the token in **n8n Credentials** — never inline in workflow JSON, never in any file under git

### App review
- `ads_read` does not require formal app review
- `ads_management` for production use **does** require Meta Marketing API authorization + app review
- Block any write-side build until app review is complete

---

## Rule 5 — Test Sequence (always)

Before pointing any new automation at a real client account:

1. **Meta sandbox ad account** — `act_999999999` style test environment
2. **Streetwise's own test ad account** — for live API behaviour
3. **One client account at low volume** — single campaign, paused only
4. **Full client rollout** — only after steps 1–3 pass

Never skip steps.

---

## Rule 6 — What to Document Per Client

In `clients/[Name]/received/access-credentials.md` (kept OUT of git if private repo):

```markdown
# [Client Name] — Meta Access

## Business Manager
- BM Name:
- BM ID:
- Owner: [Client name + email]
- Streetwise role: Partner (Admin / Advertiser / Analyst)

## Ad Accounts
| Name | ID | Currency | Timezone | Streetwise role |
|---|---|---|---|---|

## Pages
| Name | ID | Streetwise role |
|---|---|---|

## Pixels / CAPI
- Pixel ID:
- CAPI status: configured / pending / not in scope
- Server-side endpoint:

## App / Token (if write access in scope)
- App name:
- App ID:
- Token storage: n8n credential ID [name]
- Token created:
- Permissions granted:
```

---

## Rule 7 — Off-boarding

When an engagement ends:
1. Client removes Streetwise as Partner from their BM
2. Streetwise revokes any system user token created for the engagement
3. Final report archived in `clients/[Name]/outputs/`
4. Note in `clients/[Name]/brief.md`: "Engagement ended YYYY-MM-DD, access revoked YYYY-MM-DD"

Never linger with access after engagement ends. Removes liability.

---

## Hard Rules Summary

1. ALWAYS partner access; NEVER house client assets in Streetwise BM
2. ALWAYS minimum permission required; never higher
3. ALWAYS create API campaigns as PAUSED first
4. ALWAYS test in sandbox → own → low-volume client before full rollout
5. NEVER store tokens in workflow JSON or git
6. NEVER skip Meta app review before deploying `ads_management` writes
7. NEVER use browser automation in Ads Manager — only the official Marketing API
