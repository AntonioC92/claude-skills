# Verification Protocol — Post-Save

> The exact sequence to run after saving an RSA (or any campaign element). Do not skip steps. The "I'm pretty sure it saved" failure mode is responsible for the highest-cost defect class in this skill's history — wrong headlines living in a live ad for sessions.

## Why this exists

Two failure modes drove this protocol into existence:

1. **Unsaved work lost between sessions.** Session 1 makes changes, achieves "Good" ad strength, doesn't click Save (because "I'll save once everything's perfect"). Session ends, browser closes, work is gone.
2. **Saved value differs from typed value.** Angular silent change-detection failure — see `angular-interaction-patterns.md`. Field shows new text, model holds old text, Save writes old text. The defect is invisible until you read it back from the list view or from a fresh editor session.

The protocol below catches both.

## The mandatory sequence

Run this end-to-end for every RSA. No partial completion.

### Step 1 — Click Save

The Save button is at the top of the editor. Click it. Single click.

### Step 2 — Wait for the confirmation toast

A green or grey toast appears at the bottom of the screen with text like "Ad saved" or "Changes saved". Wait until this appears before doing anything else.

If the toast does not appear within 5 seconds:
- Check for an error toast (red). If present, read it and address the cause.
- Check the Save button — if it's still active (not greyed out), the click didn't register. Click again.
- Do not move on until you've seen a successful save toast.

### Step 3 — Screenshot the save confirmation

`computer.screenshot` immediately. Save the screenshot path to the spec under "Save confirmation screenshot links". This is the audit trail — proof of the save event with timestamp and final field state visible.

### Step 4 — Navigate to the Ads list view

URL: `https://ads.google.com/aw/ads?campaignId=<id>&adGroupId=<id>`

Wait 4 seconds for the table to render.

### Step 5 — Read the ad strength column

The Ads list view has an "Ad strength" column showing one of: Poor / Average / Good / Excellent. This is the ground truth, NOT the editor's live meter (which can lag or show stale state).

**Required outcome:** Good or Excellent.

If it shows Poor or Average:
- Open the editor on this ad
- Run the keyword coverage check against the spec
- Identify which keyword(s) are missing
- Replace the weakest non-essential headline using `angular-interaction-patterns.md` Pattern A (modal)
- Save, then re-run this protocol from Step 1

### Step 6 — Read back every headline

Open the editor on the saved ad. Open the "Choose up to 15 headlines" modal. Read every chip's text. Compare against the spec line by line.

For each headline:
- Spec text matches chip text exactly → ✅
- Any difference (extra space, wrong word, truncation) → ❌

Any ❌ = re-edit using Pattern A, save, re-run this protocol from Step 1.

This step has caught silent Angular failures that would otherwise have shipped to the live ad. Do not skip it because the editor showed the right text earlier — the editor view can show one value while the saved value differs.

### Step 7 — Read back all 4 descriptions

Same protocol as Step 6, for the descriptions.

### Step 8 — Update the spec

In the spec file's "Build outcome" section:
- Record the final ad strength (Step 5 value)
- Link the save confirmation screenshot path (Step 3)
- Note any discrepancies caught at read-back (Steps 6–7) — these become postmortem entries
- Add a timestamp

The spec now functions as the as-built record.

## Editor meter vs Ads list column — which is ground truth?

| Surface | When it updates | Reliability |
|---|---|---|
| Editor's live ad strength meter | After each registered keystroke | Can show stale value if Angular missed a keystroke; can show un-saved value |
| Ads list view "Ad strength" column | Only after a successful Save | Reflects the actually-saved state — ground truth |

When the two disagree after save, trust the list view. Re-open the editor and re-verify.

## When the protocol can be abbreviated

Never for first-time RSA builds. Never for client deliveries. Never when ad strength changed during the session.

Acceptable to skip Step 6/7 (full read-back) only when:
- The RSA was untouched in this session (e.g. you opened it just to confirm something else)
- AND the Ads list column already shows Good/Excellent
- AND you've previously read back this exact RSA in a prior session

In any other case, full sequence end-to-end. The 3 minutes of read-back is cheap relative to a wrong headline running live for a week.
