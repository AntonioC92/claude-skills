---
name: sw-claude-surface-router
description: >
  Routes every work request to the right Claude surface — Chat, Cowork, or
  Claude Code — based on task type. Apply this skill before starting any
  non-trivial Streetwise task to avoid burning Cowork quota at 2–3× the rate
  unnecessarily, or stalling on Chat when Claude Code would have finished
  the work autonomously. Triggers on: Claude surface, which surface, Cowork
  vs Chat, Claude Code vs Cowork, where should I do this, surface routing,
  use Cowork or, do this in Chat or, switch to Claude Code, model routing,
  tool routing, quota management, save quota, burn rate, conserve quota,
  too much usage. Pair with `sw-token-guard` for in-task efficiency.
---

# Streetwise Claude Surface Router
**Why this exists:** Cowork sessions burn quota at 2–3× the rate of Chat. Picking the wrong surface for a task wastes capacity. This skill routes work to the right surface before the task starts.

---

## The Three Surfaces

| Surface | Best for | Burn rate vs Chat |
|---|---|---|
| **Chat (claude.ai)** | Q&A, research, strategy, planning, draft writing | 1× |
| **Cowork (desktop app)** | Executing deliverables from prepared files; Slack/CRM/Drive automation; multi-MCP workflows | 2–3× |
| **Claude Code (CLI / IDE)** | Autonomous coding, terminal work, git operations, repo edits, n8n workflow JSON, skill-file maintenance | 1× (similar to Chat) |

---

## Decision Tree

Run this top-down. Stop at the first match.

```
Is the work primarily code, repo files, terminal commands, or git ops?
  → Claude Code

Does the task require live data from MCP integrations (Slack, Gmail, Drive,
HubSpot, ad platforms) AND produce a file deliverable for a client?
  → Cowork

Is the task answering a question, researching, planning, drafting, or
producing content that will be pasted into another tool?
  → Chat

Does the task involve editing existing skill files, CLAUDE.md, or n8n
workflow JSON in a versioned folder?
  → Claude Code

Is the user asking for something that genuinely needs MCP integrations
that only run in Cowork (e.g., reading Slack channels, sending emails)?
  → Cowork

Default → Chat
```

---

## Concrete Routing by Task Type

| Task | Right surface | Why |
|---|---|---|
| "What's our positioning for SaaS clients?" | Chat | Pure thinking, no integrations needed |
| "Read these 5 lead Slack channels and analyse Sean's voice" | Cowork | Needs Slack MCP, produces file output |
| "Update the data-normalization-schema skill to add LinkedIn fields" | Claude Code | Repo file edit, git involved |
| "Draft a proposal for Acme Corp based on the discovery call" | Chat | Drafting work; paste into Google Doc after |
| "Build the n8n Multi-Client Router workflow JSON" | Claude Code | Code/JSON, version-controlled |
| "Pull last 7 days of Meta Ads data and generate a report HTML" | Cowork | Needs Meta MCP + file output |
| "Analyse this CSV of campaign performance and find anomalies" | Chat (with file upload) | Pure analysis, no live data |
| "Create the campaign brief from this discovery call transcript" | Chat | Drafting work |
| "Set up the new client folder and generate a brief.md template" | Claude Code | File creation in repo |
| "Send a follow-up email to the prospect we just spoke to" | Cowork | Needs Gmail MCP |
| "Edit and push a skill file change to GitHub" | Claude Code | File + git |
| "Fill in this PDF form with client details" | Cowork | PDF MCP integration |

---

## When NOT to Use Cowork

Cowork is the most expensive surface. Avoid it when:
- The task is pure Q&A or research → Chat
- The task is purely code/repo work → Claude Code
- The task is drafting that the user will paste into another tool manually
- The task is iterative refinement of a single document (use Chat with the doc pasted in)

The exception: if the task requires multiple MCP tool calls (Slack + Gmail + Drive in one flow), Cowork is worth the burn because the alternative is multiple tool-by-tool runs in Chat.

---

## When Claude Code Beats Cowork

Claude Code is preferred over Cowork when:
- Editing files in a versioned repo (skills, CLAUDE.md, workflow JSON)
- Running terminal commands (git, npm, n8n CLI)
- Iterating on code with autonomous file edits
- Working in a long session where context accumulates around the codebase

Reference: SWE-bench scores Claude Code at 72.7% vs ChatGPT + Codex at 69.1%. Use Claude Code for autonomous coding work.

---

## Mid-Task Switching

Sometimes a task starts in one surface and should move to another. Switch when:

- **Started in Chat, hit a wall on file output** → switch to Cowork
- **Started in Cowork, just answering follow-up questions** → switch to Chat
- **Started in Cowork, need to push to GitHub** → switch to Claude Code
- **Started in Claude Code, need to read Slack** → switch to Cowork
- **Hitting usage limits in current surface** → switch to a fresh chat in another surface, pass a summary as context

---

## Hard Rules

1. Never start a Cowork session for pure Q&A — always Chat first, escalate to Cowork only if integrations are needed
2. Never start a Chat session for autonomous repo work — Claude Code finishes it faster and cheaper
3. Always pass a summary when switching surfaces mid-task — context doesn't carry between Chat and Cowork
4. Never run two surfaces in parallel for the same task — pick one and finish it there
5. When usage is tight, default to Chat or Claude Code over Cowork
6. Cowork's value is the integrations. If the task doesn't use them, Cowork is wasted.

---

## Integration with Other Skills

- `sw-token-guard` — runs INSIDE the chosen surface to minimise burn within the session
- `sw-operations-targeted-edit-mode` — Claude Code mostly; partial edits across all surfaces
- `sw-research-workflow` — Chat-heavy (Perplexity → Claude Projects pattern)
- `sw-client-onboarding` — multi-step task that often spans surfaces (Chat for brief + Cowork for access setup + Claude Code for folder scaffolding)
