---
description: Analyze and fix your paywall using Voltaire's revenue intelligence
argument-hint: [optional: specific issue to focus on]
---

Run the Voltaire paywall optimization workflow:

1. Call `mcp__voltaire__get_stats` to check the current state
2. Use the `fix_paywall` MCP prompt from the voltaire server to get the full action plan
3. Execute the plan step by step, starting with SDK installation if needed
4. Apply the paywall fix identified by Voltaire's analysis
5. Call `mcp__voltaire__mark_applied` after completing each step

Always explain what you're doing before making changes to the codebase.
