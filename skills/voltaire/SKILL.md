---
name: voltaire
description: Use when the user wants to optimize their paywall or improve conversion rates. Triggered by: "fix my paywall", "optimize my paywall", "/voltaire", "improve conversions".
---

# Voltaire — Paywall Optimization

## Overview
Voltaire is a revenue intelligence layer. It tracks paywall events, computes conversion metrics, benchmarks them against industry data, and exposes everything via MCP. Your job is to pull that data, explore the codebase, and apply a concrete fix.

## Available MCP tools
- `mcp__voltaire__create_app` — first-run bootstrap: create the app (name, category). Returns the API key.
- `mcp__voltaire__setup` — connect Stripe: takes `stripe_secret_key`, optionally `github_repo_url`
- `mcp__voltaire__get_stats` — current state: revenue connection, SDK status, conversion rate, data volume
- `mcp__voltaire__analyze_paywall` — full data dump: conversion vs benchmark, revenue, behavioral metrics, 7-day trend, previously applied fixes
- `mcp__voltaire__get_recommendation` — latest weekly recommendation (Pro users only)
- `mcp__voltaire__mark_applied` — log what was changed so future runs have full context

## Workflow

The only time you stop and wait is when you need something from the user (app name, Stripe key). Everything else runs in one session without interruption.

First, call `mcp__voltaire__get_stats` and determine which mode you're in:

---

### First run (Stripe not connected OR SDK not installed)

1. **App not created** → call `mcp__voltaire__create_app`. Ask for name + category if needed, then immediately continue.
2. **Stripe not connected** → look for `STRIPE_SECRET_KEY` in backend `.env` files (`backend/.env`, `server/.env`, `api/.env`, etc. — Stripe keys are never in the frontend). If found:
   - **Check if the key starts with `sk_test_`** → stop and tell the user: "This is a Stripe test key. Voltaire needs your live key to analyze real revenue. Find it at dashboard.stripe.com/apikeys → Secret key (`sk_live_...`). Without it, there's no real data to work with." Wait for the live key before continuing.
   - If the key starts with `sk_live_` → call `mcp__voltaire__setup` — **do not display the key** — then immediately continue.
   - If no key found → ask the user. Same test key check applies to whatever they paste.
3. **Analyze** — call `mcp__voltaire__analyze_paywall` + explore the codebase to find the paywall. Stripe data alone is enough.
4. **Install SDK if missing** — `npm install voltaire-sdk`, write `VOLTAIRE_API_KEY` to `.env`, create the init file, add the 7 tracking calls (see SDK section below).
5. **Fix** — propose a concrete change, wait for confirmation, apply it.
6. **Log** — call `mcp__voltaire__mark_applied` for the SDK install and for the fix.
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

**2. Add the API key to `.env`** — `create_app` returned it. Write it yourself into the project `.env` file. Don't ask the user to do it.
```
VOLTAIRE_API_KEY=volt_xxx
```

**3. Create `src/voltaire.ts`** (or `.js`) — initialize once at app startup:
```ts
import Voltaire from 'voltaire-sdk'
Voltaire.init({ apiKey: import.meta.env.VITE_VOLTAIRE_API_KEY })
export default Voltaire
```
Adapt the env var name to the framework (`process.env.VOLTAIRE_API_KEY` for Node, `import.meta.env.VITE_VOLTAIRE_API_KEY` for Vite, etc.)

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

Find the right locations by reading the codebase — don't add them blindly. After installing, call `mcp__voltaire__mark_applied` with `type: "sdk"`.

**There is no web dashboard.** All data is surfaced through the MCP tools above. Never tell the user to "check the dashboard" or "visit the Voltaire app" — everything goes through `/voltaire` in Claude Code.

## Rules
- Never apply changes without explaining what you found first
- Always show paywall location, root cause, and proposed change before editing
- `analyze_paywall` returns raw data — you reason about it, don't expect a pre-computed diagnosis
- Always call `mark_applied` after every fix — this is what builds context over time
- Never mention a "dashboard" — Voltaire has no web UI, all output is through MCP tools
