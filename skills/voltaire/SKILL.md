---
name: voltaire
description: Use when the user wants to optimize their paywall or improve conversion rates. Triggered by: "fix my paywall", "optimize my paywall", "/voltaire", "improve conversions".
---

# Voltaire — Paywall Optimization

## Overview
Voltaire is a revenue intelligence layer. It tracks paywall events, computes conversion metrics, benchmarks them against industry data, and exposes everything via MCP. Your job is to pull that data, explore the codebase, and apply a concrete fix.

## Available MCP tools
- `mcp__voltaire__create_app` — first-run bootstrap: create the app (name, category). Returns the SDK token.
- `mcp__voltaire__setup` — connect Stripe: takes `stripe_secret_key`
- `mcp__voltaire__get_stats` — current state: revenue connection, SDK status, conversion rate, data volume
- `mcp__voltaire__analyze_paywall` — returns structured JSON with all raw data. You do all formatting and reasoning.
- `mcp__voltaire__get_recommendation` — latest weekly recommendation (Pro users only)
- `mcp__voltaire__mark_applied` — log what was changed so future runs have full context

## analyze_paywall JSON schema

`analyze_paywall` returns a JSON object. Key fields:

- `app` — name, plan, category, platform
- `lumiere_status` — one of: `"active"` (Pro), `"trial_active"` (14-day trial), `"trial_expired"`, `"locked"`
- `metrics` — cr_30d, cr_7d, sessions_30d, shown_30d, converted_30d, gate_hit_30d, upgrade_clicked_30d
- `revenue` — rev_30d, rev_7d, charges_30d
- `benchmark` — per-metric median + top_quartile, gap_pp vs benchmark
- `behavioral` — bounce_rate_under_3s_pct, avg_dismiss_seconds, sessions_before_convert_median
- `feature_correlation` — list of `{feature, cr_with, cr_without, n}` sorted by cr_with desc
- `trend_7d` — list of `{date, cr_pct, paywall_views, anomaly}`
- `lumiere` — present when `lumiere_status` is `"active"` or `"trial_active"`. Contains:
  - `timing` — `{signal, confidence, explanation, recommendation, estimated_impact}` or null
  - `cohort` — `{key_differentiator, pattern, recommendation, confidence}` or null
  - `churn` — `{signal, confidence, mrr_cents, active_subs, churn_rate_30d, mrr_growth_rate, explanation, recommendation, estimated_impact}` or null
  - `price_signal` — string or null
- `trial_findings` — present when `lumiere_status` is `"trial_expired"`. Same shape as `lumiere`. Show these to the user as the trial gate recap, then ask them to upgrade.
- `applied_fixes` — last 3 fixes with impact data (cr_before, cr_after, delta) when available

When `lumiere_status` is `"trial_expired"`: present `trial_findings` as the recap of what the agents found, then tell the user their trial ended and they can upgrade at https://app.hivoltaire.com/account.
When `lumiere_status` is `"locked"`: omit the Lumière block entirely, tell the user to upgrade at https://app.hivoltaire.com/account to unlock the agents.

## Workflow

The only time you stop and wait is when you need something from the user (app name, Stripe key). Everything else runs in one session without interruption.

First, attempt to call `mcp__voltaire__get_stats`.

**CRITICAL — if the tool is not available or the call fails with any error:**
Stop immediately. Output EXACTLY this message, word for word, nothing else:

```
Run `/mcp` in Claude Code, find **voltaire** in the list, and click **Authenticate**. A browser will open — log in with Google or GitHub. Once done, run `/voltaire:voltaire` again.
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

   Run /voltaire:voltaire anytime — the more data flows in, the sharper the analysis gets.
   ```
   If trial expired: "Your 14-day Lumière Intelligence trial has ended. Upgrade at https://app.hivoltaire.com/account to keep running the agents."

---

### Recurring run (Stripe connected + SDK installed)

Don't repeat setup. Lead with data — let the user drive.

1. **Call `mcp__voltaire__analyze_paywall`**. If on Pro, also call `mcp__voltaire__get_recommendation`.

   **If 0 SDK events received** — ask the user: "No events have been received yet. Did you add `VOLTAIRE_SDK_TOKEN` to your production environment (Render, Vercel, Netlify, etc.)? The `.env` file is local only — the SDK won't initialize in production without the token in your hosting dashboard."

2. **Present a clear state of affairs — always, before doing anything:**
   ```
   Since last run [X days ago]:

   Conversion:  X%  (benchmark: B% · gap: +/-Xpp)
   Revenue:     $X this month  [↑↓ vs last month]
   Last fix:    [description] — [impact if measurable, or "still collecting data"]
   [Anomaly alert if flagged]
   ```

   Always surface benchmarks when present in the JSON (`benchmark` field). If there's a gap vs industry median, state it explicitly — this is the number the user should care about most.

   If trial expired: add "Trial ended — upgrade at https://app.hivoltaire.com/account to restore agents." and skip the Lumière block below.

3. **Lumière Intelligence — synthesis, not three silos.**

   After the metrics block, reason across all three agents. Your job is not to relay each agent's output independently — it's to find what they're collectively saying.

   **When agents converge** (two or more point to the same root cause), write a single synthesis paragraph, then list the individual agent signals below it:
   ```
   ✦ Lumière Intelligence

   [One-paragraph synthesis: connect what Timing, Cohort, and/or Churn are saying
   into a single coherent diagnosis. Example: "The data points to a timing problem
   made worse by a value-moment gap: paywall fires before users reach 'export', the
   one feature that predicts conversion (4.1% CR vs 1.6% without it). Churn confirms
   users are leaving before they get there — median 18 days, mostly before any export.
   Fix the gate trigger and you likely move all three metrics."]

     Timing:  [signal] [★★☆] — [one-line]
     Cohort:  [key differentiator] — [one-line]
     Churn:   MRR $X · churn X%/mo · [signal]
   ```

   **When agents diverge or data is thin**, be honest — don't force a narrative:
   ```
   ✦ Lumière Intelligence

   Timing flags [X] — Cohort and Churn don't have enough data to confirm or contradict.

     Timing:  [signal] [★★☆] — [one-line]
     Cohort:  insufficient data ([N] sessions)
     Churn:   insufficient data (no subscription history)
   ```

   If an agent has insufficient data, show it inline. Never invent a synthesis from one agent alone.

   **What to lead with:** You decide what matters most based on the data. The highest-signal insight goes first — that might be a funnel gap (most sessions never reaching the paywall), a churn rate, a benchmark gap, or an agent signal. Don't follow a fixed order. Read the numbers, find the biggest problem, say that first.

   **When all agents are ★☆☆ or insufficient_data** — do not propose a fix. There is nothing actionable yet. End the Lumière block with:
   ```
   Not enough data to recommend a code change yet.
   Come back when more events have been collected — Voltaire will notify you by email.
   ```
   Nothing else. No "you might consider", no "one option would be". Nothing to code means nothing to say about code.

4. **Stop. Wait for the user.**

   Do not propose a fix. Do not suggest what to do next. The user has just seen a diagnosis — let them decide:
   - They might want to ask questions about the data
   - They might want to understand a specific pattern
   - They might say "apply the recommendation" or "fix X"
   - They might just say "ok thanks"

   All of these are valid. Your job at this point is to answer questions, not to drive toward action.

5. **When the user asks for a fix:**

   **Describe first. Touch nothing.**

   Present exactly:
   - What file and line you'd change
   - What the change is
   - Why this addresses the root cause from the analysis

   Then stop and wait for explicit confirmation ("yes", "do it", "go ahead"). Do not start editing until you have that confirmation.

   Example:
   ```
   I'd update `src/Paywall.tsx` line 42 — move the paywall trigger from
   `onAppLoad` to after the first `export` action. That's where Timing and
   Cohort both point: users who export convert at 4.1% vs 1.6% without it.
   The change is ~3 lines. Want me to apply it?
   ```

6. **After confirmation, apply and log:**
   ```
   Fix applied: [one line]
   Check back in [timeframe] to measure impact.
   ```
   Call `mcp__voltaire__mark_applied`.

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
If you skip this and the project redefines its own `EventType`, you'll get TS2345 build errors because the project's type won't include `feature_gate_hit` and `upgrade_clicked`.

Find the right locations by reading the codebase — don't add them blindly. After installing, call `mcp__voltaire__mark_applied` with `type: "sdk"`.

**Data lives in MCP tools, not the dashboard.** `app.hivoltaire.com` exists for account and billing only. Never tell the user to "check the dashboard" for their conversion data, SDK token, or recommendations — everything data-related goes through `/voltaire:voltaire` in Claude Code. The SDK token can be found there if needed, but the MCP flow gives it automatically.

## Rules
- Never apply changes without explaining what you found first
- Always show paywall location, root cause, and proposed change before editing
- `analyze_paywall` returns raw data — you reason about it, don't expect a pre-computed diagnosis
- Always call `mark_applied` after every fix — this is what builds context over time
- Never mention a "dashboard" for data — use MCP tools
