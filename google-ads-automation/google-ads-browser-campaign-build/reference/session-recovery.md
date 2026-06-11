# Session Recovery — Chrome MCP + Google Ads

> Consult when the Chrome MCP tab group is lost, the RSA editor URL renders blank, or session params look stale. These are predictable failure modes — the recovery procedures here are deterministic, not guesses.

## Failure 1 — Chrome MCP tab group lost mid-session

**Symptom:** `tabs_context_mcp` returns no active group. Subsequent navigation calls fail.

**Recovery:**

1. Call `tabs_context_mcp` with `createIfEmpty: true` to start a fresh group
2. Navigate to `https://ads.google.com/aw/overview` (the workspace root)
3. Wait 3 seconds for SPA hydration
4. Pick the correct account from the selector
5. Wait 4 seconds for the dashboard to fully load
6. Only now navigate to the deeper URL you actually wanted (campaign view, ad edit, etc.)

**Prevention:** At the start of any Google Ads build session, **always** call `tabs_context_mcp` first to verify a group exists, before any navigation.

## Failure 2 — RSA editor URL renders blank content area

**Symptom:** You navigate to `https://ads.google.com/aw/ads/edit/search?campaignId=X&adId=Y` and the page loads with the Google Ads header and side nav, but the content area is empty. Sometimes the tab title shows the correct account name; sometimes it shows just "Google Ads". The page does not error — it just doesn't render the form.

**Root cause:** The Google Ads SPA needs an active session state established by a full navigation through a standard view before the deep-link edit route hydrates. Direct deep-link to the editor URL in a freshly-created tab group skips this hydration step.

**Recovery:**

1. Navigate to the ads list view first: `https://ads.google.com/aw/ads?campaignId=X&adGroupId=Y`
2. Wait 4 seconds for the table to render
3. Click the ad row's edit action — this opens the editor with the SPA properly hydrated
4. Do NOT retry the direct edit URL a second time — it will fail again

**Prevention:** When entering an editor in a new tab or new tab group, always go via the ads list view first. Never link directly to the edit URL as the first action.

## Failure 3 — Session params (`__u`, `__c`, `ocid`) stale

**Symptom:** A URL that worked in session 1 returns a redirect to the account picker, an error toast, or a blank page in session 2.

**Root cause:** Google Ads URLs include session-scoped opaque parameters that expire when the user session refreshes. These params are NOT durable identifiers.

**Recovery:**

1. Strip the opaque params (`__u`, `__c`, `__e`, `ocid`, `f.sid`) from the URL
2. Keep only the durable params: `campaignId`, `adGroupId`, `adId`, `authuser`
3. Re-navigate with the clean URL
4. If still blank, fall through to Failure 2 recovery (route via ads list)

**Prevention:** Never save full URLs as references. Save the durable params separately (in the spec file) and reconstruct the URL when needed:

```
https://ads.google.com/aw/ads?campaignId=<id>&adGroupId=<id>&authuser=0
```

## Failure 4 — Account picker shows when you expected the workspace

**Symptom:** Navigation to a workspace URL drops you onto the account picker instead of the campaign view.

**Recovery:**

1. Click the correct account in the picker
2. Wait for full dashboard load (4 seconds minimum)
3. Re-navigate to the originally intended URL — it will work now that an account context is set

This happens after sign-in, after session expiry, and sometimes after switching tab groups. Not a bug — just an extra hop.

## Failure 5 — Captcha appears

**Symptom:** A reCAPTCHA challenge appears during account creation, signup wizards, or after suspicious activity flags.

**Recovery:** Claude does not solve captchas. Stop, screenshot the captcha state, and hand the session to the user to complete the challenge. Do not retry the action in hopes of bypassing the captcha — it will reappear and may trigger additional security checks against the account.

## URL fallback cheat sheet

Save these in the spec for every campaign you build:

| What | URL pattern |
|---|---|
| Workspace root | `https://ads.google.com/aw/overview` |
| Campaign view | `https://ads.google.com/aw/campaigns?campaignId=<id>` |
| Ad group view | `https://ads.google.com/aw/keywords?campaignId=<id>&adGroupId=<id>` |
| Ads list view | `https://ads.google.com/aw/ads?campaignId=<id>&adGroupId=<id>` |
| API Center | `https://ads.google.com/aw/apicenter` |
| Manager accounts list | `https://ads.google.com/aw/manager-accounts` |
| Account settings | `https://ads.google.com/aw/settings/account` |

All durable. None require session params to work.
