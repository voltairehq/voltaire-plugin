# Voltaire — Claude Code Plugin

Fix your paywall conversion rate with AI — directly inside Claude Code.

Voltaire connects your Stripe revenue data to your codebase, benchmarks your conversion rate against industry data, and applies targeted fixes through Claude Code.

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
| `/voltaire-status` | Quick pulse check — conversion rate, revenue, trend |
| `/voltaire-digest` | Weekly digest — what moved, what worked, what to watch |
| `/voltaire-undo` | Revert the last fix Voltaire applied |

Or say it naturally — "fix my paywall", "how is my conversion rate", "undo that last change" — and Claude picks it up automatically.

---

## What `/voltaire` does

1. Connects your Stripe account and imports revenue history
2. Benchmarks your conversion rate against your category (SaaS, mobile, etc.)
3. Explores your codebase to find the paywall
4. Proposes a fix — waits for your confirmation before touching anything
5. Installs the SDK to start collecting behavioral data
6. Logs what was changed so future sessions build on it

---

## Requirements

- [Claude Code](https://claude.ai/code)
- A Stripe account

Free plan: SDK install + data tracking + paywall analysis. Pro plan ($49/mo): weekly AI recommendations ranked by revenue impact + anomaly alerts + behavioral signals.

---

## Links

- [hivoltaire.com](https://hivoltaire.com)
- [Docs](https://hivoltaire.com/docs)
- hello@hivoltaire.com
