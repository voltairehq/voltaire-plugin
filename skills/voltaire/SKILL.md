---
name: voltaire
description: Use when the user wants to optimize their paywall or improve conversion rates. Triggered by: "fix my paywall", "optimize my paywall", "/voltaire", "improve conversions".
---

# Voltaire — Paywall Optimization

## Overview
Voltaire is a revenue intelligence layer. It tracks paywall events, computes conversion metrics, benchmarks them against industry data, and exposes everything via MCP. Your job is to pull that data, explore the codebase, and apply a concrete fix.

## Available MCP tools
- `mcp__voltaire__create_app` — first-run bootstrap: create the app (name, category). Returns the SDK token.
- `mcp__voltaire__setup` — connect Stripe: takes `stripe_secret_key`, optionally `github_repo_url`
- `mcp__voltaire__get_stats` — current state: revenue connection, SDK status, conversion rate, data volume
- `mcp__voltaire__analyze_paywall` — full data dump: conversion vs benchmark, revenue, behavioral metrics, 7-day trend, previously applied fixes
- `mcp__voltaire__get_recommendation` — latest weekly recommendation (Pro users only)
- `mcp__voltaire__mark_applied` — log what was changed so future runs have full context

## Workflow

The only time you stop and wait is when you need something from the user (app name, Stripe key). Everything else runs in one session without interruption.

First, attempt to call `mcp__voltaire__get_stats`.

**CRITICAL — if the tool is not available or the call fails with any error:**
Stop immediately. Output EXACTLY this message, word for word, nothing else:

```
Run `/mcp` in Claude Code, find **voltaire** in the list, and click **Authenticate**. A browser will open — log in with Google or GitHub. Once done, run `/voltaire` again.
```

Do NOT suggest `npx`, do NOT suggest editing settings.json, do NOT improvise. Output only the message above, then stop.

---

If `mcp__voltaire__get_stats` succeeds, determine which mode you're in:

---

### First run (Stripe not connected OR SDK not installed)

**Start with a short, confident intro** — one or two sentences. Tell the user Voltaire is connected, what you can see already (plan, whether Stripe is linked, SDK status from `get_stats`), and what you're about to do. Be direct — not a bullet list, just a sentence.

Then run this sequence **without stopping** unless you need input:

1. **Stripe first** — immediately look for `STRIPE_SECRET_KEY` in backend `.env` files (`backend/.env`, `server/.env`, `api/.env`, etc. — never in the frontend). Don't explore the whole codebase first — go straight to `.env` files.
   - If `sk_test_...`: stop, tell the user they need their live key (`sk_live_...`) from dashboard.stripe.com/apikeys.
   - If `sk_live_...`: call `mcp__voltaire__setup` silently (don't display the key), then continue.
   - If not found: ask the user to paste it. Same test-key check applies.
2. **Analyze** — call `mcp__voltaire__analyze_paywall`. Then find the paywall in the codebase.
3. **SDK** — if not installed, install it now (see SDK section). The SDK token is in `get_stats` — use it directly. If `VOLTAIRE_API_KEY` already exists in the codebase, rename it to `VOLTAIRE_SDK_TOKEN` everywhere.
4. **Fix** — propose a concrete change, wait for confirmation, apply it.
5. **Log** — call `mcp__voltaire__mark_applied` for the SDK install and for the fix.
7. **First-run summary:**
   ```
   Here's what Voltaire set up:
   ✅ Stripe connected — revenue history imported
   ✅ SDK installed — 7 tracking events now collecting data
   ✅ First fix applied — [one line]

   What happens automatically:
   - Every day: digest email with conversion delta
   - Every hour: anomaly detection
   - Every Monday: new recommendation from latest data

   Run /voltaire anytime — the more data flows in, the sharper the analysis gets.
   ```
   If Free plan: "🔒 Weekly recommendation locked. Upgrade to Pro ($49/mo) at https://app.hivoltaire.com/account"

---

### Recurring run (Stripe connected + SDK installed)

Don't repeat setup. Lead with data — let the user drive.

1. **Call `mcp__voltaire__analyze_paywall`**. If on Pro, also call `mcp__voltaire__get_recommendation`.

   **If 0 SDK events received** — ask the user: "No events have been received yet. Did you add `VOLTAIRE_SDK_TOKEN` to your production environment (Render, Vercel, Netlify, etc.)? The `.env` file is local only — the SDK won't initialize in production without the token in your hosting dashboard."

2. **Present a clear state of affairs — always, before doing anything:**
   ```
   Since last run [X days ago]:

   Conversion:  X% → Y%  [↑↓ +Z%]   (benchmark: B%)
   Revenue:     $X this month  [↑↓ vs last month]
   Last fix:    [description] — [impact if measurable, or "still collecting data"]
   [Anomaly alert if flagged]

   Pro recommendation: [one line] — est. +$X/mo  (or 🔒 if Free)
   ```

3. **Wait for the user to decide.** Don't propose a fix automatically. They might want to:
   - Ask questions about the data
   - Ask what's causing a specific pattern
   - Ask for a fix recommendation
   - Say "apply the recommendation" or "fix X"

4. **Once they ask for an action** — propose it clearly (location, change, reason), wait for confirmation, apply it, call `mcp__voltaire__mark_applied`.

5. **After applying:**
   ```
   Fix applied: [one line]
   Check back in [timeframe] to measure impact.
   ```

## SDK installation

When SDK is not installed:

**1. Install the package**
```bash
npm install voltaire-sdk
```

**2. Add the SDK token to `.env`** — `create_app` returned it. Write it yourself into the project `.env` file. Don't ask the user to do it.
```
VOLTAIRE_SDK_TOKEN=vt_xxx
```
⚠️ After writing the token, tell the user: "I've added `VOLTAIRE_SDK_TOKEN` to your `.env` file. If your app is deployed (Render, Vercel, Netlify, Railway, etc.), you also need to add it as an environment variable in your hosting dashboard — `.env` files are never deployed. Without this, the SDK won't initialize in production."

**3. Create `src/voltaire.ts`** (or `.js`) — initialize once at app startup:
```ts
import Voltaire from 'voltaire-sdk'
// Use the right env accessor for this project's framework:
// - Vite/React:  import.meta.env.VITE_VOLTAIRE_SDK_TOKEN
// - Next.js:     process.env.NEXT_PUBLIC_VOLTAIRE_SDK_TOKEN
// - Node/other:  process.env.VOLTAIRE_SDK_TOKEN
// Check package.json and existing env usage in the codebase to determine the correct one.
Voltaire.init({ apiKey: process.env.VOLTAIRE_SDK_TOKEN })
export default Voltaire
```

**4. Add the 7 tracking calls** in the right places:
```ts
import Voltaire from './voltaire'

Voltaire.track('session_started')                          // on app open
Voltaire.track('feature_used', { feature: 'feature_name' }) // on key feature use
Voltaire.track('feature_gate_hit', { feature: 'x' })      // user hit premium gate (blocked)
Voltaire.track('upgrade_clicked')                          // user tapped upgrade CTA
Voltaire.track('paywall_shown')                            // paywall displayed
Voltaire.track('paywall_dismissed')                        // user closed without paying
Voltaire.track('paywall_converted')                        // successful subscription
```

**TypeScript projects:** always call `Voltaire.track()` directly — never wrap it in a function with a custom event string type. If the project has an existing analytics wrapper that defines its own event type, import the SDK's type instead of redefining it:
```ts
import type { EventType } from 'voltaire-sdk'
```
Failure to do this causes TS2345 build errors (`feature_gate_hit not assignable to EventType`).

Find the right locations by reading the codebase — don't add them blindly. After installing, call `mcp__voltaire__mark_applied` with `type: "sdk"`.

**Data lives in MCP tools, not the dashboard.** `app.hivoltaire.com` exists for account and billing only. Never tell the user to "check the dashboard" for their conversion data, SDK token, or recommendations — everything data-related goes through `/voltaire` in Claude Code. The SDK token can be found there if needed, but the MCP flow gives it automatically.

## Rules
- Never apply changes without explaining what you found first
- Always show paywall location, root cause, and proposed change before editing
- `analyze_paywall` returns raw data — you reason about it, don't expect a pre-computed diagnosis
- Always call `mark_applied` after every fix — this is what builds context over time
- Never mention a "dashboard" for data — use MCP tools
