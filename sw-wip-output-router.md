---
name: sw-wip-output-router
description: >
  Routes every work-in-progress file output to the correct `04-working/`
  subfolder before any Write or Edit operation. Apply this skill ANY time
  Claude is about to save, write, or create a file inside the
  streetwise-consultancy folder. Triggers on: save to file, create a file,
  write the doc, output to, deliver the, generate a, build a doc, build a
  report, build a deck, draft the, produce a, export to, save the output,
  put it in, where should I save, output folder, working folder, 04-working,
  client deliverable, internal doc, write this to. Hard rule: never default
  to `outputs/` or to the project root — always pick the right
  `04-working/` location. This skill OVERRIDES any other default file path.
---

# Streetwise WIP Output Router
**Risk if violated:** Files end up in wrong folders, breaking the 04-working / 03-Deliverables separation. Drafts get mistaken for finals; finals get diluted with WIP. Both founders lose trust in the folder structure.

**Apply BEFORE every Write or Edit call** that creates a file inside `~/Documents/streetwise-consultancy/`. Not after. Not "let me check then save" — decide path FIRST, then Write.

---

## The Only Rule

```
Is this work-in-progress (anything not yet client-approved / sign-off-ready)?
  YES → Save to one of:
    • clients/[Client Name]/04-working/       (if client-specific)
    • business-development/04-working/        (if internal Streetwise)

  NO  → Only then save to:
    • clients/[Client Name]/03-Deliverables/  (finalised, signed-off client output)
    • business-development/[subfolder]/       (finalised internal asset)
```

**Default assumption:** every file Claude creates is WIP. Promote to "finalised" only when the user explicitly signs off.

---

## Decision Tree (run silently before every file save)

### Step 1 — Is this inside the streetwise-consultancy folder?
- **No** (session outputs folder, /tmp, scratch) → skill doesn't apply, save where it makes sense
- **Yes** → go to Step 2

### Step 2 — Is this for a specific client or internal Streetwise?
Look at the task for these cues:

**Client cues (route to `clients/[Name]/04-working/`):**
- A client name is mentioned (Chef Academy, DP Gates, Kubiieo, Nineteenth Golf, Orbitone, CareerCoin, or any future client)
- The task is about a campaign, ad, report, audit, strategy doc for that client
- The user said "for [client]"
- The conversation is about client deliverables, client comms, client kickoff

**Internal cues (route to `business-development/04-working/`):**
- The task is about Streetwise's own brand, website, sales pipeline, case studies
- The output is for Sean & Antonio internally (not a paying client)
- Mentions: outreach systems, internal reports, brand assets, sales decks, website copy, pricing docs, founder content, biz dev
- Mentions: Pillar 1 (brand), Pillar 3 (financial growth), Pillar 4 (new revenue)

**Ambiguous** → ask once: "Is this for a specific client, or internal Streetwise?"

### Step 3 — Confirm the exact target path
Before calling Write, the path MUST match one of:

```
/Users/Antonio/Documents/streetwise-consultancy/clients/[Client Name]/04-working/[filename]
/Users/Antonio/Documents/streetwise-consultancy/business-development/04-working/[filename]
```

If your draft path doesn't match → STOP and reroute.

---

## What NOT to Do (banned patterns)

1. **NEVER save WIP to `clients/[Name]/outputs/`.** That folder doesn't exist in the new convention. Old clients (Orbitone, CareerCoin) may still have it — leave alone, but for new work always use `04-working/`.

2. **NEVER save WIP to `clients/[Name]/03-Deliverables/`.** That folder is RESERVED for client-approved finals. Putting drafts there pollutes the signal.

3. **NEVER save WIP to `clients/[Name]/01-received/` or `02-Strategy/`.** `01-received/` is read-only client assets. `02-Strategy/` is the agreed plan, frozen once approved.

4. **NEVER save internal Streetwise WIP to the root of `business-development/` or to its subfolders** (`assets/`, `outreach/`, `paid marketing/`, `reporting/`, `seo/`, `website/`). Those are for finalised, published assets. Always use `business-development/04-working/`.

5. **NEVER save anywhere inside the project that's not `04-working/` UNLESS the user has explicitly approved the output as final.** Examples of "approved as final": "this is signed off, move it to 03-Deliverables", "publish this to the website folder", "this is the final version."

6. **NEVER auto-create new folders like `outputs/`, `drafts/`, `work/`, `tmp/`, `wip/`, `temp/`** inside the streetwise-consultancy structure. The folder convention is fixed: `01-received` / `02-Strategy` / `03-Deliverables` / `04-working` for clients; `business-development/04-working` for internal.

---

## When to Move a File Out of `04-working/`

The user must explicitly approve a move with phrases like:
- "this is final, move it to deliverables"
- "approve and ship"
- "promote to 03-Deliverables"
- "publish this"
- "send to the client" + clear sign-off context

When that happens:
- Client work → move from `clients/[Name]/04-working/` to `clients/[Name]/03-Deliverables/`
- Internal work → move from `business-development/04-working/` to the appropriate `business-development/[subfolder]/` (e.g. `assets/case-studies/`, `website/`, etc.)

Use `mv` not `cp` — don't leave a duplicate behind. The folder convention is one source of truth.

---

## Naming Convention Inside `04-working/`

Optional but recommended for clarity:
- `[topic]-[YYYY-MM-DD].ext` — for dated artefacts (reports, audits, plans)
- `[topic]-v[N].ext` — for iterating drafts where versions matter
- `[topic].ext` — for a single working draft that will be moved when finalised

Example:
- `dublin-beer-festival-case-study.html`
- `kubiieo-week-plan-2026-05-12.html`
- `chef-academy-onboarding-v2.md`

---

## How This Skill Should Show Up in Claude's Responses

When saving a file, Claude should briefly state the routing decision:

> Saving to `clients/Kubiieo/04-working/google-ads-brief.html` per sw-wip-output-router (client WIP).

OR

> Saving to `business-development/04-working/sales-pipeline-draft.md` per sw-wip-output-router (internal WIP).

This makes the routing audit-able and reminds the user the file isn't promoted yet.

---

## Hard Rules

1. ALWAYS pick the path BEFORE calling Write, not after
2. ALWAYS default to `04-working/` for any new file unless explicitly approved as final
3. ALWAYS state the routing decision in the response so the user can catch errors
4. NEVER create new folders outside the `01-received` / `02-Strategy` / `03-Deliverables` / `04-working` convention (clients) or `business-development/04-working/` (internal)
5. NEVER move a file out of `04-working/` without explicit user approval
6. If unclear whether client vs internal, ASK once — don't guess
7. This skill takes precedence over generic file-saving instincts

---

## Valid Folder Destinations (full map)

| File type | Destination |
|---|---|
| Client WIP (drafts, plans, reports in progress) | `clients/[Client]/04-working/` |
| Client finals (approved, signed-off) | `clients/[Client]/03-Deliverables/` |
| Session context summaries (client) | `clients/[Client]/05-Context MD/` |
| Session context summaries (internal) | `business-development/05-Context MD/` |
| Internal Streetwise WIP | `business-development/04-working/` |
| Internal Streetwise finals | `business-development/[subfolder]/` |

`05-Context MD/` is managed exclusively by `sw-session-checkpoint`. Do not save any other file type there.

---

## Integration with Other Skills

- `sw-token-guard` runs FIRST (entry gate, decides task type)
- `sw-wip-output-router` runs SECOND, right before any Write/Edit
- `sw-session-checkpoint` handles `05-Context MD/` routing — defer to it for all session summary files
- `sw-document-routing` decides file FORMAT (Google Doc vs HTML vs PDF) — independent of WHERE the file goes
- `sw-client-onboarding` creates the `04-working/` folder for new clients at kickoff
- `sw-reporting-professional-html-document` produces HTML deliverables, which still land in the right `04-working/` per this skill
