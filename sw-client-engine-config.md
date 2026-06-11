---
name: sw-client-engine-config
description: >
  Defines per-vertical KPI thresholds and AI-prompt context overrides used by
  the n8n reporting engine so one workflow serves many client types without
  rebuilding. Apply this skill any time a new client is being onboarded, a
  client is being assigned to a vertical, an existing vertical's benchmarks
  need updating, or AI report output is reading wrong for a given industry.
  Triggers on: client engine config, vertical config, KPI thresholds, industry
  benchmarks, performance flag thresholds, hospitality benchmarks, B2B SaaS
  benchmarks, DTC benchmarks, events benchmarks, vertical override, AI prompt
  context per vertical, custom thresholds, client KPI mapping, multi-client
  router config, per-client benchmark, edelweiss config, dp gates config,
  beer festival config, alifeplus config.
---

# Streetwise Client Engine Config
**Purpose:** Override the default thresholds in `sw-paid-campaigns-data-normalization-schema` and inject industry-specific context into `sw-workflows-optimisation-analysis-prompt-template` based on the client's vertical. Without this, a hospitality client and a B2B SaaS client get the same benchmark — which is wrong for both.

---

## Where This Lives in the Architecture

- The **Multi-Client Router** (n8n workflow) reads the `clients` sheet
- Each client row has a `vertical` column (one of the values defined below)
- Before triggering the platform engine, the router loads this skill's vertical config and overrides default thresholds + injects the AI prompt context block

---

## Default Thresholds (apply when no vertical is set)

From `sw-paid-campaigns-data-normalization-schema`:
| Flag | Trigger |
|---|---|
| HIGH_FREQUENCY_FATIGUE | Frequency ≥ 3.5 |
| LOW_CTR | CTR < 0.8% with > 5,000 impressions |
| HIGH_CPA | CPA > $100 |
| LOW_ROAS | ROAS < 1.5x |
| LOW_CVR | CVR < 1% with > 200 clicks |

---

## Vertical Configurations

### `hospitality`
**Examples:** Edelweisshütte, restaurants, bars, lodges, hotels
**Primary KPI:** Bookings or table covers
**Secondary KPI:** Cost per booking
**Sales cycle:** Same-day to 2 weeks
**Audience pattern:** Local + tourist; geo-radius targeting; seasonal peaks

```javascript
thresholds: {
  HIGH_FREQUENCY_FATIGUE: 4.5,    // higher tolerance — local audiences are small
  LOW_CTR:                1.0,    // higher expectation — visual ads work well
  HIGH_CPA:               40,     // bookings cost less than B2B leads
  LOW_ROAS:               2.5,    // lower margins, need stronger ROAS
  LOW_CVR:                2.0     // higher CVR expected (warm intent)
}
```

**AI prompt context block:**
> The client is a hospitality business. Focus optimisation on booking volume, cost per booking, and seat utilisation. Local audience saturation is a key risk — recommend audience refresh sooner than for national brands. Seasonal demand patterns (holidays, weekends, weather) should drive bid pacing. Creative should emphasise atmosphere, food/experience photography, and social proof (reviews, repeat customers).

---

### `events`
**Examples:** Dublin Beer Festival, Jobbio Career Fair, summits, conferences
**Primary KPI:** Tickets sold OR registrations
**Secondary KPI:** Cost per ticket / registration
**Sales cycle:** Days to weeks; sharp peak before event date
**Audience pattern:** Time-bound, high urgency closer to event date

```javascript
thresholds: {
  HIGH_FREQUENCY_FATIGUE: 6.0,    // tolerable as event approaches — urgency wins
  LOW_CTR:                1.2,    // creative needs to drive clicks at scale
  HIGH_CPA:               25,     // tickets are usually low-ticket
  LOW_ROAS:               5.0,    // events have strong ROAS when run well
  LOW_CVR:                3.0     // landing pages should convert hot
}
```

**AI prompt context block:**
> The client is an events business with a fixed event date. Prioritise tempo: spend should accelerate as the event approaches. Frequency tolerance increases in the final 7 days because urgency works. Recommend retargeting heavily in the final 14 days. Creative should emphasise scarcity (sold-out warnings, limited tickets) and FOMO (last year's highlights, attending names).

---

### `b2b-saas`
**Examples:** SaaS companies, B2B platforms, enterprise software
**Primary KPI:** MQLs (or SQLs if pipeline-tracking is mature)
**Secondary KPI:** Cost per MQL → cost per opportunity → cost per closed deal
**Sales cycle:** 30–180 days, often longer
**Audience pattern:** LinkedIn-led, ABM possible; intent-based targeting

```javascript
thresholds: {
  HIGH_FREQUENCY_FATIGUE: 3.0,    // lower — saturating an ABM list fast is fine
  LOW_CTR:                0.6,    // LinkedIn benchmarks are lower than Meta
  HIGH_CPA:               250,    // CPL of $100–500 is normal for B2B
  LOW_ROAS:               null,   // ROAS unreliable; use pipeline value instead
  LOW_CVR:                4.0     // lead-gen forms should convert higher
}
```

**AI prompt context block:**
> The client is B2B SaaS with a long sales cycle. ROAS is a misleading metric here — focus on MQL volume, lead quality (job titles, company size), and pipeline velocity. LinkedIn Lead Gen Forms typically beat landing pages for top-of-funnel; recommend testing both for any new audience. Frequency tolerance is lower because B2B audiences are smaller and saturate faster. Always cross-reference cost-per-lead against typical deal value (e.g. $5K ACV needs <$500 CAC to be viable).

---

### `dtc-ecommerce`
**Examples:** Alifeplus, consumer goods, subscription products
**Primary KPI:** First-purchase CAC
**Secondary KPI:** Blended CAC, CVR, repeat-purchase rate
**Sales cycle:** Same-session to 7 days
**Audience pattern:** Lookalikes, interest-based, retargeting heavy

```javascript
thresholds: {
  HIGH_FREQUENCY_FATIGUE: 4.0,    // standard for cold prospecting
  LOW_CTR:                1.5,    // visual product ads should perform
  HIGH_CPA:               40,     // varies — overridden per AOV
  LOW_ROAS:               2.0,    // first-purchase ROAS — true ROAS is LTV-based
  LOW_CVR:                1.5     // standard ecommerce CVR floor
}
```

**AI prompt context block:**
> The client is DTC ecommerce. Differentiate new customer CAC from blended CAC — blended hides which acquisition source is profitable vs subsidised by retention revenue. Recommend creative testing with single-variable structure (one variable per test). On retention, flag opportunities for post-purchase flows, winback sequences, and replenishment triggers via Klaviyo. Subscription products should be evaluated against payback period, not month-1 ROAS.

---

### `education-coaching`
**Examples:** Chefs Office Academy, online courses, high-ticket coaching
**Primary KPI:** Leads (top of funnel) → Sales calls (mid) → Closed deals (bottom)
**Secondary KPI:** Cost per lead, cost per sales call
**Sales cycle:** 7–30 days
**Audience pattern:** Interest + lookalike + retargeting; lead magnets work well

```javascript
thresholds: {
  HIGH_FREQUENCY_FATIGUE: 4.0,
  LOW_CTR:                1.0,
  HIGH_CPA:               60,     // CPA on a lead, not a sale
  LOW_ROAS:               null,   // ROAS misleading; use cost-per-call instead
  LOW_CVR:                2.5
}
```

**AI prompt context block:**
> The client is an education/coaching business with a multi-step funnel. Recommend isolating lead capture into a dedicated low-friction campaign (lead magnet → ebook download or quiz) separate from purchase campaigns. Purchase campaigns retarget the warm pool the lead campaign generates. Creative should be written in the language of the trade — not generic "transform your career" copy. Reference: Chefs Office Academy structural change reduced top-of-funnel cost by >90% by isolating lead capture.

---

### `professional-services`
**Examples:** Consulting, agencies, law firms, accounting
**Primary KPI:** Booked discovery calls
**Secondary KPI:** Cost per call, call-to-client conversion rate
**Sales cycle:** 14–60 days
**Audience pattern:** LinkedIn-heavy; ABM common; outbound + inbound mix

```javascript
thresholds: {
  HIGH_FREQUENCY_FATIGUE: 3.0,
  LOW_CTR:                0.8,
  HIGH_CPA:               150,    // discovery calls are higher-cost than ebooks
  LOW_ROAS:               null,
  LOW_CVR:                3.0
}
```

**AI prompt context block:**
> The client is a professional services firm. Discovery call quality matters more than call volume — recommend tracking show-up rate and call-to-client conversion alongside cost-per-call. LinkedIn outbound (Sean's domain) should be evaluated separately from paid; pair with `sw-content-voice-sean` for outbound copy patterns. Recommend retargeting with case study creative, not generic credentials.

---

### `local-trades`
**Examples:** DP Gates, electricians, plumbers, trades, local installers
**Primary KPI:** Quote requests / lead form submissions
**Secondary KPI:** Cost per quote, quote-to-job conversion rate
**Sales cycle:** 1–14 days
**Audience pattern:** Geo-tight, intent-driven (Google Search heavy)

```javascript
thresholds: {
  HIGH_FREQUENCY_FATIGUE: 5.0,    // small geo audiences saturate fast — accept it
  LOW_CTR:                3.0,    // search ads should have high CTR
  HIGH_CPA:               80,     // quotes are mid-cost
  LOW_ROAS:               4.0,    // job tickets are high; ROAS should be strong
  LOW_CVR:                5.0     // quote forms should convert hot
}
```

**AI prompt context block:**
> The client is a local trades business with geographic constraints. Recommend Google Search heavy mix over social (intent-driven). Native lead-gen forms typically outperform landing-page redirects for trades because friction kills mobile conversions. Reference: DP Gates closed a $1,995 high-ticket sale via Meta native lead gen — lead forms beat external landing pages for high-ticket B2C/B2B trades.

---

### `real-estate-investment`
**Examples:** Appraiva, property platforms, real-estate SaaS
**Primary KPI:** Subscriptions / signups
**Secondary KPI:** CAC, LTV/CAC ratio, demo show rate
**Sales cycle:** 7–30 days; demo-driven
**Audience pattern:** Investor-targeted, often US/AU geo

```javascript
thresholds: {
  HIGH_FREQUENCY_FATIGUE: 4.0,
  LOW_CTR:                1.0,
  HIGH_CPA:               300,    // higher because LTV justifies it (~$5K)
  LOW_ROAS:               null,   // use LTV/CAC ratio instead
  LOW_CVR:                1.5
}
```

**AI prompt context block:**
> The client is a real-estate / investment-tech business with an LTV around $5K. Evaluate against LTV:CAC ratio (target 4:1+). Demo show rate is the leading indicator of close rate — a low show rate (<50%) means leads are unqualified or mis-set. Recommend pixel/tracking validation as the first action since attribution drift is common. Reference: Appraiva's CAC moved from $680 to $1,200 month-over-month with stable spend — a structural funnel issue, not a creative one.

---

## How to Add a New Client to a Vertical

1. In `clients` sheet (Multi-Client Router), set `vertical` column to one of the values above
2. If client doesn't fit cleanly, pick the closest and override individual thresholds in a `vertical_override` column (JSON):
   ```json
   {"HIGH_CPA": 150, "LOW_CVR": 1.0}
   ```
3. AI prompt context auto-injects from the matching vertical block above
4. Document any override reasoning in `clients/[name]/brief.md`

---

## How to Add a New Vertical

If a client genuinely doesn't fit any vertical:
1. Add a new section to this skill following the same template (Examples → Primary KPI → Thresholds → AI prompt context block)
2. Update the Multi-Client Router to recognise the new value
3. Run a 30-day calibration: track whether the new vertical's thresholds correctly flag issues without over-triggering
4. Adjust thresholds based on real campaign data, not guesses

---

## Hard Rules

1. Never run a client through the engine without a vertical assigned (default thresholds are intentionally conservative — they will under-flag for some verticals and over-flag for others)
2. Always review thresholds quarterly against actual client performance data
3. Always document threshold overrides in the client's `brief.md`
4. Never invent a new vertical without adding it to this skill first
5. AI prompt context blocks are not optional — they're what make the report read like industry-specific advice rather than generic ad analysis
6. When in doubt between two verticals, pick the one with the more conservative thresholds (will over-flag rather than under-flag)

---

## Integration with Other Skills

- `sw-paid-campaigns-data-normalization-schema` — defines the default thresholds this skill overrides
- `sw-workflows-optimisation-analysis-prompt-template` — receives the AI prompt context block per vertical
- `sw-workflows-architecture-build-protocol` — Multi-Client Router architecture
- `sw-client-onboarding` — vertical assignment happens at onboarding
