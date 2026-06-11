---
name: sw-utm-governance
description: >
  The single Streetwise UTM naming convention applied to every paid campaign,
  email link, social post, and outbound message across every client. Apply
  this skill any time a tracking parameter, campaign URL, or link is being
  built or audited. Triggers on: UTM, UTM parameters, tracking parameters,
  utm_source, utm_medium, utm_campaign, utm_content, utm_term, campaign
  naming, link tracking, attribution setup, GA4 channel grouping, source
  medium, link builder, URL builder, tracking link, audit campaign URLs,
  attribution issues, dirty data. Prevents the reporting cross-contamination
  that destroys client attribution.
---

# Streetwise UTM Governance
**Risk level if violated:** MEDIUM — bad UTMs corrupt analytics for months and cause attribution disputes with clients.
**Owner:** Antonio (technical) | applied by anyone building a link.

---

## Core Rule

Every link Streetwise creates that drives traffic — paid ad, email, social post, outbound DM, podcast plug, anything — must include UTMs. The convention below is mandatory and not editable per client.

---

## The Convention (5 parameters)

```
?utm_source=[platform]
&utm_medium=[channel-type]
&utm_campaign=[client-yyyymm-campaign-name]
&utm_content=[ad-or-creative-id]
&utm_term=[audience-or-keyword]
```

All values: lowercase, hyphenated, no spaces, no special characters except `-`.

---

## Allowed Values per Parameter

### `utm_source` — the platform the click came from

| Value | When |
|---|---|
| `meta` | Facebook + Instagram (always combined) |
| `google` | Google Ads (search + display + YouTube + Pmax) |
| `linkedin` | LinkedIn Ads + LinkedIn organic |
| `tiktok` | TikTok Ads |
| `pinterest` | Pinterest Ads |
| `email` | Any email platform (Brevo, HubSpot, Klaviyo, GHL) |
| `outbound` | Manual outreach (Sean's LinkedIn DMs, cold email tools) |
| `referral` | Partner / affiliate links |
| `event` | Live event QR codes / handouts |
| `youtube` | YouTube video links (paid or organic) |
| `podcast` | Podcast show notes |

Not on this list → ask Antonio before adding.

### `utm_medium` — the channel type

| Value | When |
|---|---|
| `cpc` | Paid search and paid social with CPC bidding |
| `display` | Display + retargeting banners |
| `video` | Video ads (Meta Reels, YouTube TrueView, LinkedIn video) |
| `social-organic` | Organic social posts |
| `social-paid` | Paid social posts (alternate for `cpc` when the report needs the distinction) |
| `email-broadcast` | One-off email broadcasts |
| `email-nurture` | Lifecycle / sequence emails |
| `email-transactional` | Receipts, confirmations |
| `dm` | Direct messages (LinkedIn, Instagram) |
| `affiliate` | Partner referrals |
| `qr` | QR codes (event handouts, print) |
| `organic` | Unpaid web traffic where attributable |

### `utm_campaign` — the campaign label

Format: `[client]-[yyyymm]-[campaign-name]`

Examples:
- `streetwise-202605-q2-launch`
- `acme-202605-summer-promo`
- `dublinbeerfest-202504-tickets`

Rules:
- Client name = same shortname every time (define once in `clients/[name]/brief.md`)
- Date = year+month of campaign start, not creation date
- Campaign name = descriptive, ≤30 chars, hyphenated

### `utm_content` — the specific ad/creative

Format: `[ad-id]-[variant]` or `[creative-name]-[variant]`

Examples:
- `v001-square`
- `v047-vertical`
- `hero-image-a`

Used to differentiate variants (see `sw-ad-variant-generator`).

### `utm_term` — the audience or keyword

For paid search: the actual keyword (Google Ads can auto-populate via `{keyword}`).
For paid social: the audience name (e.g. `lookalike-1pct-purchasers`, `interest-saas-founders`).
For organic posts / DMs: leave blank.

---

## Examples

### Meta paid campaign for a client
```
https://acme.com/landing?
utm_source=meta
&utm_medium=cpc
&utm_campaign=acme-202605-summer-promo
&utm_content=v012-square
&utm_term=lookalike-1pct-purchasers
```

### Sean's LinkedIn cold DM
```
https://streetwiseconsultancy.com/case-studies/beer-festival?
utm_source=outbound
&utm_medium=dm
&utm_campaign=streetwise-202605-linkedin-outreach
&utm_content=case-study-beerfest
```

### Email nurture sequence step
```
https://acme.com/demo?
utm_source=email
&utm_medium=email-nurture
&utm_campaign=acme-202605-saas-trial
&utm_content=email-3-cta-bottom
```

---

## Build Process

1. **Always use a URL builder** — never hand-type UTMs in a campaign manager. Errors compound.
   - Recommended: Google's Campaign URL Builder OR a Streetwise-internal builder in a Notion/Sheets template
2. **Lowercase everything** — UTMs are case-sensitive in GA4. `Meta` ≠ `meta`.
3. **No URL encoding** in the values — `summer promo` becomes `summer-promo`, NOT `summer%20promo`.
4. **Test the link** — paste into a browser, confirm it lands on the right page and the GA4 DebugView captures the UTMs correctly.

---

## Audit Cadence

Once per quarter, run an audit:

1. Pull last 90 days of GA4 source/medium data for each client
2. Flag any sessions where:
   - `utm_source` is empty (link missing UTMs)
   - `utm_medium` is empty
   - `utm_campaign` is generic (e.g. `untitled`, `test`, `default`)
   - `utm_source` is not on the allowed list
3. Trace flagged sessions back to the campaign and fix the source link
4. If the source is a third-party (e.g. agency-managed link), notify the client

Document audit findings in `clients/[name]/outputs/Reports/utm-audit-YYYY-MM-DD.md`.

---

## Why the Convention is Strict

| Without governance | With governance |
|---|---|
| GA4 has 50+ source/medium combinations, half of them duplicates (`Facebook`, `facebook`, `FB`, `meta`) | One canonical set, dashboards work |
| Attribution disputes with clients ("how did this lead come in?") | Every lead is traceable to a campaign + creative + audience |
| Reports take 4 hours to clean before they can be used | Reports auto-populate from clean data |
| Cross-channel comparison is impossible | LinkedIn vs Meta vs Google ROAS is directly comparable |

---

## Hard Rules

1. Never publish a link without UTMs (paid OR organic where Streetwise controls the link)
2. Never invent a new `utm_source` or `utm_medium` value without adding it to this skill first
3. Always lowercase, always hyphens, never spaces or underscores
4. Always client-yyyymm-campaign in `utm_campaign` — no exceptions
5. Always test the link before launch
6. Quarterly audit is mandatory, not optional
7. If a client provides their own UTMs that conflict with this convention, write a tracking layer that adds Streetwise UTMs in addition (don't fight the client; double-track)

---

## Integration with Other Skills

- `sw-paid-campaigns-data-normalization-schema` — normalised reports rely on clean UTMs
- `sw-seo-gsc-ga4-normalization-schema` — GA4 channel grouping depends on these values
- `sw-ad-variant-generator` — variant IDs become `utm_content` values
- `sw-client-onboarding` — the client shortname is locked in at onboarding
