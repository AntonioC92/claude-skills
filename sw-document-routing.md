---
name: sw-document-routing
description: >
  Routes every document creation request to the correct format based on document type.
  ALWAYS consult this skill before building any document, proposal, report, plan, or deliverable.
  Triggers on: proposal, pitch, deck, offer, engagement letter, scope of work, audit, campaign plan,
  action plan, strategy doc, report, brief, onboarding doc, project plan, or any request to
  "create a document", "write a proposal", "build a plan", or "prepare something for a client".
  This skill overrides any default format preference — the routing rules here are final.
---

# Document Routing Skill

Before creating any document, consult this routing table to determine the correct output format.
Never default to HTML, Markdown, or any other format without checking here first.

---

## Routing Table

| Document Type | Format | When |
|---|---|---|
| Pitch | Google Doc | Any pitch to a prospect or lead |
| Commercial Proposal | Google Doc | Pre-signature offers, pricing, packages |
| Engagement Proposal | Google Doc | Scope + terms sent before a client signs |
| Partnership Proposal | Google Doc | Any proposal to a potential partner pre-agreement |
| Offer Letter / Quotation | Google Doc | Formal pricing documents sent to prospects |
| Project Plan / Operational Plan | HTML | Internal or client-facing execution plans |
| Campaign Plan | HTML | Meta, Google, LinkedIn campaign strategy docs |
| Audit Report | HTML | SEO, ads, analytics, or technical audits |
| Action Plan | HTML | Step-by-step executional breakdowns |
| Strategy Document | HTML | Channel strategy, funnel strategy, growth plans |
| Onboarding Document | HTML | Client onboarding, team onboarding, process docs |
| Status Report | HTML | Progress updates, performance reports |
| Technical Brief | HTML | Specs, requirements, integration docs |

---

## Rule Summary

**Google Doc** → anything that goes to a prospect *before* they sign. The goal is to close.

**HTML** → anything executional, operational, or project-based. The goal is to deliver or document.

---

## How to Build Each Format

### Google Doc (pitch / pre-sale proposals)

Use the **Google Drive MCP** to create a Google Doc directly in the user's Drive.

Steps:
1. Call `tool_search` with query `"google drive create file"` to load the Google Drive tool.
2. Use `Google Drive:create_file` to create a new Google Doc.
3. Populate with the appropriate proposal content.
4. Return the Google Doc link to the user.

Do NOT create an HTML file and call it a "proposal" if the document is pre-signing.
Do NOT create a local `.docx` unless the user explicitly asks for Word format.

### HTML Document (executional / project documents)

Use the **`sw-reporting-professional-html-document`** skill to build a polished, single-file HTML output.

Steps:
1. Read `sw-reporting-professional-html-document.md` (in this same `claude-skills/` folder) for full build instructions.
2. Follow that skill's instructions to produce the HTML file.
3. Present the file to the user via `present_files`.

Do NOT create a Google Doc for audits, plans, or campaign strategies.

---

## Edge Cases

- **"Proposal" is ambiguous** → Ask: "Is this going to a prospect before signing, or is it an internal/executional project plan?" Then route accordingly.
- **User asks for Word / PDF** → Override this routing and follow the user's explicit format request.
- **User asks for slides** → This skill does not cover presentations. Use Canva, Gamma, or pptx skill as appropriate.
- **Proposal + execution combined in one request** → Split into two documents: Google Doc for the commercial part, HTML for the plan.
