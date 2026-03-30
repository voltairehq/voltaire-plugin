---
name: voltaire-digest
description: Weekly digest of paywall performance — what moved, what worked, what to watch. Triggered by "voltaire digest", "weekly report", "what happened this week".
argument-hint: [optional: time range, e.g. "last 2 weeks"]
---

First, check if the Voltaire MCP tools are available by attempting to call `mcp__voltaire__get_stats`.

If the tool call fails, tell the user:
"Run `/mcp` in Claude Code, find **voltaire** in the list, and click **Authenticate**. A browser will open — log in with Google or GitHub. Once done, run `/voltaire-digest` again."
Then stop.

If the tools are available:

1. Call `mcp__voltaire__analyze_paywall` — gather the full data dump.

2. If on Pro, call `mcp__voltaire__get_recommendation` — include this week's prioritized fix.

3. Produce a structured weekly digest:
   ```
   📬 Voltaire Weekly Digest — week of [date]

   ── Conversion ──────────────────────────────
   This week:  X.X%  (was Y.Y% last week  [Δ +/- Z%])
   Benchmark:  B.B%  ([above/below] by X.X points)
   Trend:      ↑ / → / ↓

   ── Revenue ─────────────────────────────────
   This week:  $X  (was $Y last week  [Δ +/- $Z])
   MRR:        $X  (if available)

   ── What happened ───────────────────────────
   [Fixes applied this week, from mark_applied history]
   [Anomalies detected and their impact]

   ── Behavioral signals (Pro) ─────────────────
   Top feature gate hit:  [feature name]  (X times)
   Most dismissed paywall:  [context if known]
   Upgrade click rate:  X%

   ── This week's recommendation ───────────────
   [From get_recommendation — or "No data yet" if SDK < 7 days]

   ── To act on this ───────────────────────────
   Run /voltaire to apply the next fix.
   ```

4. If SDK not installed: skip behavioral signals, note "Install SDK via `/voltaire` to unlock behavioral data."

5. If Free plan: replace recommendation section with "🔒 Weekly recommendations require Pro ($49/mo)."

Keep it factual and scannable — no padding, no marketing language.
