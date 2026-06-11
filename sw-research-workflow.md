---
name: sw-research-workflow
description: >
  Standardised Streetwise research workflow combining Perplexity Spaces,
  Claude Projects, and NotebookLM into a single pipeline for ICP work,
  competitor analysis, content briefs, prospect research, and pre-pitch
  intel. Apply this skill any time research is needed before drafting,
  pitching, or strategic decision-making. Triggers on: research workflow,
  ICP research, competitor research, prospect research, market research,
  content brief, pre-pitch intel, deep research, perplexity, notebooklm,
  research pipeline, source finding, claude projects research, secondary
  research, primary research, due diligence, account research, before I
  pitch, before I write, research first.
---

# Streetwise Research Workflow
**Purpose:** A repeatable three-tool research pipeline that produces actionable intel without paying for enterprise research platforms. Used for every ICP, competitor, prospect, and content brief.

---

## The Three-Tool Stack

| Tool | Role | When to use |
|---|---|---|
| **Perplexity (with Spaces)** | Source-finding & research | First step on every project. Best for: live web data, citations, comparative analysis, "what do experts say about X". |
| **Claude Projects (claude.ai)** | Execution & creation from research outputs | Second step. Best for: turning research into briefs, proposals, content, frameworks. |
| **NotebookLM** | Deep analysis of a fixed document corpus | Optional third step. Best for: cross-document synthesis when you have a large set of files (transcripts, PDFs, articles) you want one cohesive view across. |

These are complementary, not competitive. Don't pick one — sequence them.

---

## Streetwise Perplexity Spaces (current set)

Maintain ~10 Spaces, each calibrated for a specific research lens. Naming convention: `[Topic] — [Lens]`.

| Space | Lens / Use case |
|---|---|
| Performance marketing intelligence | Channel benchmarks, platform updates, paid media trends |
| Industry-specific marketing | Vertical-specific tactics (DTC, SaaS, hospitality, events) |
| Competitor analysis | Pull positioning, pricing, content strategy of named competitors |
| Guide content research | Source material for paid ebooks, courses, lead magnets |
| ICP research | Buyer profiles, jobs-to-be-done, common objections per industry |
| Prospect intel | Pre-pitch research on specific companies before discovery calls |
| AI / automation industry | Updates on Claude, n8n, AI marketing tools, LLM market |
| Sales / outbound techniques | LinkedIn outbound, cold email, sales call frameworks |
| Tracking & attribution | GA4, GTM, server-side, attribution models |
| SEO + AI visibility | GEO (Generative Engine Optimization), AI Overviews, citation strategies |

Each Space accumulates context over time — the more you use it, the better its retrieval becomes for that lens.

---

## Standard Research Pipeline (4 Steps)

### Step 1 — Frame the question (Chat or Claude Project)
Before opening Perplexity:
1. Write down the actual decision the research will inform
2. Define what "good enough" looks like — what level of detail closes the question
3. Set a time box (1 hour for ICP research, 30 min for prospect intel, 4 hours for a content brief)

If the research scope is unclear, the rest of the pipeline drifts. Don't skip this.

### Step 2 — Source-finding (Perplexity Space)
1. Pick the right Space for the lens
2. Ask 3–5 targeted questions, not one broad one
3. Verify citations — Perplexity hallucinates fewer than ChatGPT but still hallucinates. Click sources for any claim that will end up in client-facing work
4. Save the response + source URLs to a research-doc

Output of Step 2: a research-doc with quotes, numbers, and citations — NOT a finished deliverable.

### Step 3 — Execution & creation (Claude Project)
1. Open the relevant Claude Project (with skills + CLAUDE.md context loaded)
2. Paste the research-doc as context
3. Ask Claude to synthesise into the actual deliverable (proposal, content brief, ICP doc, competitor matrix)
4. Use `sw-content-voice-antonio` or `sw-content-voice-sean` for voice-aligned output

Output of Step 3: a draft deliverable in Streetwise voice.

### Step 4 — Deep synthesis (NotebookLM, optional)
Use NotebookLM only when:
- You have 5+ source documents (transcripts, PDFs, articles)
- You need cross-document patterns (e.g., "what do all 12 of these sales call transcripts have in common?")
- The synthesis needs grounded citations from the corpus

Skip NotebookLM for ad-hoc research. It's overkill for single-question research.

---

## Use-Case Recipes

### ICP research (new vertical or new client type)
1. Perplexity Space: ICP research
2. Questions: "What does the buying committee look like for [vertical]? What are the top 3 objections? What does a typical sales cycle look like? Where do they get information?"
3. Claude Project: synthesise into a 1-page ICP doc with sections: Profile · JTBD · Objections · Channels · Triggers
4. Time box: 1 hour

### Pre-pitch prospect research (before a discovery call)
1. Perplexity Space: Prospect intel
2. Questions: "What does [company] do? Recent news? Who is the buyer (LinkedIn lookup)? Who funds them? What are their stated growth challenges?"
3. Claude Project (with `sw-content-voice-sean`): produce a 1-page call-prep doc
4. Time box: 30 min

### Content brief (for a paid product or guide)
1. Perplexity Space: Guide content research
2. Questions across 5–8 angles on the topic
3. NotebookLM: drop 5–10 long-form sources, ask for cross-doc patterns
4. Claude Project: synthesise into outline + key arguments + supporting data
5. Time box: 4 hours

### Competitor analysis (battlecard)
1. Perplexity Space: Competitor analysis
2. For each competitor: positioning, pricing, content angle, recent moves, weaknesses
3. Claude Project: produce a competitor matrix HTML via `sw-reporting-professional-html-document`
4. Time box: 2 hours per competitor

### Industry trend / channel benchmark research
1. Perplexity Space: Performance marketing intelligence
2. Questions: "What are current CPL benchmarks for [vertical] on [platform]? Recent platform changes? Successful creative formats?"
3. Claude Project: synthesise into a one-page benchmark doc to inform `sw-client-engine-config` updates
4. Time box: 1 hour

---

## Output Format Rules

Research outputs always follow this structure:

```markdown
# [Topic] Research — [YYYY-MM-DD]
**Research lens:** [which Perplexity Space]
**Time spent:** [N hours]
**Decision this informs:** [the decision from Step 1]

## Key findings
1. [Finding] — source: [URL]
2. [Finding] — source: [URL]

## Numbers / benchmarks
- [Metric]: [Value] — source: [URL]

## Quotes
> "[Quote]" — [Author, Source]

## Open questions
- [What we still don't know]

## Recommended action
[The 1-line decision based on the research]
```

Save to: `research/[topic]-[YYYY-MM-DD].md` in the relevant project folder.

---

## Hard Rules

1. Never use research outputs in client-facing work without verifying citations — Perplexity can hallucinate. Click every source URL before quoting.
2. Always frame the question first (Step 1) — research without a decision-target drifts into hours of unfocused reading.
3. Always time-box. 1 hour for ICP, 30 min for prospect intel, 4 hours for a content brief. Going over means scope creep.
4. Never paste raw Perplexity output into a deliverable. Always pass through Claude Project for voice and structure.
5. NotebookLM is for corpus analysis only — don't use it for single-question research.
6. Save all research outputs to a `research/` folder in the relevant project — they compound in value.
7. When a finding feels too good (e.g. "industry CPL is 80% lower than expected"), assume hallucination until verified by 2 independent sources.

---

## Integration with Other Skills

- `sw-content-voice-antonio` / `sw-content-voice-sean` — research synthesis happens in their voice
- `sw-reporting-professional-html-document` — competitor matrices, ICP docs, content briefs as HTML deliverables
- `sw-claude-surface-router` — research is Chat-heavy; only escalates to Cowork or Claude Code at execution
- `sw-client-onboarding` — pre-kickoff prospect/ICP research is part of every onboarding
- `sw-proposal-format-router` — pre-discovery proposals require ICP + prospect research first
- `sw-client-engine-config` — vertical benchmarks come from periodic industry-trend research
