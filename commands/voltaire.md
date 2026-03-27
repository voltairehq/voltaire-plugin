---
description: Analyze and optimize your paywall using Voltaire's revenue intelligence
argument-hint: [optional: specific issue to focus on]
---

First, check if the Voltaire MCP tools are available by attempting to call `mcp__voltaire__get_stats`.

If the tool call fails or the tools are not available, tell the user:
"Run `/mcp` in Claude Code, find Voltaire in the list, and click **Authenticate**. A browser will open to connect your account. Once done, run `/voltaire` again."
Then stop.

If the tools are available, call `mcp__voltaire__get_stats` and determine which mode you're in:

---

## First run (Stripe not connected OR SDK not installed)

The only stops are when you need input from the user (app name, Stripe key). Everything else runs in one session without interruption.

1. **App not created** → call `mcp__voltaire__create_app`. Ask for name + category if needed, then immediately continue.

2. **Stripe not connected** → look for `STRIPE_SECRET_KEY` in backend `.env` files only (`backend/.env`, `server/.env`, `api/.env`, etc. — never in the frontend). If found, call `mcp__voltaire__setup` without displaying the key. If not found, ask the user to paste it. Then keep going immediately.

3. Call `mcp__voltaire__analyze_paywall` + explore the codebase to find the paywall. Stripe data alone is enough for a first fix — don't wait for SDK.

4. If SDK not installed → install it now in this same session: `npm install voltaire-sdk`, write `VOLTAIRE_API_KEY` to `.env` (don't ask the user — do it yourself), create the init file, add the 7 tracking calls in the right places.

5. Propose a concrete fix based on what you found. Wait for confirmation before editing.

6. After any change, call `mcp__voltaire__mark_applied` (once for SDK install, once for the fix).

7. End with a summary:
   ```
   Here's what Voltaire set up:
   ✅ Stripe connected — revenue history imported
   ✅ SDK installed — 7 tracking events now collecting data
   ✅ First fix applied — [one line]

   What happens automatically:
   - Every day: digest email with conversion delta
   - Every hour: anomaly detection
   - Every Monday: new recommendation from latest data

   Run /voltaire again in a week — behavioral data will be flowing and the analysis will be much sharper.
   ```
   If Free plan: "🔒 Weekly recommendation locked. Upgrade to Pro ($49/mo) at https://app.hivoltaire.com/account"

---

## Recurring run (Stripe connected + SDK installed)

Don't repeat setup. Focus entirely on what's new and what to do next.

1. Call `mcp__voltaire__analyze_paywall` — look at the data with fresh eyes:
   - What moved since last time? (conversion rate delta, revenue delta)
   - Any anomalies flagged?
   - Impact of the last fix if data is available
   - Which behavioral signals stand out (Pro)?

2. If on Pro → call `mcp__voltaire__get_recommendation` for this week's prioritized fix.

3. Propose the next fix based on what the data shows. Reference what changed since last run. Wait for confirmation before editing. Apply, then call `mcp__voltaire__mark_applied`.

4. End with a short summary:
   ```
   Since last time: [conversion X% → Y%, revenue delta, anomaly if any]
   Fix applied: [one line]
   Next run: check back after [timeframe] to measure impact.
   ```

---

Always explain what you found before making changes. Wait for confirmation before touching the codebase.

**Important:** There is no Voltaire web dashboard. Never tell the user to "check the dashboard" — all data surfaces through these MCP tools. Everything goes through `/voltaire`.
