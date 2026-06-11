# Angular Interaction Patterns — Google Ads Editor

> Consult when typing into RSA headline, description, or other editor fields is failing or behaving oddly. The patterns here are specific to Claude driving the editor via Chrome MCP — they do not apply to humans editing manually.

## Why this exists

The Google Ads editor is a single-page Angular application with reactive forms. When Claude types into a field via `computer.type` or `form_input`, three things can happen:

1. **Success.** Text appears, character counter updates, ad strength meter recalculates. Continue.
2. **Visual-only success.** Text appears in the field, but the character counter and the strength meter do not update. The Angular model is stale. **If you save here, the OLD value is written to the live ad.** This is the silent-defect class — most expensive to catch later.
3. **Outright failure.** Text does not appear. Rare but happens when the field never received focus.

The character counter is the canonical signal that Angular registered the input. If it didn't move, the model didn't update — no matter what you see on screen.

## The reliable interaction patterns, in order of preference

### Pattern A — Modal-based edit (PREFERRED for bulk)

The "Choose up to 15 headlines" modal contains a chip per headline with a ✏️ inline edit button. Each edit happens in its own popup with its own form, which uses a different (more reliable) change-detection path than the inline fields.

**Use Pattern A whenever 3 or more headlines need entering or correcting.**

Steps:
1. Type a single character into any headline field
2. Press Tab → the modal opens
3. For each chip, click ✏️, type the new headline in the popup, verify character counter updates, click Save in the popup
4. After all chips edited, click Save on the modal
5. Verify the chip text in the modal matches your spec — read every one back

### Pattern B — Triple-click + retype (FALLBACK)

For single headline corrections, or when the modal is not yet open.

Steps:
1. Triple-click the field to select all existing text (selects the entire line, not just one word)
2. Type the new headline
3. **Verify the character counter updated.** If the counter still shows the old length, the model is stale — repeat Pattern B.
4. If a second triple-click + retype still doesn't update the counter, fall back to Pattern A by triggering the modal.

### Pattern C — Direct typing (AVOID)

Clicking into a field and typing without first selecting all is the highest failure-rate path. Avoid unless the field is empty AND you're confident no prior value exists.

## Verification rules

After any field edit:

- **Character counter must reflect the new text length.** Not the old length. If it didn't change, the model is stale.
- **Ad strength meter should recalculate.** It updates within ~1 second. If it doesn't move after entering a headline that should have improved it (e.g. covering a previously uncovered keyword), the model is likely stale.
- **Save button enables.** A disabled Save button after editing is a tell that Angular hasn't registered any change.

If two of the three above don't behave as expected, the model is stale. Re-enter using a different pattern.

## Tab key behaviour

Pressing Tab in a headline field has different effects depending on how many headlines are filled:

- **<15 headlines filled:** Tab moves focus to the next headline field. Normal behaviour.
- **15 headlines filled:** Tab opens the "Choose up to 15 headlines" modal. This is by design — Google surfaces the modal as the bulk-edit UI once the RSA is "full".

This is useful — Pattern A leverages this exact behaviour to open the modal. But it can also surprise you mid-edit, so know which state you're in before pressing Tab.

## Description fields

Descriptions (4 per RSA, ≤90 chars) use the same Angular form pattern as headlines. Same patterns apply. The modal-equivalent for descriptions is the same "Choose up to 4 descriptions" view triggered by the same Tab mechanic.

## When the strength meter lags

The live ad strength meter in the editor updates within a second of a registered change. The Ads list view column updates only after Save. The two can disagree briefly during a build — this is normal. The Ads list value is the ground truth.

If the editor meter shows "Good" but the list view shows "Average" after save, something didn't save. Re-open and read back.

## Failure modes that look like Angular bugs but aren't

- **"Include popular keywords" suggestion changing between sessions** — this is dynamic and intentional. Not a bug. Do not chase it. See SKILL.md anti-patterns.
- **Save button greyed out despite visible edits** — usually because a required field elsewhere is invalid (e.g. final URL malformed). Not a typing issue.
- **Modal won't open on Tab** — typically because the field has invalid content (over char limit, banned characters). Fix the field first.
