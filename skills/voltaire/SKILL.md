---
name: voltaire
description: Use when the user wants to optimize their paywall or improve conversion rates. Triggered by: "fix my paywall", "optimize my paywall", "/voltaire", "improve conversions".
---

# Voltaire — Paywall Optimization

## Overview
Voltaire is a revenue intelligence layer. It tracks paywall events, computes conversion metrics, benchmarks them against industry data, and exposes everything via MCP. Your job is to pull that data, explore the codebase, and apply a concrete fix.

## Available MCP tools
- `mcp__voltaire__create_app` — first-run bootstrap: create the app (name, category). Returns the API key.
- `mcp__voltaire__setup` — connect Stripe: takes `stripe_secret_key`, optionally `github_repo_url`
- `mcp__voltaire__get_stats` — current state: revenue connection, SDK status, conversion rate, data volume
- `mcp__voltaire__analyze_paywall` — full data dump: conversion vs benchmark, revenue, behavioral metrics, 7-day trend, previously applied fixes
- `mcp__voltaire__get_recommendation` — latest weekly recommendation (Pro users only)
- `mcp__voltaire__mark_applied` — log what was changed so future runs have full context

## Workflow

1. **Get current state** — call `mcp__voltaire__get_stats`:
   - App not created → call `mcp__voltaire__create_app` (ask for app name and category). Stop.
   - Stripe not connected → call `mcp__voltaire__setup` (ask for Stripe secret key — find it at dashboard.stripe.com/apikeys). Stop.
   - Otherwise → continue below.

2. **Analyze** — call `mcp__voltaire__analyze_paywall` + explore the codebase to find the paywall. Stripe data alone is enough for a first diagnosis — don't wait for SDK data.

3. **Install SDK if missing** — if `get_stats` showed SDK not installed, install the 7 events now (see below). Do this alongside the analysis, not instead of it. Tell the user: "I've also added tracking events — next time you run `/voltaire` there will be behavioral data to work with."

4. **Fix** — propose a concrete change based on what you found. Show the user the paywall location, root cause, and proposed fix before touching anything.

5. **Pro recommendation** — if on Pro, call `mcp__voltaire__get_recommendation` for this week's prioritized fix.

6. **Log the fix** — call `mcp__voltaire__mark_applied` after every applied change. This is what makes future runs smarter.

## SDK events to install
When SDK is not present, add these events to the app:
- `session_started` — on app open
- `feature_used` — on key feature interaction (pass `feature_name`)
- `feature_gate_hit` — when a user hits a premium gate and is blocked (strongest upgrade intent signal)
- `upgrade_clicked` — when a user taps an upgrade/pricing CTA but doesn't complete payment
- `paywall_shown` — when paywall is displayed
- `paywall_dismissed` — when user closes without converting
- `paywall_converted` — on successful subscription

After installing the SDK, tell the user: "Events are now tracking. Run `/voltaire` again once you have real traffic and the data will show up here."

**There is no web dashboard.** All data is surfaced through the MCP tools above. Never tell the user to "check the dashboard" or "visit the Voltaire app" — everything goes through `/voltaire` in Claude Code.

## Rules
- Never apply changes without explaining what you found first
- Always show paywall location, root cause, and proposed change before editing
- `analyze_paywall` returns raw data — you reason about it, don't expect a pre-computed diagnosis
- Always call `mark_applied` after every fix — this is what builds context over time
- Never mention a "dashboard" — Voltaire has no web UI, all output is through MCP tools
