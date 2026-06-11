---
name: sw-client-onboarding
description: >
  Standardised onboarding protocol for every new Streetwise Consultancy client.
  Use this skill the moment a new client is signed — Upwork win, direct
  contract, referral, or partnership. Triggers on: new client, onboard a
  client, client kickoff, starting with a client, add a new client, client
  setup, won an Upwork job, signed a client, client onboarding, kickoff plan,
  first week with client, brief.md, set up a client folder, what do I need
  from the client. Produces the access checklist, file structure, kickoff
  agenda, and 30-day plan template so no step is forgotten.
---

# Streetwise Client Onboarding Protocol
**Owner:** Joint (Sean for sales handover, Antonio for delivery setup)
**Trigger event:** Contract signed OR Upwork milestone funded.
**Time-to-execute:** ≤ 48 hours from signature.

---

## Step 0 — Decide Lead Operator

| Service line | Default lead |
|---|---|
| LinkedIn outbound, sales-led, live events | Sean |
| Paid media, tracking, automation, SEO, AI workflows | Antonio |
| Mixed engagement | Both, named in brief.md |

State the lead operator in the kickoff Slack message. Only one person owns the client weekly — even if both deliver work.

---

## Step 1 — Access Checklist (request from client BEFORE kickoff call)

Send this list within 24h of signing. If any item blocks, flag it on the kickoff call.

### Always required
- [ ] Company website + landing page URLs in scope
- [ ] Logo files + brand guidelines (colours, fonts, voice)
- [ ] Business goals and KPIs in writing
- [ ] Target audience / ICP profile
- [ ] Decision-maker contact + secondary contact
- [ ] Slack channel or preferred async comms (we propose Slack)

### If paid media is in scope
- [ ] **Meta Business Manager partner access** — apply `sw-meta-business-manager-protocol`
- [ ] Google Ads access — request MCC linking (Streetwise MCC ID: TBD)
- [ ] LinkedIn Ads — request agency access via Campaign Manager
- [ ] Existing pixel / CAPI / conversion event audit
- [ ] Historical performance export (last 90 days minimum)

### If tracking / automation is in scope
- [ ] GA4 access — Editor or Marketer role minimum
- [ ] GTM access — Edit or Publish role
- [ ] CRM access (HubSpot / Salesforce / GHL / other)
- [ ] Email platform access (Brevo / Klaviyo / Mailchimp)
- [ ] DNS / domain access if email auth changes are needed

### If SEO is in scope
- [ ] GSC (Google Search Console) access — Owner or Full
- [ ] Sitemap URL
- [ ] Existing keyword list / ranking data
- [ ] CMS access (WordPress / Webflow / Shopify) at Editor level

---

## Step 2 — Internal File Setup

Create the client folder under `clients/` in the streetwise-consultancy repo:

```
clients/[Client Name]/
├── brief.md                    ← MUST be filled before delivery starts
├── received/                   ← anything the client sent us
│   ├── brand/
│   ├── historical-data/
│   └── access-credentials.md   ← URLs, account IDs (NEVER secrets)
└── outputs/
    ├── strategy/
    ├── ads/
    ├── reports/
    ├── automation/
    └── meetings/
```

**Never commit credentials, tokens, or passwords to git.** Use n8n Credentials or 1Password references only.

---

## Step 3 — brief.md Template

Create `brief.md` from this template before the kickoff call. Fill it during the call.

```markdown
# [Client Name] — Engagement Brief
**Signed:** YYYY-MM-DD | **Lead operator:** Sean / Antonio | **Service tier:** Starter / Growth / Scale

## 1. Business Snapshot
- What they sell:
- Who buys it (ICP):
- Average deal value:
- Sales cycle length:
- Geography:

## 2. Goals & KPIs (90-day)
- Primary KPI:
- Secondary KPI:
- North-star revenue/lead target:
- Reporting cadence: weekly / biweekly / monthly

## 3. Channel Mix (in scope)
- Paid media: Meta / Google / LinkedIn / TikTok / Pinterest
- Outbound: LinkedIn / cold email
- SEO / content: yes / no
- Automation: yes / no
- Live events: yes / no

## 4. Budget
- Monthly retainer (Streetwise):
- Monthly ad spend (client):
- Tools/subscriptions covered by:

## 5. Constraints
- Off-limits topics:
- Compliance / legal flags:
- Brand guideline link:

## 6. Decision Process
- Who approves campaigns:
- Approval SLA:
- Communication preference: Slack / email / weekly call

## 7. Existing Stack
- CRM:
- Analytics:
- Ad platforms:
- Email:
- Other:

## 8. Initial 30-Day Plan
Three concrete bets, each with a measurable outcome.
1.
2.
3.
```

---

## Step 4 — Kickoff Call Agenda (45–60 min)

Run this exact agenda. Record the call (with consent).

1. **Goals re-confirmation (10 min)** — read goals back; client confirms or corrects
2. **Audience deep-dive (10 min)** — ICP, objections, what they wish prospects knew
3. **Historical performance review (10 min)** — what worked, what didn't, why
4. **Channel + budget split (10 min)** — confirm scope and money flow
5. **Reporting cadence + communication (5 min)** — Slack channel + weekly time slot
6. **Next 14 days: who does what (10 min)** — fill brief.md §8 live

Output: brief.md fully populated + Loom recap sent within 24h.

---

## Step 5 — First-Week Internal Setup (Antonio + Sean)

Within 5 business days:
- [ ] All access from Step 1 confirmed working
- [ ] GA4/GTM audit done if paid media in scope (use `sw-seo-gsc-ga4-normalization-schema` for data layer)
- [ ] UTM convention applied — apply `sw-utm-governance` (when built) or current standard
- [ ] Reporting workflow duplicated in n8n for this client (use `sw-workflows-architecture-build-protocol`)
- [ ] Client added to multi-client router with active=true
- [ ] First weekly report scheduled
- [ ] Kickoff Loom + welcome doc sent

---

## Step 6 — Day-30 Review

Hold a 30-minute internal review on day 30:
- Did we hit the three bets in brief.md §8?
- What's the new bet for days 31–60?
- Is the client's reporting/comms cadence working?
- Any expansion (new service line) signals?

If yes to expansion → trigger upsell conversation in week 5.

---

## Hard Rules

1. Never start delivery work before brief.md §1–§7 are filled
2. Never store client credentials in git (only platform IDs and account names)
3. Never accept access at lower than the minimum role per service line
4. Never skip the kickoff call, even for "easy" projects — that's where scope drift gets prevented
5. Always send the kickoff Loom within 24h of the call
