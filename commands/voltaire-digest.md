---
description: Weekly digest — what moved, what worked, what to watch
argument-hint: [optional: time range, e.g. "last 2 weeks"]
---

First, check if the Voltaire MCP tools are available by attempting to call `mcp__voltaire__get_stats`.

If the tool call fails, tell the user:
"Run `/mcp` in Claude Code, find Voltaire in the list, and click **Authenticate**. Once done, run `/voltaire-digest` again."
Then stop.

If the tools are available:

1. Call `mcp__voltaire__analyze_paywall` — gather the full data dump.

2. If on Pro, also call `mcp__voltaire__get_recommendation` — include this week's prioritized recommendation in the digest.

3. Produce a structured weekly digest:

   ```
   📬 Voltaire Weekly Digest — week of [date]

   ── Conversion ──────────────────────────────
   This week:  X.X%  (was Y.Y% last week  [Δ +/- Z%])
   Benchmark:  B.B%  ([above/below] by X.X points)
   Trend:      [7-day sparkline direction: ↑ / → / ↓]

   ── Revenue ─────────────────────────────────
   This week:  $X  (was $Y last week  [Δ +/- $Z])
   MRR:        $X  (if available)

   ── What happened ───────────────────────────
   [List fixes applied this week from mark_applied history]
   [Any anomalies detected and their impact]

   ── Behavioral signals (Pro) ─────────────────
   Top feature gate hit:  [feature name]  (X times)
   Most dismissed paywall:  [context if known]
   Upgrade click rate:  X%

   ── This week's recommendation ──────────────
   [From get_recommendation — or "No data yet" if SDK < 7 days]

   ── To act on this ──────────────────────────
   Run /voltaire to apply the next fix.
   ```

4. If SDK not installed: skip behavioral signals section, note "Install SDK via `/voltaire` to unlock behavioral data."

5. If Free plan: replace recommendation section with "🔒 Weekly recommendations require Pro ($49/mo)."

6. Keep the digest factual and scannable — no marketing language, no padding.
