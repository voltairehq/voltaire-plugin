# Voltaire — Claude Code Plugin

Voltaire is an AI-powered paywall optimization tool that connects your revenue data to your codebase. It analyzes conversion rates, benchmarks them against industry standards, identifies what is killing your paywall performance, and helps you apply targeted fixes — directly inside Claude Code.

---

## What Voltaire Does

Most SaaS and mobile apps leave significant revenue on the table because their paywall is shown at the wrong moment, with the wrong message, or with the wrong structure. Voltaire fixes that by:

- **Tracking the full conversion funnel** — from session start to paywall shown, dismissed, and converted
- **Benchmarking your metrics** against real industry data for your category and pricing model
- **Diagnosing the root cause** of low conversion — timing, friction, copy, pricing structure
- **Delivering a weekly recommendation** (Pro) — one concrete, high-impact fix per week, ready to apply
- **Applying fixes in your codebase** — via Claude Code, guided by the `fix-paywall` skill and command

---

## Installation

Install the Voltaire plugin for Claude Code by running:

```
/plugin install voltaire@voltaire-app
```

This will:
- Register the Voltaire MCP server (`https://voltaire-back.onrender.com/mcp`) in your Claude Code environment
- Make the `fix-paywall` skill available for natural-language invocation
- Add the `/fix-paywall` slash command to Claude Code

---

## The `fix-paywall` Skill

The skill is triggered automatically when you ask Claude Code things like:

- "fix my paywall"
- "optimize my paywall"
- "improve my conversion rate"
- "apply Voltaire fix"

Claude will invoke the Voltaire workflow:

1. Check your current Voltaire state (no data yet, first digest available, or weekly recommendation ready)
2. Pull the full diagnostic from `mcp__voltaire__analyze_paywall`
3. Present a clear action plan before touching anything
4. Apply the fix — SDK installation, paywall repositioning, copy change, or structural update
5. Log the applied change via `mcp__voltaire__mark_applied`

---

## The `/fix-paywall` Command

You can also invoke the workflow directly as a slash command:

```
/fix-paywall
```

Or with a specific focus area:

```
/fix-paywall the trial upsell screen is converting below 2%
```

The command runs the same workflow as the skill: check state, get the plan, execute step by step, confirm before changes.

---

## Requirements

- **Claude Code** — [claude.ai/code](https://claude.ai/code)
- **Voltaire account** — sign up at [voltaire.app](https://voltaire.app)

A free Voltaire account is enough to get started. The SDK installation and first paywall diagnostic are available on the free plan. Weekly AI recommendations require a Pro subscription.

---

## How It Works End-to-End

```
Your app  →  Voltaire SDK  →  Voltaire backend  →  Claude Code plugin
   |              |                  |                      |
tracks         sends 5           analyzes &            applies the
5 events       events            benchmarks             fix in your
               (session,         your funnel            codebase
               feature,
               paywall shown/
               dismissed/
               converted)
```

The 5 SDK events Voltaire tracks:

| Event | When to fire |
|---|---|
| `session_started` | App/page load |
| `feature_used` | User engages with a core feature |
| `paywall_shown` | Paywall or upgrade prompt is displayed |
| `paywall_dismissed` | User closes the paywall without converting |
| `paywall_converted` | User starts a trial or completes a purchase |

---

## Links

- Website: [voltaire.app](https://voltaire.app)
- Plugin repository: [github.com/voltaire-app/voltaire-plugin](https://github.com/voltaire-app/voltaire-plugin)
- Support: hello@voltaire.app
