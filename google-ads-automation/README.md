# Google Ads Automation

Skills for working with Google Ads — building campaigns through the web UI, reporting on performance via the API, surfacing optimization recommendations, and (eventually) bulk-generating creative variants. Each skill below is a standalone protocol Claude can invoke; together they cover the end-to-end Google Ads workflow Streetwise runs for clients.

## Skills in this category

### Active

#### [`google-ads-browser-campaign-build/`](./google-ads-browser-campaign-build/)

Build, edit, and verify Google Ads campaigns through the web UI when API-based creation isn't viable (e.g. Performance Max signal upload, sitelinks, callouts, or single-RSA fixes). Codifies the Angular SPA interaction patterns, pre-build keyword coverage audit, session recovery, and mandatory post-save verification protocol that emerged from the Kubiieo Campaign #2 build (June 2026).

**Use when:** Building a new campaign, fixing an existing RSA, auditing live ad copy against a spec, or onboarding a new Google Ads client.

### Planned (not yet built)

- **`google-ads-performance-engine/`** — n8n workflow that pulls GAQL campaign metrics nightly, normalizes them via the shared paid-campaigns schema, runs Claude AI analysis, and ships a polished report to client Drive. Mirrors the Meta Ads Performance Engine architecture. Depends on: Streetwise MCC + developer token (in progress with Sean).
- **`google-ads-optimisation-recommender/`** — Layered on top of the Performance Engine: mines the search terms report for negative-keyword candidates, surfaces RSA asset-coverage gaps, audits bid strategies, and proposes weekly action items.
- **`google-ads-bulk-builder/`** — API-based campaign authoring for cases where Google Ads Editor + browser-build doesn't scale (50+ ad variants per `sw-ad-variant-generator`, multi-client template rollout). Lowest priority — only built once the browser-build skill stops being sufficient.

## Conventions

- All skills in this category share the `sw-paid-campaigns-data-normalization-schema` for cross-platform reporting consistency (same schema as `meta-ads-automation/` once that lands).
- AI analysis output follows `sw-workflows-optimisation-analysis-prompt-template`.
- UTM and tracking governance follows `sw-utm-governance`.
- All client work follows the Phase 1 → 2 → 3 → 4 sequencing model (pre-build → build → save+verify → postmortem) defined in the browser-campaign-build skill — even API-based skills inherit the verify+postmortem habit.

## When to use which skill (decision tree)

```
Need to build or edit Google Ads campaign elements?
│
├── Need to write/edit campaigns, ad groups, RSAs, sitelinks?
│   │
│   ├── One-off / small batch / no API yet?
│   │   → google-ads-browser-campaign-build (browser UI)
│   │
│   └── 50+ variants / multi-client template?
│       → google-ads-bulk-builder (when built; currently use Editor + CSV)
│
├── Need to read performance data?
│   → google-ads-performance-engine (when built; currently use UI exports or Looker Studio)
│
└── Need to surface optimization opportunities?
    → google-ads-optimisation-recommender (when built)
```

## Owner

Antonio Caruso ([caruso.martech@gmail.com](mailto:caruso.martech@gmail.com)) — Streetwise Consultancy.

Co-maintained with Sean (MCC + AU client operations).
