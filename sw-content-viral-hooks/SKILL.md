---
name: sw-content-viral-hooks
description: Streetwise hook generator. Triggers whenever Antonio or Sean is writing LinkedIn content, sales outbound openers, Upwork proposal first lines, blog post intros, video scripts, email subject lines, or any short opener that needs to stop a reader. Reference patterns are extracted from a 46,606-row TikTok viral-hooks dataset (top-outlier performers only), translated into Streetwise operator voice with our actual numbers from CLAUDE.md §7. Triggers on phrases like "write a LinkedIn post", "draft a hook", "open this with something punchy", "first line of the proposal", "what should the headline be", "make this scroll-stopping", "give me three hooks", "rewrite the intro", "this post is flat", or any request to produce or rework an opener for marketing content.
---

# sw-content-viral-hooks

## Purpose

Generate opener hooks that match the Streetwise voice (operator-to-operator, numbers-first, no fluff) using structural patterns proven viral in the social-media dataset at `./hooks-*.md`.

This skill is **not a copy machine**. It does not output rewordings of TikTok hooks. It extracts the *structure* that made those hooks viral and re-expresses it through Streetwise voice and real Streetwise case studies.

## When to use

- Sean writing a LinkedIn post or sales-led outbound opener
- Antonio writing a founder LinkedIn post, blog intro, or case-study lead
- Any first line for: Upwork proposals, cold email subjects, video scripts, deck title slides, ad creative headlines, landing-page heroes
- Any time someone says "this is flat", "needs more punch", "rewrite the open", "what's the hook"

## Mandatory pre-checks (run BEFORE generating)

1. **Identify the Service Line** (CLAUDE.md §5) the content is about. Default hook patterns are mapped per service line — see the table below.
2. **Identify the Strategic Pillar** (CLAUDE.md §3). Pillar 1 (Build the Brand) content uses different hook angles than Pillar 2 (Grow Existing Client Outcomes) — case studies vs founder POV.
3. **Locate one concrete proof asset** the post will reference. Options: a specific case study from §7 (Dublin Beer Festival €4,639→€72K, Jobbio Career Fair €30K→€135K + 60 MQLs, SaaS A/B test CPL £149→£34.70, Media Client Summit, DP Gates $1,995, Edelweisshütte). If no proof exists, generate one POV-led hook instead of a results-led hook.
4. **Confirm the author voice**: Sean (sales/outbound), Antonio (paid media/automation), or Streetwise (agency). Voice register shifts slightly between the three.

## The 5 hook patterns (load `pattern-templates.md` for full examples)

| Pattern | Structure | Best for service lines |
|---|---|---|
| 1. Specific number → specific outcome | `[Spend/input number]. [Result number]. [One-line mechanism].` | Paid Media, Live Events, AI Workflows |
| 2. Provocative reframe | `[Common belief]. [Sharp contradiction].` | LinkedIn Outreach, Sales Enablement, Strategy |
| 3. Inside-the-room | `[Specific named asset / mechanism] + [why it works].` | All services — show the work |
| 4. Confession with a turn | `[Specific past mistake]. [Specific lesson + new approach].` | Founder POV, Strategy, Sales Enablement |
| 5. Pattern interrupt | `[Fragment / single word / number stack].` then full post. | High-stakes posts, milestones, case-study reveals |

## Service-line → pattern mapping

| Service Line | Preferred patterns | Avoid |
|---|---|---|
| Paid Media (Antonio) | 1, 3, 5 | 4 unless the post is overtly founder-positioning |
| LinkedIn Outreach (Sean) | 2, 4, 1 | 5 — too dramatic for outbound openers |
| Email Automation (Antonio) | 3, 1 | 5 |
| SEO & UX (Antonio) | 3, 1 | 2 unless the post is contrarian on SEO orthodoxy |
| AI Workflows (Antonio) | 3, 4, 1 | 2, 5 |
| Live Events (Sean) | 1, 5 | 4 |
| Sales Enablement (Sean) | 2, 4 | 5 |
| Strategy & Branding (joint) | 4 | 1 without a strategy-specific number |

## Mandatory output checks (run AFTER generating, BEFORE returning)

A hook FAILS the check if ANY of these is true. If it fails, regenerate.

1. **Contains a banned word from CLAUDE.md §6**: game-changing, unlock, supercharge, 10x, next-level, synergy, ecosystem, holistic, best-in-class, leverage (as a verb), delve, moreover.
2. **Uses an exclamation mark** (anywhere — even at the end).
3. **Opens with "We help businesses..." or any variation** of that agency-cliché form.
4. **Uses em-dashes (—) inside marketing prose** (AI tell per §6).
5. **Generic without a specific number, named channel, named client, or specific mechanism**. If the hook could plausibly belong to any agency, it's wrong.
6. **References a Streetwise case study claim NOT in CLAUDE.md §7**. We don't invent numbers.
7. **Emojis at the front of the hook** in any formal context (proposals, decks). Light social use OK on LinkedIn but never as the first character.

## Default output format

When asked for hooks, return **3 options** unless the user asks for a specific number:

```
HOOK 1 (Pattern X — for [audience/channel]):
[the hook]

HOOK 2 (Pattern Y — for [audience/channel]):
[the hook]

HOOK 3 (Pattern Z — for [audience/channel]):
[the hook]

For each: one-line note on why this opener works for this audience.
```

If the user asks for a full post, return ONE hook + the full post body using the chosen pattern.

## Source files in this skill

- `hooks-business.md` — top-80 Business-category viral hooks (outlier ≥ 30)
- `hooks-productivity.md` — top-80 Productivity-category viral hooks
- `hooks-finance.md` — top-80 Finance-category viral hooks
- `hooks-dev-tools.md` — Dev Tools category (small but high-quality)
- `pattern-templates.md` — the 5 patterns with Streetwise translation examples and real-case-study applications

Load `pattern-templates.md` whenever invoking this skill. The reference hook files are for occasional inspection — do not dump them into output, they are noisy in isolation.

## What NOT to use this skill for

- Long-form blog posts beyond the first 1-3 lines (use `marketing:draft-content` or `content-antonio`)
- Proposal bodies (use `sw-proposal-format-router`)
- Subject lines longer than 8 words (use `marketing:email-sequence`)
- Anything for a regulated industry (finance / health / legal) where claim-substantiation matters more than hook punch — flag and refer to a human reviewer

## Voice cross-check

This skill MUST defer to `sw-content-voice-antonio` when the author is Antonio, and `sw-content-voice-sean` when the author is Sean. If those skills are loaded, their voice rules override anything ambiguous here. The hook patterns in this file are voice-agnostic structural shells.
