---
name: data-normalization-schema
description: Defines the shared campaign data schema, computed KPI formulas, and performance flag logic used across all platform modules. Use this skill whenever building or modifying a Transform node for any ad platform (Meta, Google Ads, LinkedIn).
---

# SKILL: Campaign Data Normalization Schema
**Version:** 1.0 | **Project:** AI Marketing Intelligence Engine

## Master Normalized Schema
```javascript
{
  platform:          string,   // "meta" | "google" | "linkedin"
  client_name:       string,
  campaign_id:       string,
  campaign_name:     string,
  spend:             float,
  clicks:            int,
  impressions:       int,
  reach:             int,
  frequency:         float,
  conversions:       int,
  revenue:           float,
  ctr:               float,
  cpc:               float,
  cpm:               float,    // computed
  cpa:               float,    // computed, null if 0 conversions
  cvr:               float,    // computed
  roas:              float,    // computed
  performance_flags: array,
  date_range:        string,
  analyzed_at:       string    // ISO timestamp
}
```

## Computed KPIs
```javascript
const cpa  = conversions > 0 ? +(spend / conversions).toFixed(2) : null;
const cvr  = clicks > 0      ? +((conversions / clicks) * 100).toFixed(2) : 0;
const roas = spend > 0       ? +(revenue / spend).toFixed(2) : 0;
const cpm  = impressions > 0 ? +((spend / impressions) * 1000).toFixed(2) : 0;
```

## Performance Flags
```javascript
const flags = [];
if (frequency >= 3.5)                flags.push('HIGH_FREQUENCY_FATIGUE');
if (ctr < 0.8 && impressions > 5000) flags.push('LOW_CTR');
if (cpa !== null && cpa > 100)       flags.push('HIGH_CPA');
if (roas > 0 && roas < 1.5)          flags.push('LOW_ROAS');
if (cvr < 1 && clicks > 200)         flags.push('LOW_CVR');
```

## Output Structure
```javascript
return [{ json: { campaigns: normalizedCampaigns, total_campaigns: normalizedCampaigns.length } }];
```

## Validation Rules
- `spend` must be float, never string
- `cpa` must be null (not 0) when conversions = 0
- `performance_flags` must be array (empty array if no flags)
- `analyzed_at` must be ISO format
- `platform` must be lowercase string