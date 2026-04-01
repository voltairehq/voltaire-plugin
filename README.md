# Voltaire: Claude Code Plugin

Revenue intelligence for your paywall. Tracks user behavior, analyzes conversion data, and surfaces targeted fixes directly in Claude Code.

---

## Install

```bash
curl -fsSL https://mcp.hivoltaire.com/install | bash
```

A browser window opens to connect your Voltaire account. No API key to copy.

---

## Commands

| Command | What it does |
|---------|-------------|
| `/voltaire` | Full session: connect Stripe, analyze codebase, apply a fix |
| `/voltaire-status` | Quick pulse check: conversion rate, revenue, trend |

Or say it naturally: "fix my paywall", "how is my conversion rate", "what happened this week", and Claude picks it up automatically.

---

## What `/voltaire` does

**First run:**
1. Connects your Stripe account and imports revenue history
2. Benchmarks your conversion rate against your SaaS category
3. Explores your codebase to find the paywall
4. Installs the SDK and wires 7 behavioral tracking events
5. Proposes a targeted fix, waits for your confirmation before touching anything
6. Logs what was changed so future sessions build on it

**Every run after (Lumière Intelligence):**

Three agents run in parallel on every `/voltaire`:

- **Timing:** when your paywall fires vs the optimal moment, with confidence rating based on data volume
- **Cohort:** what separates converters from bouncers, the one feature that predicts conversion
- **Churn:** MRR health, monthly churn rate, median days before cancel, top cancellation reasons from Stripe

**In the background:**
- Hourly anomaly detection: email alert if conversion drops significantly
- Daily digest with conversion delta, anomalies, and session patterns
- Weekly recommendation ranked by revenue impact, every Monday

---

## Plans

**14-day trial:** Full Lumière Intelligence access from the first `/voltaire` — all three agents, daily digest, anomaly alerts, weekly recommendations.

**Lumière Intelligence ($49/mo):** Everything in the trial, continued. Upgrade at [app.hivoltaire.com/account](https://app.hivoltaire.com/account).

---

## Requirements

- [Claude Code](https://claude.ai/code)
- A Stripe account

---

## Links

- [hivoltaire.com](https://hivoltaire.com)
- [Docs](https://hivoltaire.com/docs)
- hello@hivoltaire.com
