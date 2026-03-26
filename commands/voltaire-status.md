---
description: Quick pulse check on your paywall — conversion rate, revenue, and any anomalies since last run
argument-hint: [optional: specific metric to focus on]
---

First, check if the Voltaire MCP tools are available by attempting to call `mcp__voltaire__get_stats`.

If the tool call fails, tell the user:
"Run `/mcp` in Claude Code, find Voltaire in the list, and click **Authenticate**. Once done, run `/voltaire-status` again."
Then stop.

If the tools are available:

1. Call `mcp__voltaire__get_stats` — check connection status, SDK status, data volume.

2. Call `mcp__voltaire__analyze_paywall` — extract:
   - Current conversion rate vs benchmark
   - Revenue this week vs last week
   - Any anomalies flagged
   - Last fix applied and its impact (if measurable)
   - 7-day trend direction (up/flat/down)

3. Output a compact status card — no prose, just facts:
   ```
   📊 Voltaire Status — [date]

   Conversion:  X.X% (benchmark: Y.Y%)  [↑↓ vs last week]
   Revenue:     $X this week  [↑↓ $Y vs last week]
   Data points: X events collected
   Last fix:    [one line, date]
   Trend:       ↑ improving / → flat / ↓ declining

   [Any anomaly alerts if present]
   ```

4. If data is thin (SDK not installed or < 7 days of data): say so clearly. Suggest running `/voltaire` to complete setup.

5. If on Free plan and an anomaly was detected: mention "🔒 Anomaly alerts and behavioral signals require Pro ($49/mo)."

No fixes, no recommendations — this is read-only. For a full analysis and fix, run `/voltaire`.
