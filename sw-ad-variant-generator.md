---
name: sw-ad-variant-generator
description: >
  Defines the Streetwise pipeline for generating 50–100 on-brand ad variants
  from a single creative brief, using AI to draft copy and Figma to export
  visual assets. Use this skill whenever a client needs scaled ad creative
  testing across Meta, Google, or LinkedIn — and any time you'd otherwise
  hand-write 30+ ad variations. Triggers on: ad variants, ad copy at scale,
  CSV ad copy, Figma ad export, creative pipeline, creative testing matrix,
  100 ad variants, headline variations, body copy variations, ad creative
  generation, variant generator, scaled ads. Productisable as a paid
  Streetwise service ("Creative Engine — install + monthly").
---

# Streetwise Ad Variant Generator
**Stage:** Productisable service. Standalone offer ($1,500 install + monthly).
**Lead operator:** Antonio (technical pipeline) | Sean (creative angle brief)
**Outputs:** CSV → Figma → Meta/Google/LinkedIn upload-ready creatives.

---

## Pipeline Overview

```
Creative Brief (input)
    ↓
AI Variant Generation (Claude API or n8n Code node)
    ↓
CSV (one row per variant: headline · body · CTA · angle · audience)
    ↓
Figma Plugin (maps CSV columns to template layers)
    ↓
Exported Frames (one PNG/JPG per row)
    ↓
Human Review Gate (approve / reject per variant)
    ↓
Meta / Google / LinkedIn Upload
```

Each step is one node / one tool. Never combine.

---

## Step 1 — Creative Brief (Input)

A brief.md must exist before any variant generation. Required fields:

```markdown
# [Client] — Variant Brief — [Campaign Name]
- Platform: meta / google / linkedin
- Campaign objective: leads / sales / video views
- Audience(s): list each with one-line ICP description
- Core offer: what they get
- Proof points: 1–3 specific numbers / case studies / quotes
- Voice constraints: brand guidelines link
- Off-limits: claims, terms, themes to avoid
- Variant count target: 30 / 50 / 100
- Angles to test: list 5–10 (problem, solution, proof, contrarian, comparison, etc.)
```

If the brief is missing any of these, STOP and ask. Don't generate variants from a thin brief — they'll all sound the same.

---

## Step 2 — AI Variant Generation

Use Claude (Sonnet) via n8n HTTP Request node OR Anthropic API directly.

### Prompt structure
```
You are generating ad variants for [client] running on [platform] for [audience].

Brief:
[paste full brief]

Generate [N] variants. Each variant must include:
- headline (≤[platform char limit])
- body (≤[platform char limit])
- CTA (one of: [allowed CTAs for platform])
- angle (one of: [angles from brief])
- audience (which audience from brief this variant targets)

Output ONLY a CSV with columns: variant_id, headline, body, cta, angle, audience.
No preamble, no markdown, no explanation. CSV only.

Voice rules: [paste from brand guidelines]
Banned phrases: [paste from brand]
```

### Platform character limits (use exactly)

| Platform | Headline | Primary text / body | CTA |
|---|---|---|---|
| Meta (Feed) | 27 chars (mobile-safe) | 125 chars (mobile-safe) | Pick from Meta CTA list |
| Meta (Reels) | 40 chars | 72 chars | Pick from Meta CTA list |
| Google Search RSA | 30 chars × 15 | 90 chars × 4 description lines | Auto / Sitelinks |
| LinkedIn Sponsored | 200 chars headline | 600 chars intro | Pick from LinkedIn CTA list |

---

## Step 3 — CSV Output Schema

The CSV is the contract between Claude and Figma. Schema must be exact:

```csv
variant_id,headline,body,cta,angle,audience,platform,client
V001,"Headline text","Body text","Learn More","problem","SaaS founders","meta","ClientName"
V002,"Headline text","Body text","Sign Up","proof","SaaS founders","meta","ClientName"
```

### Validation rules
- variant_id: V### sequential
- No commas inside cells (use proper CSV escaping with quotes)
- No newlines inside cells
- Headline ≤ platform limit
- Body ≤ platform limit
- CTA from allowed list per platform
- Angle from brief.md angles list
- Audience from brief.md audiences list

If any row fails validation, drop it and regenerate that row only — don't reject the whole batch.

---

## Step 4 — Figma Plugin Mapping

Figma master template requirements:
- Named text layers exactly matching CSV column names: `{headline}`, `{body}`, `{cta}`
- Image layer placeholder if creative includes a hero image
- Variants in Figma's native variants system: one per audience or platform format

### Plugin choice
- **Bannerify** (Figma plugin) — paid, robust, used in production
- **TextRunner** — free, simpler, fewer features
- **Custom Figma plugin** — long-term route; build on top of Figma REST API

### Output
One PNG (or JPG, or MP4 for video) per CSV row, named: `[client]_[campaign]_V###.png`

Export resolution per platform:
- Meta Feed: 1080×1080 (square) or 1080×1350 (4:5)
- Meta Reels: 1080×1920
- LinkedIn: 1200×627 (single image) or 1080×1080 (square)
- Google Display: 1200×628 + 1080×1080 + 300×600

---

## Step 5 — Human Review Gate

Mandatory. Never auto-upload to Meta/Google/LinkedIn.

Review checklist per variant:
- [ ] Copy matches brief voice
- [ ] No banned phrases / off-limits topics
- [ ] No claims that need legal review
- [ ] Image renders correctly (no clipped text, no cropping issues)
- [ ] CTA matches landing page promise
- [ ] Tracking parameters in destination URL

Reject reason codes (for retraining the prompt):
- COPY_OFF_BRAND
- CLAIM_RISK
- VISUAL_BROKEN
- CTA_MISMATCH
- DUPLICATE_OF_VARIANT_X

---

## Step 6 — Upload

### Meta
- Use Meta Marketing API via n8n
- Apply `sw-meta-business-manager-protocol` rules
- Always create as `status: PAUSED`
- Pair with `sw-approval-gate-pattern` (when built) for Slack approve/reject before flip to ACTIVE

### Google Ads
- Use Google Ads API via n8n
- RSAs accept multiple headlines/descriptions per ad — bundle the variants

### LinkedIn
- Campaign Manager API
- Sponsored Content + Lead Gen Forms only at this stage

---

## Memory Layer (avoid repeats across cycles)

After every cycle, log results to a Google Sheet:

| variant_id | spend | clicks | conversions | cpa | ctr | result_tag | reused_angle |
|---|---|---|---|---|---|---|---|

`result_tag`: WINNER / RUNNER_UP / KILLED / TEST
`reused_angle`: yes / no

Before next cycle, feed this sheet to Claude as context. Prompt addition:

> Avoid repeating angles already labelled WINNER in the past 30 days unless explicitly testing fatigue. Avoid repeating any angle labelled KILLED in the past 90 days.

This is what makes the system iterative, not generative. Without the memory layer, every cycle drifts back to the same angles.

---

## Pricing Architecture (productised offer)

| Tier | Variants/month | Channels | Includes | Price |
|---|---|---|---|---|
| Install (one-off) | n/a | n/a | Brief workshop, Figma template, n8n pipeline, first 50 variants | $1,500 |
| Monthly Light | 50 | 1 platform | Variant gen + review + upload | $750/mo |
| Monthly Standard | 100 | 2 platforms | Above + creative angle workshop | $1,500/mo |
| Monthly Pro | 200 | 3 platforms | Above + memory-layer dashboard | $2,500/mo |

(Prices indicative — confirm with Sean before quoting.)

---

## Hard Rules

1. Never run variant generation without a complete brief.md (all fields)
2. Never auto-upload to ad platforms — human review gate is mandatory
3. Always validate CSV against platform char limits BEFORE Figma export
4. Always log results to memory sheet after each cycle
5. Always apply `sw-meta-business-manager-protocol` for Meta uploads
6. Never reuse a variant labelled KILLED inside 90 days
