---
name: sw-proposal-format-router
description: >
  Routes every proposal request to the correct length, structure, and voice.
  Use this skill whenever a proposal is being drafted at Streetwise — Upwork
  applications, post-discovery direct proposals, partnership pitches, or
  retainer renewals. Triggers on: write a proposal, draft a proposal, Upwork
  proposal, Upwork application, respond to this Upwork job, post-call
  proposal, post-discovery proposal, consultative proposal, retainer
  proposal, partnership proposal, scope of work, SOW, send a pitch. Layered
  on top of `sw-document-routing` (which picks file format) — this skill picks
  CONTENT shape and voice.
---

# Streetwise Proposal Format Router

`sw-document-routing` decides Google Doc vs HTML.
`sw-content-voice-antonio` is Antonio's voice (direct client only — explicitly NOT for Upwork).
This skill decides **which template, which length, which voice** before either of those run.

---

## Step 1 — Identify Proposal Type

Ask the user (only if not obvious from context):
> Is this an Upwork application, a post-discovery direct-client proposal, a partnership pitch, or a renewal?

| Type | Length | Voice | Format | Lead operator |
|---|---|---|---|---|
| **Upwork application** | 250–400 words | `sw-content-voice-sean` | In-platform text | Sean |
| **Direct client (post-discovery)** | 4–8 pages | `sw-content-voice-antonio` | Google Doc → see `sw-document-routing` | Antonio |
| **Partnership pitch** | 2–4 pages | Joint voice (Sean intro + Antonio depth) | Google Doc | Joint |
| **Retainer renewal** | 2 pages | Whoever owns the account | Google Doc | Account owner |

If unclear → STOP and ask. Do not default.

---

## Step 2A — Upwork Application (250–400 words)

### Structure (no headers, prose flow)
1. **Mirror line** — open by repeating the specific situation from the job post in your own words. Never "I'm excited to apply."
2. **Direct relevance hook** — name the closest Streetwise case study with numbers (Beer Festival €72K from €4.6K, Career Fair €135K from €30K, SaaS CPL £149→£34.70). Pick the one closest to the prospect's situation.
3. **Plan in 3 lines** — "If we worked together, week 1 we'd…, week 2 we'd…, by day 30 you'd see…"
4. **Quick credibility** — one line on Streetwise (full-stack agency, EU + AU coverage, 95% retention).
5. **One closing question** — qualifying ask. Never "let me know if you'd like to chat."

### Hard rules
- Word count: 250–400. Hard ceiling 400.
- No bullet points (Upwork formatting strips them anyway).
- No "I'm excited", "I'd love to", "I hope this finds you well".
- No portfolio links unless the job post explicitly asks for them.
- Always include one specific number from a relevant case study.
- Never mention "agency" if the post is hostile to agencies — say "two-person operator team" instead.
- Match the prospect's tone: if they're casual, soften; if formal, stay precise.

### Voice
Use `sw-content-voice-sean` (when built). Until then, use this register:
- Direct, confident, no fluff
- Frame yourself as the practitioner who runs the system, not a salesperson selling it
- One concrete outcome > three abstract claims

---

## Step 2B — Direct Client Proposal (4–8 pages, post-discovery)

### Structure
1. **Executive Summary** — what they told us their problem is, in our words (1 short page)
2. **Situation Analysis** — diagnosis of the gap, naming specifics (1 page)
3. **Strategy** — channel mix, audience, funnel logic (1–2 pages)
4. **Deliverables & Timeline** — what we ship, by when (1 page; tabular OK)
5. **Investment** — pricing tier + what's included (½ page)
6. **Why Streetwise** — 2–3 case studies with numbers, matched to their industry (½–1 page)
7. **Next Steps** — one clear ask: sign, schedule kickoff, etc. (¼ page)

### Hard rules
- Use Antonio's voice via `sw-content-voice-antonio` skill — diagnostic, results-anchored
- Numbers in every claim. No "great results", "industry-leading", "best-in-class"
- Never use this format for Upwork. The voice is wrong for that channel.
- Always include 2–3 numbered case studies, matched by industry where possible
- Pricing presented as a tier (Starter/Growth/Scale) with what's included, not as a quote-only line
- Never send without internal review by the operator who didn't write it

### Format
Route through `sw-document-routing`:
- Pre-signature → Google Doc
- Internal version for record → also save as HTML in `clients/[name]/outputs/Proposal/`

---

## Step 2C — Partnership Pitch (2–4 pages)

### Structure
1. Why us, why you, why now (½ page)
2. What we're proposing — split of revenue / referral / co-marketing (½–1 page)
3. What each side brings (1 page)
4. First 90 days — three concrete bets (½ page)
5. Asks + decision-making process (¼ page)

### Hard rules
- Joint sign-off required (Sean + Antonio) before sending
- Always reference specific deal flow / audience / capability the partner provides
- Lead with their value to the partnership, not ours

---

## Step 2D — Retainer Renewal (2 pages)

### Structure
1. What we delivered last cycle (numbers)
2. What changed in the market or their business
3. Recommended adjustment for next cycle (scope / pricing / KPIs)
4. One-line ask: "Confirm by [date] to lock in [start date]"

### Hard rules
- Always lead with delivered numbers, not effort
- Never propose a renewal at the same scope as the prior cycle without justification
- If results were poor, acknowledge it in §1 — don't bury it
- Pricing change >10% requires Sean + Antonio joint sign-off

---

## Voice Quick-Switch Reference

| If proposal type is | Then voice rules from |
|---|---|
| Upwork | `sw-content-voice-sean` |
| Direct client (post-discovery) | `sw-content-voice-antonio` |
| Partnership | Antonio's depth + Sean's directness — see joint examples in `brand/voice-and-tone.md` |
| Renewal | Whoever owns the account |

---

## Hard Rules (apply to ALL proposal types)

1. Always include at least one numbered case study
2. Never lead with "I'm excited", "I'd love to", or any version of those
3. Never use em dashes in formal proposals (commas, colons, or new sentences)
4. Never use "leverage", "synergy", "ensure", "utilise", "drive alignment", "streamline"
5. Always close with a clear next step — never "let me know if you have any questions"
6. Internal review before sending: paid media proposals → Antonio; outbound/sales-led → Sean
