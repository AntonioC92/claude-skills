---
name: professional-html-document
description: >
  Generates professionally designed, single-file HTML documents in a refined navy, gold and beige visual style.
  Use this skill whenever the user asks to create any structured professional document as an HTML file, including:
  action plans, operational plans, project plans, proposals, engagement proposals, client reports, status reports,
  responsibility splits, weekly task breakdowns, onboarding documents, or any formal documented deliverable.
  Triggers on phrases like "create an action plan", "build a proposal", "write a report", "project plan HTML",
  "client document", "operational plan", "responsibility split", "engagement document", "status report",
  "documented plan", or "make a professional HTML document". Always use this skill when the output should be
  a polished, styled HTML file suitable for sharing with a client or colleague. The skill produces a single
  self-contained HTML file ready to push to GitHub Pages or share directly.
---

# Professional HTML Document Skill

Produces a single-file, self-contained HTML document for professional use. Covers action plans, proposals,
reports, and any structured client-facing deliverable. The design system uses Playfair Display headings,
DM Sans body text, and DM Mono for labels. Colours are fully customisable via CSS variables but default
to the navy, gold and beige palette established in the Chef's Office project.

## Document types this skill covers

| Type | Use when |
|---|---|
| Action Plan | Week-by-week task breakdown with owners, timelines, deliverables |
| Proposal | Scope of work, phases, commercial model, next steps |
| Engagement Report | Performance summary, findings, recommendations |
| Onboarding Document | Responsibilities, tools, processes for a new engagement |
| Status Report | Progress against plan, blockers, upcoming tasks |
| Responsibility Split | Two-column ownership breakdown between collaborators |

---

## Step 1: Identify document type and gather inputs

Ask the user (or extract from context) before generating:

| Input | Description |
|---|---|
| Document type | Action plan / proposal / report / other |
| Project / client name | Used in header and footer |
| Owners or parties | Names of people, teams, or companies involved |
| Owner roles | Short label per person (e.g. "Strategy & Creative") |
| Brand colours | Override defaults if client has a specific palette |
| Time period | e.g. 2-week plan, 90-day roadmap, Q1 report |
| Section content | Goals, tasks, phases, findings — whatever fits the type |
| Confidentiality level | Shown in header/footer |

If the user provides a brief, contract, or existing document, extract all inputs from it directly.

---

## Step 2: Design system defaults

```css
:root {
  --primary:    #1F2A44;   /* deep navy */
  --accent:     #C7A86D;   /* muted gold */
  --neutral:    #E6D9C3;   /* warm beige */
  --charcoal:   #333333;
  --muted:      #6b7280;
  --white:      #ffffff;
  --bg:         #f0ece4;

  /* Two-party colour coding — adjust per project */
  --owner1-bg:   #daeaf6;
  --owner1-text: #1F4E79;
  --owner2-bg:   #fdf0d5;
  --owner2-text: #7B4F00;
  --both-bg:     #e8f5e9;
  --both-text:   #2E7D32;
  --danger:      #8b3333;
}
```

**Fonts — always load from Google Fonts:**
```html
<link href="https://fonts.googleapis.com/css2?family=Playfair+Display:wght@400;700;900&family=DM+Sans:wght@300;400;500;600&family=DM+Mono:wght@400;500&display=swap" rel="stylesheet">
```

---

## Step 3: Document structure

All sections are optional except Header and Footer. Pick what fits the document type.

```
1. Header              — client/project name, document title, meta tags
2. Context section     — mission, background, or executive summary
3. Responsibility split — two-column ownership table (action plans, proposals)
4. Phase or week blocks — structured task tables (action plans, onboarding)
5. Scope of work        — phase descriptions with deliverables (proposals)
6. Commercial model     — pricing or engagement structure (proposals)
7. Findings / results   — summary of performance or audit (reports)
8. Deliverables table   — full output list with owner and deadline
9. Reference section    — brand guide, technical specs, or guidelines
10. Hold / out of scope — items explicitly not covered in this phase
11. Footer              — client, document type, phase, confidential label
```

---

## Step 4: Component patterns

### Section header
```html
<div class="section-header">
  <span class="section-num">01</span>
  <h2 class="section-title">Section Title</h2>
  <div class="section-rule"></div>
</div>
```

### Owner pills
```html
<span class="owner-pill owner-1">Name</span>
<span class="owner-pill owner-2">Name</span>
<span class="owner-pill owner-both">Both</span>
```

### Task table row (action plans)
```html
<tr>
  <td class="td-day">1-2</td>
  <td class="td-owner"><span class="owner-pill owner-1">Name</span></td>
  <td>Task description</td>
  <td class="td-notes">Short directive note</td>
</tr>
```

### Phase block (proposals)
```html
<div class="phase-block">
  <div class="phase-header">
    <span class="phase-title">Phase 1: Setup</span>
    <span class="phase-tag">Weeks 1-3</span>
  </div>
  <ul class="phase-list">
    <li>Deliverable one</li>
    <li>Deliverable two</li>
  </ul>
</div>
```

### Info card
```html
<div class="card">
  <p><strong>Key point:</strong> supporting detail here.</p>
</div>
```

### Hold / out of scope box
```html
<div class="hold-box">
  <div class="hold-label">Out of scope / on hold</div>
  <div class="hold-title">These items are not covered in this document</div>
  <ul><li>Item</li></ul>
</div>
```

---

## Step 5: Copy rules

Apply to all text content without exception:

- No em dashes. Use a colon, comma, or full stop instead.
- No "it's X, not Y" contrast framing. State what something is.
- No filler words: "genuinely", "essentially", "seamlessly", "straightforward".
- No informal words: "scrambles", "guru", "nailing it", "kills it".
- Notes and descriptions: short, directive, specific.
- Card text: plain declarative sentences. One idea per sentence.
- Headings: sentence case, no trailing punctuation.

---

## Step 6: Output

- Single `.html` file, fully self-contained
- All styles in a `<style>` block in `<head>`
- Google Fonts via `<link>` tag (the only external dependency)
- File named: `[client]-[document-type].html` e.g. `antigravity-proposal.html`
- Output to `/mnt/user-data/outputs/`
- If a GitHub token and repo are in session context, push automatically after creating

---

## Step 7: Base template

See `assets/base-template.html` for the full working HTML with all CSS.
Copy it as a starting point and replace content. Do not rewrite CSS from scratch.
Adapt the section structure to match the document type being produced.
