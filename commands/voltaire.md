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
   - App not created → call `mcp__voltaire__create_app` (ask for app name and category)
   - Stripe not connected → call `mcp__voltaire__setup` with Stripe secret key
   - SDK not installed → install the 7 tracking events in the codebase
   - Data available → proceed to full analysis

2. Call `mcp__voltaire__analyze_paywall` for the full data dump. Explore the codebase to find the paywall, understand the root cause, propose a concrete fix.

3. If on Pro, call `mcp__voltaire__get_recommendation` and apply the prioritized fix.

4. After any change, call `mcp__voltaire__mark_applied` with a brief note.

Always explain what you found before making changes. Wait for confirmation before touching the codebase.
