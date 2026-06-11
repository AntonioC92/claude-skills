---
name: sw-seo-ai-visibility-tracking
description: Defines how to ingest AI Overview citation data from DataForSEO or Bing Webmaster Tools, normalize it against the existing campaign schema, and add an AI Visibility section to any performance report. Use this skill whenever building or modifying an AI visibility tracking module, or when adding AI search presence data to an existing Meta Ads or SEO report. Always apply this schema before writing any AI visibility ingestion or normalization code.
---

# SKILL: AI Visibility Tracking Schema
**Version:** 1.0 | **Project:** AI Marketing Intelligence Engine
**Category:** SEO

---

## Purpose

AI search (Google AI Overviews, ChatGPT, Perplexity, Copilot) is now a traffic source alongside organic and paid.
This skill defines how to track whether a site is being cited in AI-generated answers, normalize that data, and surface it inside existing performance reports.

---

## Data Sources Supported

| Source | Type | Cost | What It Provides |
|---|---|---|---|
| Bing Webmaster Tools | CSV export | Free | Copilot citation data, grounding queries, page-level citations |
| DataForSEO AI Overview API | API | ~$0.01/query | Google AI Overview content + cited URLs |
| Perplexity Sonar API | API | Low cost | LLM responses with web citations included |
| SerpApi | API | From $75/mo | Full SERP JSON including AI Overviews |

**Recommended starting point:** Bing Webmaster Tools (free) + DataForSEO (pay-as-you-go).

---

## Master Normalized Schema

```javascript
{
  source:              string,   // "bing_wmt" | "dataforseo" | "perplexity" | "serpapi"
  account_name:        string,   // from Client Registry
  query:               string,   // the search query or prompt
  ai_platform:         string,   // "google_ai_overview" | "copilot" | "chatgpt" | "perplexity"
  cited:               boolean,  // was this site cited in the AI response
  cited_url:           string,   // specific URL cited (null if not cited)
  citation_position:   int,      // position in citation list (null if not cited)
  ai_response_snippet: string,   // brief excerpt of AI-generated answer (max 200 chars)
  competitor_cited:    boolean,  // was a known competitor cited instead
  competitor_url:      string,   // which competitor was cited (null if none)
  impressions:         int,      // how many times query triggered AI response (if available)
  date_range:          string,   // date range of data
  analyzed_at:         string    // ISO timestamp
}
```

---

## Computed Fields

Always compute these in the Transform node:

```javascript
const citationRate   = totalQueries > 0
  ? +((citedQueries / totalQueries) * 100).toFixed(2)
  : 0;

const competitorGap  = competitorCitedQueries.filter(q =>
  !siteCitedQueries.includes(q)
);  // queries where competitor is cited but you are not

const avgCitationPos = citedResults.length > 0
  ? +(citedResults.reduce((sum, r) => sum + r.citation_position, 0) / citedResults.length).toFixed(1)
  : null;
```

---

## Visibility Flags Logic

```javascript
const flags = [];
if (cited === false && competitor_cited === true)          flags.push('COMPETITOR_CITED_NOT_YOU');
if (citation_position > 3 && cited === true)              flags.push('LOW_CITATION_POSITION');
if (citationRate < 10 && totalQueries > 20)               flags.push('LOW_AI_VISIBILITY');
if (cited === true && citation_position === 1)             flags.push('TOP_AI_CITATION');
if (competitorGap.length > 5)                             flags.push('SIGNIFICANT_COMPETITOR_GAP');
```

---

## Output Structure from Transform Node

```javascript
return [{
  json: {
    ai_citations:         normalizedCitations,
    total_queries:        normalizedCitations.length,
    cited_count:          normalizedCitations.filter(c => c.cited).length,
    citation_rate:        citationRate,
    competitor_gap:       competitorGap,
    avg_citation_pos:     avgCitationPos,
    account_name:         accountName,
    date_range:           dateRange
  }
}];
```

---

## DataForSEO Integration

```javascript
// HTTP Request node — DataForSEO AI Overview
// Method: POST
// URL: https://api.dataforseo.com/v3/serp/google/ai_overview/live/advanced
// Auth: Basic (username: DataForSEO login, password: DataForSEO API key)

const payload = [{
  keyword:       query,
  location_code: 2840,    // US — update per client location
  language_code: "en"
}];

// Response fields to extract:
// items[].ai_overview.text          — AI-generated answer text
// items[].ai_overview.references[]  — array of cited URLs
// items[].ai_overview.references[].url — each cited URL
```

---

## Bing Webmaster Tools (CSV Import)

Bing WMT does not have an API yet. Export CSV manually:

1. Go to Bing Webmaster Tools → Performance → Copilot
2. Export as CSV
3. Drop file in `/data/ai-visibility/bing_wmt_YYYY-MM-DD.csv`

Expected CSV columns:
```
Query | Impressions | Citations | Pages Cited | Date
```

n8n Code node to parse:
```javascript
const csvData = $json.fileContent;
const rows    = csvData.split('\n').slice(1);  // skip header

return rows.map(row => {
  const [query, impressions, citations, pagesCited, date] = row.split(',');
  return {
    json: {
      source:      'bing_wmt',
      ai_platform: 'copilot',
      query:       query?.trim(),
      impressions: parseInt(impressions) || 0,
      cited:       parseInt(citations) > 0,
      cited_url:   pagesCited?.trim() || null,
      date_range:  date?.trim()
    }
  };
});
```

---

## Report Section: AI Visibility

Add this section to any existing report (Meta Ads or SEO) when AI visibility data is available:

```
## AI Search Visibility

**Citation Rate:** X% of tracked queries return a citation for [site]
**Avg Citation Position:** X.X
**Top Cited Page:** [URL]

### Queries Where Competitors Are Cited (Not You)
| Query | Competitor Cited | Your Position |
|---|---|---|
| ...  | competitor.com   | Not cited     |

### Top Citation Opportunities
Queries where you rank organically but are not yet cited in AI responses.
These represent the highest-leverage content optimization targets.

### Recommended Actions
- [ ] Update [page] to include more direct, citable answer formats
- [ ] Add FAQ schema to [page] targeting [query]
- [ ] Consolidate [page A] and [page B] — competing for same AI citations
```

---

## Cross-Source Enrichment

When GSC data is also available, compute this overlap:

```javascript
// Find queries where you rank organically but are NOT cited in AI
const organicNotAICited = gscQueries.filter(q =>
  q.position <= 10 &&
  !aiCitations.find(c => c.query === q.query && c.cited === true)
);
// These are your highest-priority GEO (Generative Engine Optimization) targets
```

---

## Validation Rules

Before passing to AI:
- `cited` must be boolean, never string
- `citation_position` must be null (not 0) when `cited` is false
- `ai_platform` must be one of the defined enum values
- `source` must be lowercase string
- `analyzed_at` must be ISO format: `new Date().toISOString()`
- `ai_response_snippet` must be max 200 characters
