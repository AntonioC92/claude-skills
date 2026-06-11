# Google Ads Campaign Build — Issues & Lessons Log
**Campaign:** Campaign #2 — Assessments  
**Purpose:** Feed into the automated Google Ads build skill to prevent recurrence  
**Sessions covering this build:** 3+ sessions, June 2026

---

## 1. Ad Strength Never Reached "Good" Automatically

### What happened
The Cost Optimisation RSA sat at "Average" across two full sessions. It never resolved to "Good" on its own, even after all 15 headlines and 4 descriptions were entered.

### Root causes identified
- **Angular SPA change detection:** Google Ads runs as an Angular single-page app. When Claude types into a headline field programmatically, Angular's reactive form does not always register the change. The value appears in the field visually but the internal model stays stale. This means ad strength recalculates against the old value, not the new one.
- **"Include popular keywords" suggestion is dynamic:** The keywords Google surfaces under this suggestion change between sessions and after every save. Covering the keywords shown in session 1 did not satisfy the suggestion in session 2 because different keywords were shown.
- **Spec headlines did not cover all campaign keywords:** The spec's 15 headlines for Cost Optimisation did not include exact coverage of "aws cost analysis" or "aws bill analysis", both of which are phrase-match campaign keywords. Google surfaced these as uncovered. Two headlines had to be replaced outside the original spec to satisfy the suggestion.

### What eventually worked
1. Replaced H7 ("Find AWS Waste Quickly") with "AWS Bill Analysis" — typed directly via triple-click to select + type.
2. Ad strength jumped to "Good" immediately after this single change.
3. Coverage of "aws cost analysis" via H12 replacement was not needed after H7 resolved the "Include popular keywords" partial.

### Skill improvement needed
- Before saving any RSA, check whether "Include popular keywords" is fully satisfied (blue checkmark, not partial).
- Cross-reference the campaign's keyword list against the RSA headlines. Any phrase-match keyword that does not appear as a substring in at least one headline should trigger a warning.
- If ad strength is "Average" after 15 headlines are entered, identify the specific uncovered keywords and replace the lowest-value spec headline with a headline that covers them.
- Do not treat ad strength as a "nice to have" check at the end. Run it iteratively as headlines are entered.

---

## 2. Angular Form Change Detection Failures

### What happened
Claude typed into RSA headline fields but Angular did not always register the new value. The field showed the new text visually but the strength meter and character counter did not update. Saving at this point would have written the old value to the ad.

### Approaches tried
- Typing directly after triple-click: sometimes worked, sometimes didn't register in Angular.
- Pressing space then backspace after typing: intended to trigger Angular `(input)` event, inconsistent.
- Pressing Tab after typing: accidentally triggered the "Choose up to 15 headlines" modal instead of moving focus.

### What eventually worked
- Triple-click to select all existing text, then type new text directly. When the character counter updated, Angular had registered the change.
- Using the **"Choose up to 15 headlines" modal inline edit** (✏️ icon on each chip) — this is a separate edit flow inside the modal that uses its own form, which registers changes reliably. This was the most reliable path for bulk headline edits.

### Skill improvement needed
- After typing into a headline field, verify the character counter has updated before moving on. If it has not changed, the Angular model has not registered the input.
- Prefer the modal approach for bulk headline corrections: click a headline field, type one character, press Tab to trigger the "Choose up to 15 headlines" modal, then use the ✏️ inline edit icons on each chip.
- Never rely on a single keystroke (space/backspace) to trigger Angular re-evaluation. Use a full triple-click + retype cycle.

---

## 3. Spec Headline Discrepancies Not Caught Until Mid-Session

### What happened
Four headlines in the Cost Optimisation RSA were built with different text than the spec. This was not discovered until session 2 when the modal was accidentally opened and the headline chips were visible side-by-side.

**Discrepancies found:**
| Position | Built as | Spec required |
|---|---|---|
| H7 | Spot Hidden Cloud Waste | Find AWS Waste Quickly |
| H8 | Save 20% on Cloud Costs | AWS Spend Cut by 20%+ |
| H10 | Bespoke Cost Savings Plan | Free Cost Optimisation Plan |
| H11 | Appeared correct | Trusted by TPG & Cloudec (confirmed) |

### Root cause
No verification step after building the ad that compared the saved headline text against the spec line by line.

### Skill improvement needed
- After saving an RSA, read back the current headline values from the editor and compare them against the spec exactly.
- Flag any mismatch before moving on to the next ad group.
- Add a post-save verification step to the build sequence that runs this check automatically.

---

## 4. Session Continuity — Unsaved Work Lost Across Sessions

### What happened
Session 1 changed H4 to "AWS Cost Optimisation" and achieved "Good" ad strength but did not click Save. The session ended. Session 2 reopened the editor and the change was gone.

In this session, the browser tab group was lost (Chrome extension disconnected), which forced a tab group recreation. The RSA editor URL with session-specific params (`__u`, `__c`, `ocid`) failed to render content in the new tab group, requiring navigation to the ads list first.

### Skill improvement needed
- **Always save immediately after achieving Good or Excellent ad strength.** Do not leave the editor open with unsaved changes.
- After saving, take a screenshot of the confirmation to record the final state.
- If the RSA editor URL with session params fails to render content (blank content area, only header showing), navigate to `/aw/ads?campaignId=...&adGroupId=...` first, then click edit on the ad from the ads list. Do not retry the direct editor URL more than once.
- Record ad IDs, ad group IDs, and campaign IDs in the build spec from the start so the edit URL can be reconstructed at any point.

---

## 5. Google Ads Editor URL Reliability

### What happened
The direct RSA editor URL (`/aw/ads/edit/search?campaignId=...&adId=...`) rendered a blank content area in a new tab group, even though the URL was technically valid and the tab title showed "Kubiieo - Google Ads". The page header loaded but no form content appeared.

### Root cause
The Google Ads SPA requires an active session state in the tab group. When the tab group was recreated, the session cookies existed but the SPA needed a full page navigation from a standard Ads view before the editor route would hydrate correctly.

### Skill improvement needed
- If the RSA editor content area is blank after 10 seconds: navigate to `/aw/ads?campaignId=X&adGroupId=Y` first. Wait for full load. Then open the ad edit from the ads table row. This reliably hydrates the SPA before the edit view loads.
- Do not navigate directly to the edit URL as the first action in a new tab or tab group.

---

## 6. Tab Group / Chrome Extension State

### What happened
The Chrome extension tab group was lost mid-session. `tabs_context_mcp` returned no active group. Had to call `tabs_context_mcp` with `createIfEmpty: true` to start a new group, then re-navigate to Google Ads from scratch.

### Skill improvement needed
- At the start of any Google Ads build session, call `tabs_context_mcp` to verify a tab group exists before any navigation.
- If the group is gone mid-session, use `tabs_context_mcp` with `createIfEmpty: true`, then navigate to the campaign view before attempting any deep-link to the editor.
- Save a bookmark of the campaign ads list URL (without session params) as a fallback: `https://ads.google.com/aw/ads?campaignId=X&adGroupId=Y&authuser=N`

---

## 7. "Include Popular Keywords" Suggestion Is Unpredictable

### What happened
The keywords shown under "Include popular keywords in your headlines" changed multiple times:
- Session 1: showed "aws cost management consulting" (not covered)
- Session 2 after modal save: showed "aws cost analysis" and "aws bill analysis" (not covered)

Each time ad strength was close to resolving, a new uncovered keyword appeared.

### Root cause
Google dynamically selects which campaign keywords to highlight based on search volume and relevance at that moment. It is not a static list.

### Skill improvement needed
- Do not rely on the "Include popular keywords" suggestion to tell you which keywords need coverage. Instead, proactively cross-reference every campaign keyword against the RSA headlines before entering a single headline.
- Run a pre-build check: for each phrase-match keyword in the ad group, does any RSA headline contain the keyword's primary term as a substring? If not, plan a headline that covers it.
- This check should happen at spec review time, before opening the Google Ads editor.

---

## 8. No Automated Post-Save Verification

### What happened
Multiple times work was done in the editor and the save button was not clicked before the session ended or the tab closed. There was no final confirmation step that verified the saved state matched the intended state.

### Skill improvement needed
- The last action in any ad edit session must be: (1) click Save, (2) wait for the confirmation toast, (3) take a screenshot, (4) read back the ad strength from the overview.
- Add a checklist step to the build skill that explicitly requires a saved-state screenshot before marking the RSA as complete.
- After saving all RSAs, navigate to the Ads list view and confirm ad strength is shown as "Good" or "Excellent" in the table column. This is the ground truth, not the editor's live calculation.

---

## 9. "Tab Key Opens Modal" Behaviour

### What happened
Pressing Tab after typing in a headline field did not move focus to the next field. Instead, it triggered the "Choose up to 15 headlines" modal. This was unexpected and caused a detour.

### Note
The modal turned out to be useful for bulk edits. But the trigger is non-obvious and should be documented.

### Skill improvement needed
- Document in the skill: pressing Tab in a Google Ads RSA headline field when 15 headlines are already filled opens the "Choose up to 15 headlines" modal. This modal shows all headlines as chips with ✏️ inline edit buttons. It is actually the best UI for making multiple headline corrections at once.
- To edit a headline chip in the modal: click ✏️, type new text in the popup field, verify character count updates, click Save on the popup. Then click Save on the modal.
- Use this modal intentionally for any session where 3 or more headlines need correction.

---

## 10. Build Sequence Issues — What Should Change

### Current sequence (from spec)
1. Create ad group
2. Add keywords
3. Enter RSA headlines and descriptions
4. Check ad strength
5. Save

### Problems with this sequence
- Step 4 happens after all 15 headlines are entered, which means discovering an "Average" rating late with no easy path to fix it.
- No pre-check of keyword vs headline coverage before entering the editor.
- No post-save verification step.

### Recommended sequence for the skill
1. **Pre-build keyword audit:** For each ad group, list all phrase-match keywords. Identify which keywords are covered by the spec headlines (substring match). Flag any keyword without coverage and plan a replacement headline before opening the editor.
2. Create campaign and ad group settings.
3. Add keywords (phrase + exact).
4. Enter RSA headlines — after every 3rd headline, check if the ad strength is updating.
5. After 15 headlines entered: confirm "Include popular keywords" suggestion shows all green or fully satisfied. If partial, identify the uncovered keyword and swap the weakest non-essential headline.
6. Enter 4 descriptions.
7. Verify ad strength is "Good" or "Excellent" in the live meter.
8. Click Save. Wait for confirmation toast.
9. Screenshot the saved ad. Navigate to the Ads list. Confirm ad strength shows in the table.
10. Move to next ad group.

---

## Summary Table

| # | Issue | Impact | Fix |
|---|---|---|---|
| 1 | Ad strength stuck at Average | 2+ sessions wasted | Pre-check keyword vs headline coverage before entering editor |
| 2 | Angular change detection failures | Incorrect values silently left in fields | Triple-click + retype; verify character counter updates |
| 3 | Spec headlines not verified after save | 4 wrong headlines in live ad | Post-save read-back and spec comparison |
| 4 | Unsaved work lost between sessions | Had to redo changes from session 1 | Save immediately after achieving Good; screenshot confirmation |
| 5 | Direct editor URL fails in new tab group | Blank editor, wasted wait time | Navigate to ads list first, then open edit |
| 6 | Tab group lost mid-session | Had to rebuild session state | Check tab group exists at session start |
| 7 | "Popular keywords" suggestion changes dynamically | Endless chase to cover new keywords | Proactive cross-reference at spec stage, not inside the editor |
| 8 | No post-save verification step | Unknown if changes were actually saved | Mandatory save + screenshot + list-view strength confirmation |
| 9 | Tab key opens modal unexpectedly | Detour; though modal is useful | Document and use modal intentionally for bulk edits |
| 10 | Build sequence has no pre-checks | Problems found too late | Reorder sequence: audit keywords first, verify after each group |
