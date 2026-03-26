---
description: Analyze and optimize your paywall using Voltaire's revenue intelligence
argument-hint: [optional: specific issue to focus on]
---

First, check if the Voltaire MCP tools are available by attempting to call `mcp__voltaire__get_stats`.

If the tool call fails or the tools are not available, tell the user:
"Run `/mcp` in Claude Code, find Voltaire in the list, and click **Authenticate**. A browser will open to connect your account. Once done, run `/voltaire` again."
Then stop.

If the tools are available, run the full workflow:

1. Call `mcp__voltaire__get_stats`:
   - App not created → call `mcp__voltaire__create_app`. Stop.
   - Stripe not connected → ask for Stripe secret key, call `mcp__voltaire__setup`. Stop.
   - Otherwise → continue.

2. Call `mcp__voltaire__analyze_paywall` + explore the codebase to find the paywall. Stripe data alone is enough for a first fix — don't wait for SDK.

3. If SDK not installed → install the 7 events in the codebase (alongside the analysis, not instead of it). Tell the user: "Tracking events added — next run will have behavioral data."

4. Propose a concrete fix based on what you found. Wait for confirmation before editing.

5. If on Pro, call `mcp__voltaire__get_recommendation` and apply the prioritized fix.

6. After any change, call `mcp__voltaire__mark_applied` with a brief note.

7. End every first run with a summary:
   - What was done (Stripe connected, codebase analyzed, fix applied, SDK installed)
   - What happens automatically (daily digest email, hourly anomaly detection, Monday recommendations)
   - How to see data again: "Run /voltaire"
   - If Free plan: "🔒 Your weekly recommendation is locked. Upgrade to Pro ($49/mo) to unlock it + behavioral signals + anomaly alerts."

Always explain what you found before making changes. Wait for confirmation before touching the codebase.

**Important:** There is no Voltaire web dashboard. Never tell the user to "check the dashboard" — all data surfaces through these MCP tools. Everything goes through `/voltaire`.
