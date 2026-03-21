---
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
   - App not created → call `mcp__voltaire__create_app` (ask for app name and category)
   - Stripe not connected → call `mcp__voltaire__setup` (ask the user for their Stripe secret key — find it at dashboard.stripe.com/apikeys)
   - SDK not installed → install tracking events (see below)
   - Data available → proceed to full analysis

2. **Analyze** — call `mcp__voltaire__analyze_paywall` to get the full data dump. This is raw data — you do the reasoning. Explore the codebase to find the paywall and understand the root cause.

3. **Fix** — propose a concrete change. Show the user what you found and what you want to change before touching anything.

4. **Pro recommendation** — if the user is on Pro, call `mcp__voltaire__get_recommendation` for the week's prioritized fix and apply it.

5. **Log the fix** — after any change is applied, call `mcp__voltaire__mark_applied` with a brief note. This is what makes future runs smarter.

## SDK events to install
When SDK is not present, add these events to the app:
- `session_started` — on app open
- `feature_used` — on key feature interaction (pass `feature_name`)
- `feature_gate_hit` — when a user hits a premium gate and is blocked (strongest upgrade intent signal)
- `upgrade_clicked` — when a user taps an upgrade/pricing CTA but doesn't complete payment
- `paywall_shown` — when paywall is displayed
- `paywall_dismissed` — when user closes without converting
- `paywall_converted` — on successful subscription

## Rules
- Never apply changes without explaining what you found first
- Always show paywall location, root cause, and proposed change before editing
- `analyze_paywall` returns raw data — you reason about it, don't expect a pre-computed diagnosis
- Always call `mark_applied` after every fix — this is what builds context over time
