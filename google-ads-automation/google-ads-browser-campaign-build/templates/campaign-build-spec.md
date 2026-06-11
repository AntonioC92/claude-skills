# Google Ads Campaign Build Spec

> Copy this file per campaign and rename it `<client>-<campaign>-spec.md`. Fill every field before opening the Google Ads editor. The keyword coverage table is the gate — no row may have a blank Coverage column when the build starts.

## Meta

| Field | Value |
|---|---|
| Client | |
| Campaign name | |
| Build owner | |
| Spec date | |
| Build start date | |
| Build complete date | |
| Source of brief | (link to client brief or strategy doc) |

## Campaign-level settings

These are locked or expensive-to-change after creation. Confirm with client before submitting.

| Setting | Value | Locked on save? |
|---|---|---|
| Campaign type | (Search / PMax / Display / Video) | Yes |
| Bid strategy | (Maximise Clicks / Maximise Conversions / Target CPA / Target ROAS / Manual CPC) | No |
| Target CPA / ROAS (if applicable) | | No |
| Daily budget (account currency) | | No |
| Currency | (must match MCC currency or set at campaign creation) | Yes |
| Networks | (Search only / Search + Display) | No |
| Locations | (countries / regions / cities) | No |
| Languages | | No |
| Audience signals | | No |
| Ad rotation | (Optimise / Rotate evenly) | No |
| Start date | | No |
| End date | (or "none") | No |
| Conversion goals selected | | No |
| Tracking template (account-level) | | No |

## Ad groups

Duplicate the block below per ad group.

---

### Ad Group: `<name>`

| Field | Value |
|---|---|
| Ad group ID (after creation) | |
| Default CPC bid (if Manual) | |
| Ad group status | (Enabled / Paused) |

#### Keywords

| Keyword | Match type | Notes |
|---|---|---|
| | Phrase | |
| | Phrase | |
| | Exact | |
| | Exact | |
| | Negative | (campaign-level or ad group-level) |

#### Responsive Search Ads

Duplicate per RSA in the ad group.

##### RSA: `<name>`

| Field | Value |
|---|---|
| Ad ID (after creation) | |
| Final URL | |
| Display path 1 | (max 15 chars) |
| Display path 2 | (max 15 chars) |
| Mobile final URL | (if different) |

**Headlines** (15 required, ≤30 chars each)

| # | Headline | Char count | Pinned position |
|---|---|---|---|
| H1 | | / 30 | (none / 1 / 2 / 3) |
| H2 | | / 30 | |
| H3 | | / 30 | |
| H4 | | / 30 | |
| H5 | | / 30 | |
| H6 | | / 30 | |
| H7 | | / 30 | |
| H8 | | / 30 | |
| H9 | | / 30 | |
| H10 | | / 30 | |
| H11 | | / 30 | |
| H12 | | / 30 | |
| H13 | | / 30 | |
| H14 | | / 30 | |
| H15 | | / 30 | |

**Descriptions** (4 required, ≤90 chars each)

| # | Description | Char count | Pinned position |
|---|---|---|---|
| D1 | | / 90 | (none / 1 / 2) |
| D2 | | / 90 | |
| D3 | | / 90 | |
| D4 | | / 90 | |

##### KEYWORD COVERAGE TABLE — fill this BEFORE opening the editor

For every keyword in this ad group, identify which headline(s) contain its primary term as a substring (case-insensitive). If any keyword has zero coverage, revise the headlines until every keyword is covered. **No row may have a blank Coverage column when the build starts.**

| Keyword | Match type | Headline(s) covering it | Coverage status |
|---|---|---|---|
| | | (e.g. H3, H7, H12) | ✅ / ❌ |
| | | | ✅ / ❌ |
| | | | ✅ / ❌ |
| | | | ✅ / ❌ |
| | | | ✅ / ❌ |

If any row shows ❌, the build is blocked. Revise headlines until all rows are ✅.

#### Sitelink extensions (ad group level, if any)

| # | Headline | Description line 1 | Description line 2 | Final URL |
|---|---|---|---|---|
| 1 | | | | |
| 2 | | | | |
| 3 | | | | |
| 4 | | | | |

#### Callout extensions (ad group level, if any)

| # | Callout text |
|---|---|
| 1 | |
| 2 | |
| 3 | |
| 4 | |

---

## Build outcome (fill after Phase 3)

| Field | Value |
|---|---|
| Campaign ID | |
| All ad group IDs | |
| Final ad strength per RSA | (e.g. CO-RSA: Good, Migration-RSA: Excellent) |
| Save confirmation screenshot links | (link to file) |
| Ads list view confirmation screenshot | (link to file) |
| Discrepancies caught at read-back | (none / list them) |
| Time-to-completion | (hours) |
| Sessions used | (count) |
| Postmortem file | (link to filled postmortem) |
