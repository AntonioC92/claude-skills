---
name: sw-approval-gate-pattern
description: >
  Defines the Streetwise pattern for inserting a human approval step before
  any AI-driven write operation goes live. Use this skill any time an n8n
  workflow, automation, or Claude-driven process is about to push changes to
  a live system — Meta Ads API, Google Ads API, LinkedIn Ads API, CRM
  contact records, scheduled email sends, content publishing, automated
  campaign edits, or any irreversible action. Triggers on: approval gate,
  approve reject, human review, write operations, campaign optimization,
  slack approval, before going live, before publishing, human-in-the-loop,
  approval workflow, review and approve, wait node, webhook callback,
  optimization engine, automated changes. Prevents AI from acting on a
  client's account without explicit human confirmation.
---

# Streetwise Approval Gate Pattern
**Risk level if missing:** HIGH — uncontrolled writes to client systems can cause budget overruns, brand-safety incidents, or account suspensions.
**Apply to:** every workflow that writes to a live external system (ad platforms, CRMs, send platforms, websites).

---

## When This Pattern is Required

Required for every workflow node that:
- Creates, edits, pauses, or deletes ad campaigns / ad sets / ads
- Adjusts campaign budgets
- Publishes social posts, blog posts, or website changes
- Sends email broadcasts or sequence triggers
- Modifies CRM lifecycle stages or list memberships
- Creates calendar events, sends meeting invites, or books resources
- Anything labeled "irreversible" or "live"

NOT required for:
- Pure read operations (analytics pulls, reporting)
- Internal workflow logic (transforms, normalisations)
- Drafts saved to internal storage (Drive, Notion, GitHub) for human review later

---

## The Pattern (n8n Implementation)

```
[Trigger / AI Generation]
        ↓
[Build Change Summary]              ← human-readable description of the change
        ↓
[Slack Send Approval Message]       ← message with approve/reject buttons + summary
        ↓
[Wait Node]                         ← workflow pauses, waits for webhook callback
        ↓
[Webhook Callback]                  ← Slack button click hits this URL
        ↓
[IF: approved?]
   Yes → [Execute Write API]       ← only now does the write happen
   No  → [Log Rejection + Stop]    ← no write, log reason for retraining
        ↓
[Log Outcome to Memory Sheet]
```

---

## Step 1 — Build Change Summary

Before sending to Slack, generate a clear summary the human can read and decide on. Format:

```
🚦 APPROVAL REQUIRED — [Workflow Name]

Client: [Client Name]
Platform: [Meta / Google / LinkedIn / Other]
Action: [CREATE_CAMPAIGN / PAUSE_AD / EDIT_BUDGET / ...]

What will change:
- [Specific change in plain English]

Reasoning:
- [Why the AI recommends this — pull from analysis output]

Estimated impact:
- [Spend / reach / risk note]

Reject reason (if any): [empty — to be filled if rejected]
```

Never send a Slack approval that says "Optimize campaigns?" — every approval must name the EXACT change being made.

---

## Step 2 — Slack Approval Message

Use Slack's interactive Block Kit format with two buttons.

```javascript
{
  text: "Approval required for [Client] – [Action]",
  blocks: [
    {
      type: "section",
      text: { type: "mrkdwn", text: changeSummary }  // from Step 1
    },
    {
      type: "actions",
      elements: [
        {
          type: "button",
          text: { type: "plain_text", text: "✅ Approve" },
          style: "primary",
          value: "approve",
          url: `${webhookUrl}?decision=approve&run_id=${runId}`
        },
        {
          type: "button",
          text: { type: "plain_text", text: "❌ Reject" },
          style: "danger",
          value: "reject",
          url: `${webhookUrl}?decision=reject&run_id=${runId}`
        }
      ]
    }
  ]
}
```

Send to:
- Client-specific Slack channel for client-facing changes (so the client sees what's being decided)
- Internal Streetwise channel for back-end automation (not client visible)

---

## Step 3 — Wait Node

n8n native Wait node configuration:
- Resume: On Webhook Call
- HTTP Method: GET
- Response Mode: Last Node
- Response Code: 200
- Timeout: 24 hours (configurable per workflow)

If timeout fires → workflow ends without writing. Log as TIMEOUT.

---

## Step 4 — Webhook Callback Handler

The Slack button URL hits the n8n webhook. Parse the query params:
- `decision`: approve | reject
- `run_id`: links back to the original workflow run

Set the value into workflow context for the IF node downstream.

---

## Step 5 — Execute Write (only if approved)

Only the approved branch contains the actual API write. Pair with platform-specific protocols:
- Meta writes → apply `sw-meta-business-manager-protocol` (PAUSED-first, etc.)
- Google Ads writes → similar discipline
- Email sends → confirm send list size and segments before fire

---

## Step 6 — Log Every Decision

Whether approved or rejected, log to the memory sheet:

| timestamp | client | workflow | action | summary | decision | decided_by | result | reject_reason |
|---|---|---|---|---|---|---|---|---|

This becomes:
1. An audit trail for the client (proof of due diligence)
2. Training data to improve future AI recommendations (if AI keeps recommending changes that get rejected, the prompt or thresholds need tuning)
3. A compliance artefact

---

## Variants

### Variant A — Per-change approval (default)
One Slack approval per individual change. Best for low-volume, high-stakes ops.

### Variant B — Batch approval
One Slack approval per batch of changes, with a linked Google Sheet listing all changes. Used when 10+ changes are recommended at once. Approver checks the sheet, then approves the whole batch.

### Variant C — Auto-approve below threshold
Any change with estimated impact below a defined threshold (e.g. <$50 spend reallocation, no audience change) auto-approves with a Slack notification only (no buttons). Anything above threshold uses Variant A.

Pick the variant in the workflow's `brief.md`. Default to Variant A unless explicitly chosen.

---

## Hard Rules

1. Never deploy a write workflow without an approval gate in production
2. Never auto-approve writes to ad accounts or send platforms (read-only ops are fine)
3. Always include a plain-English summary — never raw JSON in the Slack message
4. Always log decisions to the memory sheet
5. Always notify the client channel for client-facing changes (transparency builds trust)
6. Always set a timeout on Wait nodes — never indefinite waits
7. If the same change keeps getting rejected, the AI prompt or threshold logic needs updating — don't keep firing the same rejected proposal weekly

---

## Integration with Other Skills

- `sw-workflows-architecture-build-protocol` — overall n8n build rules
- `sw-meta-business-manager-protocol` — required for Meta API writes downstream
- `sw-paid-campaigns-data-normalization-schema` — the data structure feeding the change recommendations
- `sw-workflows-optimisation-analysis-prompt-template` — the AI analysis upstream of any approval gate
