# Google Ads Campaign Build — Postmortem

> Copy this file after every campaign build, even ones that went smoothly. Rename to `<client>-<campaign>-postmortem-YYYY-MM-DD.md`. Each postmortem feeds back into the skill — if the same issue appears twice across builds, the skill needs updating.

## Build summary

| Field | Value |
|---|---|
| Client | |
| Campaign | |
| Spec file | (link) |
| Build start | |
| Build complete | |
| Sessions used | |
| Total time | (hours) |
| Estimated time per spec | (hours) |
| Variance | (positive = over, negative = under) |
| Final outcome | (Live / Paused / Rejected / Partial) |

## What went well

> Plain prose. What worked first time. Patterns to keep.

---

## What went wrong

Duplicate the block below per distinct issue. Tag each issue against the skill section it relates to (`Phase 1`, `Phase 2 step 7`, `anti-pattern`, etc.) so future skill updates are easy to locate.

### Issue: `<short title>`

| Field | Value |
|---|---|
| Skill section | (e.g. Phase 2 step 7 — modal-first headline entry) |
| Severity | (Low / Medium / High — High = repeated past defect or major time loss) |
| Sessions affected | |
| Time lost | (estimate) |
| First seen | (this build / known recurrence) |

**What happened:**

> Concrete description. Include URLs, ad/ad group IDs, exact field values where relevant.

**Root cause:**

> Why it happened. Distinguish symptom from cause. If unclear, flag as "unknown — investigate before next build".

**What eventually worked:**

> The fix or workaround. Include exact steps if reproducible.

**Skill improvement proposed:**

> One of:
> - Update existing section (specify which, with the proposed wording)
> - Add new anti-pattern (specify the wording)
> - Add new reference doc (specify topic and what it should contain)
> - No change needed (one-off, low recurrence risk)

---

## Patterns observed

> If multiple issues in this build trace to the same underlying cause, note it here. This is where compounding insight lives.

---

## Skill updates triggered

Check after writing this postmortem. Any "High" severity issue, or any issue marked "known recurrence", triggers a skill update.

- [ ] No updates needed
- [ ] SKILL.md sequence updated — section: `____`
- [ ] New anti-pattern added — wording: `____`
- [ ] Reference doc added/updated — file: `____`
- [ ] Spec template updated — field: `____`
- [ ] Postmortem template updated — section: `____`

## Time-to-completion analysis

| Phase | Estimated | Actual | Variance | Why |
|---|---|---|---|---|
| Phase 1 (pre-editor spec + coverage check) | | | | |
| Phase 2 (in-editor build) | | | | |
| Phase 3 (save + verify) | | | | |
| Phase 4 (postmortem) | | | | |
| **Total** | | | | |

A build that comes in under estimate is as informative as one that runs over — capture why either way.
