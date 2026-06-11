---
name: sw-seo-gsc-ga4-normalization-schema
description: Defines the organic search data schema, computed SEO KPI formulas, and opportunity flag logic for GSC and GA4 data in the AI Marketing Intelligence Engine. Use this skill whenever building or modifying a Transform node for Google Search Console or Google Analytics 4 data, or when ensuring normalized organic data output is compatible with the AI analysis layer. Always apply this schema before writing any GSC or GA4 transformation code.
---

# SKILL: GSC + GA4 Data Normalization Schema
**Version:** 1.0 | **Project:** AI Marketing Intelligence Engine
**Category:** SEO

---

## Purpose

All organic search modules must normalize GSC and GA4 data to this shared schema before passing to the AI analysis layer.
This ensures Claude can analyze organic performance consistently and cross-reference it against paid campaign data from Meta or Google Ads.

---

## Master Normalized Schema

### GSC Query Schema

```javascript
{
  source:              string,   // hardcoded "gsc"
  account_name:        string,   // from Client Registry
  site_url:            string,   // GSC property URL
  query:               string,   // search query
  page:                string,   // landing page URL
  clicks:              int,      // total clicks from search
  impressions:         int,      // total impressions in search
  ctr:                 float,    // click-through rate % (provided by API)
  position:            float,    // average ranking position (provided by API)
  ctr_opportunity:     float,    // computed — expected CTR at position minus actual CTR
  opportunity_flag:    string,   // computed — see flags below
  date_range:          string,   // date preset used
  analyzed_at:         string    // ISO timestamp
}
```

### GA4 Session Schema

```javascript
{
  source:              string,   // hardcoded "ga4"
  account_name:        string,   // from Client Registry
  property_id:         string,   // GA4 property ID
  channel:             string,   // sessionDefaultChannelGroup
  page:                string,   // page path
  sessions:            int,      // total sessions
  total_users:         int,      // unique users
  bounce_rate:         float,    // bounce rate % (provided by API)
  engagement_rate:     float,    // computed — 1 - bounce_rate
  date_range:          string,   // date preset used
  analyzed_at:         string    // ISO timestamp
}
```

---

## Computed KPIs Formula

Always compute these in the Transform node:

```javascript
// GSC computed fields
const expectedCTR = position <= 1  ? 28.5
                  : position <= 2  ? 15.7
                  : position <= 3  ? 11.0
                  : position <= 5  ? 7.0
                  : position <= 10 ? 3.5
                  : 1.0;

const ctrOpportunity = +(expectedCTR - ctr).toFixed(2);  // positive = underperforming vs position
const impressionScore = impressions > 1000 ? 'HIGH' : impressions > 100 ? 'MEDIUM' : 'LOW';

// GA4 computed fields
const engagementRate = +(100 - bounceRate).toFixed(2);
```

---

## Opportunity Flags Logic

Always apply these flags in the Transform node:

```javascript
// GSC flags
const flags = [];
if (position <= 10 && ctr < 2.0 && impressions > 500)   flags.push('LOW_CTR_TOP10');
if (position > 10 && position <= 20 && impressions > 200) flags.push('PAGE2_OPPORTUNITY');
if (ctrOpportunity > 5 && impressions > 1000)             flags.push('HIGH_CTR_OPPORTUNITY');
if (clicks === 0 && impressions > 100)                    flags.push('ZERO_CLICK_WASTE');

// GA4 flags
if (bounceRate > 70 && sessions > 100)                   flags.push('HIGH_BOUNCE_RATE');
if (channel === 'Organic Search' && sessions < 10)        flags.push('LOW_ORGANIC_TRAFFIC');
```

---

## Output Structure from Transform Node

Always return in this format:

```javascript
return [{
  json: {
    gsc_queries:      normalizedQueries,
    ga4_sessions:     normalizedSessions,
    total_queries:    normalizedQueries.length,
    total_pages:      [...new Set(normalizedQueries.map(q => q.page))].length,
    site_url:         siteUrl,
    account_name:     accountName
  }
}];
```

---

## API Field Mappings

### Google Search Console API
| Normalized Field | GSC API Field |
|---|---|
| `query` | `keys[0]` (when dimension = query) |
| `page` | `keys[1]` (when dimension = query,page) |
| `clicks` | `clicks` |
| `impressions` | `impressions` |
| `ctr` | `ctr * 100` (API returns decimal, convert to %) |
| `position` | `position` |

### GA4 Analytics Data API
| Normalized Field | GA4 API Field |
|---|---|
| `channel` | `dimensionValues[0]` (sessionDefaultChannelGroup) |
| `page` | `dimensionValues[1]` (pagePath) |
| `sessions` | `metricValues[0]` (sessions) |
| `total_users` | `metricValues[1]` (totalUsers) |
| `bounce_rate` | `metricValues[2] * 100` (bounceRate, convert to %) |

---

## Cross-Source Enrichment

When both GSC and Meta Ads data are available, compute these overlap fields:

```javascript
// Paid vs Organic overlap detection
const paidKeywords   = metaCampaigns.map(c => c.campaign_name.toLowerCase());
const organicQueries = gscQueries.map(q => q.query.toLowerCase());

const overlap = organicQueries.filter(q =>
  paidKeywords.some(k => k.includes(q) || q.includes(k))
);

// Flag as: PAID_ORGANIC_OVERLAP — keyword has both paid spend and organic ranking
// Flag as: CONTENT_GAP — paid spend exists, zero organic impressions
```

---

## Google Cloud Setup Requirements

The same Google OAuth2 credential used for Google Drive can be extended for GSC and GA4.
In Google Cloud Console (`n8n meta ads automation` project):

- Enable **Google Search Console API**
- Enable **Google Analytics Data API**
- Add service account email as **Read** user in GSC property
- Add service account email as **Viewer** in GA4 property

No new credential needed in n8n — use the existing Google OAuth2.

---

## Validation Rules

Before passing to AI:
- `ctr` must be a float percentage (e.g. `2.4` not `0.024`)
- `position` must be a float, never rounded to int
- `opportunity_flag` must be an array (empty array if no flags)
- `analyzed_at` must be ISO format: `new Date().toISOString()`
- `source` must be lowercase string (`"gsc"` or `"ga4"`)
- `bounce_rate` must be a percentage float (e.g. `65.4` not `0.654`)
