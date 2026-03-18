---
name: voltaire
description: |
  Use this skill when the user wants to optimize their paywall, improve conversion rates, or apply Voltaire's revenue analysis to their codebase. Triggered by: "fix my paywall", "optimize my paywall", "/voltaire", "apply Voltaire fix", "improve conversions".
metadata:
  mcp-server: voltaire
---

# Fix Paywall — Voltaire Optimization Agent

## Overview
Voltaire analyzes your revenue data against industry benchmarks and guides you to fix your paywall directly in your codebase.

## Workflow

1. **Get current state** — call `mcp__voltaire__get_stats` to understand where the user is in their Voltaire journey (J0=no data, J1=first digest, J7=weekly reco available)

2. **Run the fix prompt** — invoke the `fix_paywall` MCP prompt which returns a state-aware action plan

3. **Execute the plan**:
   - If SDK not installed: install Voltaire SDK tracking (5 events: session_started, feature_used, paywall_shown, paywall_dismissed, paywall_converted)
   - If diagnostic available: apply the specific paywall fix identified by Voltaire
   - If weekly reco available (Pro): apply this week's recommendation

4. **Report back** — call `mcp__voltaire__mark_applied` after each step to log what was done

## Important
- Never apply changes without discussing the plan with the user first
- Always show what you found (paywall location, key moment, current issue) before making changes
- Use `mcp__voltaire__analyze_paywall` to get the full diagnostic context
- Use `mcp__voltaire__get_recommendation` for the latest weekly recommendation (Pro users)
