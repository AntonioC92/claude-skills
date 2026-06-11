---
name: sw-seo-audit-protocol
description: >
  Defines the structure, scoring, and deliverable format for a Streetwise SEO
  audit. Use this skill whenever an SEO audit is being scoped, executed, or
  delivered — paid one-off audits, client onboarding diagnostics, or
  pre-engagement health checks. Triggers on: SEO audit, technical SEO audit,
  on-page audit, organic audit, SEO health check, GSC audit, content gap
  analysis, site audit, SEO diagnosis, audit my site, ranking analysis,
  organic visibility report. Wraps the existing
  `sw-seo-gsc-ga4-normalization-schema` (data layer) and
  `sw-seo-ai-visibility-tracking` (AI layer) into a sellable deliverable.
---

# Streetwise SEO Audit Protocol
**Productisable as:** $499 one-off audit (Quick Diagnostic) | $999 deep audit (Full Audit) | included in retainer engagements.
**Lead operator:** Antonio
**Deliverable format:** HTML via `sw-reporting-professional-html-document` skill — single self-contained file

---

## Two Tiers

| Tier | Price | Scope | Time to deliver | Use when |
|---|---|---|---|---|
| **Quick Diagnostic** | $499 | 12 checks, 30-day data window, prioritised top 10 fixes | 3 business days | Lead magnet, pre-discovery, low-commitment intro |
| **Full Audit** | $999 | 30+ checks, 12-month data window, full action plan with effort/impact scoring | 7 business days | Pre-retainer scoping, post-migration, suspected algorithm hit |

Pick the tier in the brief. Never run a Full Audit when the prospect agreed to a Quick.

---

## Required Inputs (before audit starts)

- [ ] Domain(s) in scope
- [ ] GSC access — Owner or Full role
- [ ] GA4 access — Editor or Marketer role
- [ ] Crawl access — robots.txt allows Screaming Frog or equivalent
- [ ] Optional: Ahrefs / Semrush API access
- [ ] Optional: list of priority pages / commercial pages
- [ ] Optional: list of known competitors (3–5)

If any required input is missing → STOP and request before kickoff. Don't start a Quick Diagnostic without GSC at minimum.

---

## Audit Sections (all tiers)

### 1. Executive Summary
- Overall health score (0–100)
- Top 3 wins, top 3 issues, top 3 opportunities
- One-paragraph recommendation: keep doing X, fix Y, invest in Z

### 2. Visibility Snapshot (data from GSC + GA4)
Apply `sw-seo-gsc-ga4-normalization-schema` to pull:
- Total clicks, impressions, average position (last 30/90/365d depending on tier)
- Top 10 winning queries
- Top 10 underperforming queries (high impressions, low CTR)
- Top landing pages by organic sessions
- Channel split (organic vs paid vs direct vs referral)

### 3. Technical Health (Full Audit only — abbreviated for Quick)
- Crawlability: robots.txt, sitemap.xml, canonical issues
- Indexation: indexed vs submitted, soft 404s, redirect chains
- Core Web Vitals: LCP, INP, CLS — pass/fail per page template
- Mobile usability
- HTTPS / mixed content
- Structured data validity
- hreflang (if multi-locale)

### 4. On-Page Audit
- Title tag analysis (length, uniqueness, keyword presence)
- Meta description coverage
- H1 / heading structure
- Internal linking depth
- Keyword cannibalization (multiple pages targeting same query)
- Thin content (<300 words on commercial pages)
- Image alt-text coverage

### 5. Content Gap Analysis
- Top organic competitors (from Ahrefs / manual)
- Keywords competitors rank for that we don't
- Content topics we cover that competitors don't (existing strength)
- AI Overview / Copilot citation gaps — apply `sw-seo-ai-visibility-tracking`

### 6. Off-Page Snapshot (Full Audit only)
- Domain Rating / Authority trend
- Top referring domains
- Toxic / spammy backlinks (manual review)
- Anchor text distribution

### 7. Quick Wins (always)
A ranked list of 5–10 fixes that are:
- Implementable in <2 hours each
- Don't require dev resources
- Have measurable impact

Examples: missing meta descriptions on top 20 organic pages; duplicate title tags; broken internal links; outdated content on top-ranking pages; missing structured data on commercial templates.

### 8. Strategic Action Plan
Split into:

| Priority | Effort (hrs) | Estimated Impact | Action |
|---|---|---|---|
| P0 — Critical | … | … | … |
| P1 — High | … | … | … |
| P2 — Medium | … | … | … |
| P3 — Low | … | … | … |

Effort: total hours to complete. Impact: traffic / leads / revenue estimate, with reasoning.

### 9. AI Visibility Layer (always)
Apply `sw-seo-ai-visibility-tracking` to surface:
- Citation rate across Google AI Overviews / Copilot / Perplexity
- Queries where competitors are cited and we're not
- Top AI-citation opportunities (queries we rank organically but aren't cited yet)

This section is what differentiates a Streetwise audit from a generic SEO audit. Always include it.

### 10. Next Steps
- Three discrete bets for the next 30 days (matched to brief.md §8 if it's a retainer onboarding)
- One ask: "Do you want Streetwise to execute these, or hand off to your team?"

---

## Health Score Calculation

Each check returns a score 0–100. Section average → weighted total.

| Section | Weight |
|---|---|
| Technical health | 25% |
| On-page | 20% |
| Content quality / gap | 20% |
| Visibility (GSC trends) | 15% |
| AI visibility | 10% |
| Off-page (Full Audit only) | 10% |

For Quick Diagnostic: redistribute the off-page 10% into Technical (now 30%) and Content (now 25%).

Score bands:
- 80–100: Healthy, focus on growth
- 60–79: Functional, fix Quick Wins + invest in 1–2 strategic plays
- 40–59: At risk, prioritise P0/P1 fixes immediately
- 0–39: Triage mode, full rebuild likely required

---

## Deliverable Format

Final output: a single self-contained HTML file via `sw-reporting-professional-html-document` skill.

File name: `[client]-seo-audit-[YYYY-MM-DD].html`
Saved to: `clients/[client]/outputs/Reports/`

Pairs with a 30-minute walkthrough call. Never deliver an audit without a walkthrough — clients underweight written reports until they're explained out loud.

---

## Time Box (so audits don't bleed margin)

| Tier | Internal hours allowed | Hourly cost cap |
|---|---|---|
| Quick Diagnostic ($499) | ≤6 hours | ≤$83/hr internal load |
| Full Audit ($999) | ≤14 hours | ≤$71/hr internal load |

If an audit exceeds these hours, flag it as scope creep and either stop or escalate to upsell.

---

## What Makes This Audit Sellable (vs. free template audits)

1. AI visibility layer — most agencies don't track AI Overview citations yet
2. Numbers-first impact estimates — "fix this title tag → estimated +120 clicks/month" beats "consider rewriting titles"
3. Action plan with effort × impact scoring — clients can pick what to do
4. 30-min walkthrough included — relationship building, not just delivery
5. Direct path to retainer — every audit ends with the "execute or hand off" ask

---

## Hard Rules

1. Never start an audit without GSC access — guess data is worthless
2. Always include the AI Visibility section, even at Quick tier (it's the differentiator)
3. Always end with a specific next-step ask (don't leave the prospect to decide)
4. Always deliver via HTML (`sw-reporting-professional-html-document`) — never raw markdown to clients
5. Always include the 30-min walkthrough call — it's not optional
6. Time-box: stop work at the cap. Don't deliver $2,000 of effort for a $499 product
7. Pair every Full Audit with `sw-client-onboarding` if it converts to retainer

---

## Integration with Other Skills

- `sw-seo-gsc-ga4-normalization-schema` — data layer for visibility section
- `sw-seo-ai-visibility-tracking` — data layer for AI visibility section
- `sw-reporting-professional-html-document` — output format
- `sw-document-routing` — confirms HTML for audits
- `sw-client-onboarding` — handoff point if audit converts to retainer
- `sw-utm-governance` — referenced when reviewing analytics setup
