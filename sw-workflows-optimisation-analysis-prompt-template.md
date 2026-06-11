---
name: sw-workflows-optimisation-analysis-prompt-template
description: >
  Defines the standard AI prompt structure, report sections, and Build Report
  node logic used across all platform modules in the AI Marketing Intelligence
  Engine. Use this skill whenever building or modifying the Build AI Prompt
  node, Claude AI Analysis node, or Build Report node for any platform
  workflow (Meta Ads, Google Ads, LinkedIn Ads). Triggers on: build AI prompt
  node, build report node, analysis node, AI prompt template, weekly ad report,
  performance report generator, campaign analysis report, report sections.
---

# SKILL: AI Analysis Prompt & Report Template
**Version:** 1.0 | **Project:** AI Marketing Intelligence Engine

## Build AI Prompt Node Structure
```javascript
const campaigns = $json.campaigns;
const totalCampaigns = $json.total_campaigns;
const reportDate = new Date().toLocaleDateString('en-US', {
  weekday: 'long', year: 'numeric', month: 'long', day: 'numeric'
});

const campaignTable = campaigns.map((c, i) => {
  const flagStr = c.performance_flags.length > 0
    ? c.performance_flags.join(', ')
    : 'None';
  return `Campaign ${i + 1}: ${c.campaign_name}
  Spend: $${c.spend} | Clicks: ${c.clicks} | Impressions: ${c.impressions}
  CTR: ${c.ctr}% | CPC: $${c.cpc} | CPM: $${c.cpm}
  Conversions: ${c.conversions} | CPA: ${c.cpa !== null ? '$' + c.cpa : 'N/A'}
  CVR: ${c.cvr}% | ROAS: ${c.roas}x | Revenue: $${c.revenue}
  Frequency: ${c.frequency} | Reach: ${c.reach.toLocaleString()}
  Flags: ${flagStr}`;
}).join('\n\n');
```

## Required Report Sections (in this order)
1. Executive Summary
2. Campaign Performance Table (🟢 Strong | 🟡 Moderate | 🔴 Weak)
3. Top Performing Campaigns
4. Underperforming Campaigns
5. Creative Fatigue Analysis
6. Audience Optimization Opportunities
7. Budget Reallocation Plan
8. Creative Testing Ideas (Format | Angle | Hypothesis)
9. 7 Day Action Plan
10. Risk Flags
11. Report Metadata

## Build Report Node Structure
```javascript
const aiResponse = $json;
const reportDate = $('Build AI Prompt').first().json.report_date;
const campaigns  = $('Build AI Prompt').first().json.campaigns;

let aiContent = '';
if (aiResponse.content && aiResponse.content.length > 0) {
  aiContent = aiResponse.content
    .filter(block => block.type === 'text')
    .map(block => block.text)
    .join('\n');
} else {
  aiContent = 'ERROR: No AI response received.';
}

const totalSpend       = campaigns.reduce((sum, c) => sum + c.spend, 0).toFixed(2);
const totalClicks      = campaigns.reduce((sum, c) => sum + c.clicks, 0);
const totalConversions = campaigns.reduce((sum, c) => sum + c.conversions, 0);
const avgCPA = totalConversions > 0
  ? (totalSpend / totalConversions).toFixed(2)
  : 'N/A';

const dateStamp = new Date().toISOString().split('T')[0];
const fileName  = `Ads_Report_${dateStamp}.md`;
```

## Claude API Settings
- Model: `claude-sonnet-4-20250514`
- Max tokens: `4096`
- API version: `2023-06-01`
