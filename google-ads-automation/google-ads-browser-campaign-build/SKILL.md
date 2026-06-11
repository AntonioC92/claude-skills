---
name: google-ads-browser-campaign-build
description: Build, edit, or verify Google Ads campaigns directly in the Google Ads web UI via browser automation. Use when creating campaigns, ad groups, keywords, and responsive search ads (RSAs); fixing existing campaigns; or running pre-build keyword coverage audits and post-save verification. Triggers on phrases like "build a Google Ads campaign", "add an ad group", "create an RSA", "fix this Google Ads campaign", "set up Google Ads for [client]", "rebuild Google Ads", "Google Ads spec", "RSA spec", or any request involving Google Ads editor interaction. Optimized for browser-based work driven by Claude in Chrome; the Angular SPA quirks documented here are specific to programmatic interaction and do not apply to humans editing manually.
type: build-protocol
---

# Google Ads Browser Campaign Build

This skill governs how Google Ads campaigns are built and edited through the web UI, not through the API. It exists because the Google Ads editor is an Angular SPA with predictable failure modes when driven programmatically, and because the "Include popular keywords" suggestion is dynamic and cannot be chased reactively — both of which have historically wasted multiple sessions per build.

The skill applies whenever Claude is touching the Google Ads editor directly: campaign creation, ad group setup, keyword entry, RSA build, descriptions, sitelinks, callouts. It does NOT apply to API-based reads (use the Google Ads API workflow for reporting) or to bulk uploads via Google Ads Editor desktop app (different surface, different rules).

## Top-level principle

Every build defect found in this skill's history traces to one of three failures:

1. **Spec gap.** Headlines didn't cover all campaign keywords. The fix lives in the spec, not the editor.
2. **Silent input failure.** Angular didn't register a programmatic keystroke. The field shows the new text but the model is stale.
3. **No verification.** Work was saved (or wasn't) without reading back what's actually in the live ad.

Every step of this skill addresses one of those three. If you find yourself doing something not in the sequence below, you are probably about to recreate a known defect.

## The build sequence

This is the canonical order. Skipping or reordering steps reintroduces past failures.

### Phase 1 — Pre-editor (do this BEFORE opening Google Ads)

**Step 1. Fill `templates/campaign-build-spec.md` end to end.** Every field. No placeholders. Currency, geo, language, bid strategy, daily budget, ad group names, full keyword lists by match type, 15 headlines per RSA, 4 descriptions per RSA, final URL, display path, all extensions. The spec is the source of truth — what's not in the spec doesn't get built.

**Step 2. Run the keyword coverage check.** For every ad group: list every phrase-match and exact-match keyword. For each keyword, identify which headline(s) contain its primary term as a substring. If any keyword has zero coverage, **block the build** and revise headlines until every keyword has at least one covering headline. This single step prevents the "Include popular keywords" chase entirely.

> The spec template includes a coverage table — fill it as you write the headlines, not after.

**Step 3. Record IDs early.** Campaign ID, ad group IDs, ad IDs. Add them to the spec the moment Google generates them. Editor URLs can fail to hydrate in new tab groups (see `reference/session-recovery.md`); having IDs lets you reconstruct any URL at any moment.

### Phase 2 — In-editor build

**Step 4. At session start, verify Chrome MCP tab group exists.** Call `tabs_context_mcp`. If no group, create one with `createIfEmpty: true`. Never start work with an uninitialised group — when it dies mid-session you'll lose the URL trail (see `reference/session-recovery.md`).

**Step 5. Create campaign and ad group from the spec.** Standard editor flow. If any of the settings that lock on save (currency, time zone, country) need user input, stop and confirm with Antonio — these are unchangeable.

**Step 6. Add keywords (phrase, exact, then negatives in that order).** Paste in bulk via the keyword input box. Verify the count after paste matches the spec count exactly.

**Step 7. Enter RSA headlines using the modal flow, not direct field typing.** Direct typing into the 15 headline fields triggers Angular change-detection failures roughly 1 time in 4 — see `reference/angular-interaction-patterns.md`. The reliable path:

- Fill the first headline with a single character via direct field click
- Press Tab → this opens the "Choose up to 15 headlines" modal
- In the modal, use the ✏️ inline edit button on each chip to enter each headline
- Each chip edit happens in its own popup with its own form, which registers reliably
- After each chip save, verify the character count matches what you intended

If you must use direct field entry: **triple-click → type → verify the character counter updated**. If the counter didn't move, the model didn't register. Triple-click + retype.

**Step 8. After every 3 headlines, check ad strength in the live meter.** Don't wait until 15. If the meter doesn't move after a headline that should have improved it, the model didn't register the change — go back and re-enter using triple-click + retype.

**Step 9. After all 15 headlines entered, ignore the "Include popular keywords" suggestion.** If you did Step 2 properly, coverage is already there. The suggestion is dynamic and will surface different keywords across sessions — chasing it is wasted motion. The ground truth is your pre-build coverage table.

**Step 10. Enter 4 descriptions.** Same modal-first pattern preferred. Verify character counters.

### Phase 3 — Save and verify

**Step 11. Click Save. Wait for the confirmation toast. Screenshot it.** This is non-negotiable. Unsaved work has been lost between sessions before — never end a session with an open editor.

**Step 12. Navigate to the Ads list view.** Confirm ad strength column shows "Good" or "Excellent" — this is the ground truth, NOT the editor's live meter (which can show stale state).

**Step 13. Read back the saved headlines.** Open the modal one more time, list every chip's text, and compare against the spec line by line. Any discrepancy = re-edit before declaring done. This catches Angular silent-save defects where the visible text differed from the saved value.

**Step 14. Record the final state in the spec.** Update the spec file with final IDs, confirmed ad strength, and a link to the save-confirmation screenshot. The spec becomes the as-built record.

### Phase 4 — Post-build (after every build)

**Step 15. Write a postmortem using `templates/campaign-build-postmortem.md`.** Capture anything that didn't go as planned, even minor friction. Each postmortem feeds back into this skill — if the same issue surfaces twice, the skill needs updating.

## Anti-patterns (do not do)

- Do not chase the "Include popular keywords" suggestion. Pre-build coverage check is the only correct response.
- Do not rely on the editor's live ad strength meter as ground truth. Use the Ads list view column.
- Do not press Tab to move focus between headline fields when 15 are filled — it opens the modal. Either use this intentionally or click into the next field.
- Do not start a session with `tabs_context_mcp` unchecked.
- Do not navigate to the RSA edit URL directly in a new tab group. Go to `/aw/ads?campaignId=X&adGroupId=Y` first, then click edit from the row. Direct edit URLs fail to hydrate when session params don't match (see `reference/session-recovery.md`).
- Do not leave the editor open with unsaved changes. Save the moment ad strength resolves.
- Do not declare a build complete without the Ads list view confirmation.

## Files in this skill

| File | When to use |
|---|---|
| `templates/campaign-build-spec.md` | Copy and fill before every campaign build. The pre-editor source of truth. |
| `templates/campaign-build-postmortem.md` | Copy and fill after every campaign build. Captures lessons. |
| `reference/angular-interaction-patterns.md` | Consult when typing into headline/description fields is failing or behaving oddly. |
| `reference/session-recovery.md` | Consult when tab group is lost, editor URL is blank, or session params look stale. |
| `reference/verification-protocol.md` | The exact sequence for save + verify + read-back. Follow during Phase 3. |
| `examples/kubiieo-2026-06-lessons.md` | Worked example — the original lessons log that drove this skill. Worth re-reading before any sensitive build. |

## When the skill itself needs updating

If a postmortem surfaces a new issue not covered by the sequence, the anti-patterns, or the reference docs, that's a signal to update this skill. Don't let new failure modes pile up across builds without being absorbed into the protocol.
