---
name: sw-token-guard
description: >
  Actively prevents excessive token usage on every interaction. Use this skill
  at the START of every task before writing any code, loading any file, or
  generating any content. Triggers on: any request involving files, workflows,
  JSON, skills, CLAUDE.md edits, or multi-step tasks. Also triggers when the
  user asks to "build", "create", "update", "fix", "generate", or "refactor"
  anything. Also triggers on signs of inefficient prompting: large document
  processing, full document rewrites, long pasted blocks, repeated file
  references, multi-task prompts, repeated processing of the same file, or
  excessive output size. This is an always-on efficiency layer — when in doubt,
  apply it.
---

# Token Guard — Always-On Efficiency Layer

Apply this check **before every response**. Do not skip. Do not load files first.

This skill merges entry-gate logic with prompt-efficiency rewriting.
If the prompt is already efficient, proceed normally with no warning.
If inefficiencies are detected, emit the Usage Warning format in Step 5 before doing the task.

---

## Step 1 — Classify the Task

Ask internally: what category is this?

| Category | Examples | Action |
|---|---|---|
| **Trivial** | rename a file, fix a typo, update a table | Answer directly, no file loading |
| **Targeted edit** | change one node, update one section | Load ONLY that section |
| **Complex build** | new workflow, new module, full skill | Proceed to Step 2 |
| **Analysis** | review a file, summarize, explain | Load minimum needed |

If **Trivial** → skip all remaining steps and respond immediately.

---

## Step 2 — Input Minimization Check

Before loading any file, answer these:

1. **Do I need the full file or just one section?**
   - Full file needed → justify why, then load
   - Only one section → load only that section using targeted-edit-mode

2. **Is the input over 300 lines?**
   - Yes → apply document-compression-skill first
   - No → proceed

3. **Does this already exist somewhere in the project?**
   - Yes → reference it, don't regenerate it
   - No → proceed to build

4. **Am I about to load multiple skill files at once?**
   - Yes → load only the ONE most relevant skill
   - No → proceed

---

## Step 3 — Output Minimization Check

Before writing any response:

1. **Can this be done in a single pass?**
   - If yes → do it in one pass, no multi-step breakdowns unless asked
   - If no → tell the user why before proceeding

2. **Is the user asking for a full file rewrite when only a section changed?**
   - Yes → use targeted-edit-mode, return only the changed block
   - No → proceed

3. **Am I about to generate boilerplate the user didn't ask for?**
   - Yes → strip it, respond with only what was asked
   - No → proceed

---

## Step 4 — Inefficiency Risk Categories

Scan the user's prompt for these patterns. Any match triggers Step 5.

| Risk | Trigger |
|---|---|
| Large document processing | PDF analysis, long reports, full HTML generation, long code files, multiple uploaded documents |
| Full document rewriting | Request to rewrite or regenerate an entire doc when only a section actually needs editing |
| Long conversation context | Many revisions, large pasted blocks, repeated file references |
| Multi-task prompts | One request combines analysis + summarization + generation + formatting + rewriting |
| Repeated file processing | Same file already analyzed earlier in conversation |
| Large artifact editing loops | Repeated edits to the same HTML, report, code, or long doc |
| Excessive output size | Output would be very large; should be staged |

---

## Step 5 — Usage Warning Output (only if inefficiencies detected)

When any Step 4 risk fires, respond in this format BEFORE executing the task:

```
⚠️ Usage Warning: LOW / MEDIUM / HIGH

Reason
Briefly explain why the request may consume excessive usage.

Suggested Efficient Approach
Provide a staged plan to complete the task with less usage.

Optimized Prompt
Rewrite the user's request into a more efficient prompt they can copy.

Model Recommendation
Recommend a lighter model first when appropriate (Haiku for summarization or
outlining, Sonnet for final refinement, Gemini Flash for trivial tasks).
```

If the user confirms the optimized version, proceed. Otherwise execute as requested.

---

## Step 6 — Model Reminder (if relevant)

If the task is **Trivial or Targeted**:
> 💡 This task is a good candidate for Gemini 3 Flash — consider switching models to save quota.

If the task is **Complex Build**:
> 🔧 Complex task detected — Claude Sonnet or Claude Code recommended.

---

## Hard Rules (Never Break)

- Never load `CLAUDE.md` + a skill file + a workflow JSON in the same turn
- Never regenerate a complete file when only a section changed
- Never produce a multi-part response when one part would do
- Never load a skill file "just in case" — only load if directly needed
- If unsure what the user needs → ask ONE clarifying question before loading anything

---

## Integration with Existing Skills

This skill coordinates with:
- `sw-operations-targeted-edit-mode` → delegate all partial edits here
- `document-compression-skill` → delegate all large file handling here

Token Guard is the **entry gate** AND the **prompt-efficiency rewriter**.
The targeted-edit and compression skills are the **execution layer**.

---

## Priority

Always prioritize:
- smaller context
- smaller prompts
- staged execution
- targeted edits instead of full rewrites
- summaries instead of repeated file analysis

The goal is to reduce usage while keeping quality high.
