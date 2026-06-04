---
name: sw-session-checkpoint
description: >
  Monitors session heaviness and auto-saves a structured context summary to the
  correct client's 05-Context MD folder so Antonio can close the chat and resume
  in a new session without losing progress.
  ALWAYS trigger after any major task completion, when a large file is read,
  when multiple skills are loaded, after a long debugging loop, or when the user
  says "save context", "checkpoint", "session summary", "start fresh",
  "continue in new chat", "too heavy", "export this session", "is it safe to continue",
  "summarise what we did", or "save progress".
  Also triggers PROACTIVELY — Claude should run a lightweight session weight check
  after every completed task block and surface a ⚑ badge if risk is MEDIUM or HIGH.
---

# sw-session-checkpoint — Streetwise Session Manager

Prevents context ceiling crashes and gives Antonio a clean way to close any chat
and resume exactly where he left off, per client.

---

## 1 — Proactive Monitoring (always-on)

After every completed task block, Claude silently estimates session weight using
these heuristics and shows a status badge in the response:

| Signal | Weight |
|---|---|
| Each HTML/JSON file fully read (>200 lines) | +2 |
| Each skill file loaded | +1 |
| Each file written or edited | +1 |
| Long debug loop (5+ turns on same issue) | +3 |
| Large paste in conversation (>100 lines) | +2 |
| PDF or multi-page doc read | +2 |
| Session turns beyond 20 | +1 per 5 turns |

**Thresholds:**
- **0–4** → 🟢 Light — continue freely
- **5–8** → 🟡 Medium — finish current task then checkpoint
- **9+**  → 🔴 Heavy — checkpoint before next task

Show this badge at the end of each response when weight ≥ 5:

```
🟡 Session weight: MEDIUM — checkpoint recommended after this task
```
or
```
🔴 Session weight: HEAVY — save context before continuing
```

When weight is LOW, no badge needed — stay invisible and don't interrupt flow.

---

## 2 — Checkpoint Routine

When triggered (proactively or by the user), run this sequence:

### Step 1 — Identify active client
Look at the current conversation context for these cues:
- File paths mentioned (e.g. `clients/Kubiieo/...` → client is **Kubiieo**)
- Client name mentioned explicitly
- Task subject matter (Google Ads for Kubiieo, email sequence for CareerCoin, etc.)

If unclear, ask once: *"Which client is this session for, so I save the context to the right folder?"*

Save destination:
```
clients/[Client Name]/05-Context MD/[YYYY-MM-DD]-[topic]-context.md
```
For internal/Streetwise work (no client):
```
business-development/05-Context MD/[YYYY-MM-DD]-[topic]-context.md
```

### Step 2 — Generate the session summary (see format in §3)

### Step 3 — Write the file
Write the summary to the correct path using the Write tool.
Confirm with: *"Context saved to `clients/[Client]/05-Context MD/[filename]`"*

### Step 4 — Show the resume command
Print this block so Antonio can copy-paste it at the top of the next session:

---
**▶ To resume this session, start a new chat and paste:**

> I'm continuing work on [brief topic]. Load context from:
> `~/Documents/streetwise-consultancy/clients/[Client]/05-Context MD/[filename]`
> Read that file first, then pick up from: [one-line next action]

---

---

## 3 — Session Summary Format

```markdown
# Session Context — [YYYY-MM-DD] — [Topic]
**Client:** [Client Name or "Internal Streetwise"]
**Session focus:** [one sentence — what this session was about]

---

## What was done
- [action 1 — specific, with file name if applicable]
- [action 2]
- [action 3]

## Key decisions made
- [decision 1 — include reasoning if non-obvious]
- [decision 2]

## Files created or modified
| File | Location | Status |
|---|---|---|
| [filename] | [path] | WIP / Final |

## What's still open
- [unfinished item 1]
- [unfinished item 2]

## How to resume
Load this file at the start of the next session, then:
1. [first action to take]
2. [second action if needed]

## Context the next session needs
- [key fact or constraint that isn't obvious from the files]
- [any pending decision or approval needed]
```

---

## 4 — Output Routing (extends sw-wip-output-router)

This skill adds one new valid destination type to the routing rules:

| File type | Destination |
|---|---|
| Session context summaries | `clients/[Client]/05-Context MD/` |
| Session context summaries (internal) | `business-development/05-Context MD/` |
| All other WIP | `clients/[Client]/04-working/` |
| AI/automation configs, workflow JSON, MCP packs | `clients/[Client]/ai-automations/` |
| Internal Streetwise WIP | `business-development/04-working/` |

**Standard client folder structure (all 6 folders):**
```
clients/[Client Name]/
├── 01-received/
├── 02-Strategy/
├── 03-Deliverables/
├── 04-working/
├── 05-Context MD/
└── ai-automations/
```

**Never** save session context to `04-working/` — that folder is for deliverables in progress.
**Never** save it to the session sandbox (`/outputs/`).
The `05-Context MD/` folder is the only valid home for session summaries.
The `ai-automations/` folder holds built automation assets (n8n JSON, MCP packs, meta-automation configs) — it is NOT a WIP folder.

---

## 5 — Starting a New Session (how to use a saved context)

When a session starts with a reference to a `05-Context MD` file:
1. Read the file immediately (no need to ask)
2. Confirm: *"Context loaded from [filename]. Picking up from: [next action]."*
3. Do not re-explain what was done — just pick up the next action
4. Apply `sw-wip-output-router` for all files in the new session as normal

---

## 6 — Hard Rules

1. **Always save the context file BEFORE closing** — never just display it in chat
2. **Always use the Write tool** to create the MD in the correct `05-Context MD/` folder
3. **Always include the resume command block** at the end of a checkpoint
4. **Detect the client from context** — never save to the wrong client's folder
5. **Run proactively** — don't wait for the user to ask; show the weight badge when MEDIUM/HIGH
6. **This skill runs after sw-token-guard and sw-wip-output-router** in the skill stack
