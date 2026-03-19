---
description: Analyze and optimize your paywall using Voltaire's revenue intelligence
argument-hint: [optional: specific issue to focus on]
---

Run the Voltaire paywall optimization workflow:

1. Call `mcp__voltaire__get_stats` to check the current state:
   - No revenue data connected → guide the user to connect Stripe or RevenueCat at hivoltaire.com
   - SDK not installed → install the 5 tracking events in the codebase
   - Data available → proceed to full analysis and fix

2. If data is available, call `mcp__voltaire__analyze_paywall` to get the full data dump (conversion rate vs benchmark, revenue, behavioral metrics, 7-day trend, previously applied fixes). Explore the codebase to find the paywall, understand the root cause, and propose a concrete fix.

3. If the user is on Pro and a weekly recommendation exists, call `mcp__voltaire__get_recommendation` and apply it as the prioritized fix.

4. After applying any change, call `mcp__voltaire__mark_applied` with a brief note of what was changed so future runs have full context.

Always explain what you found before making any changes. Wait for user confirmation before touching the codebase.
