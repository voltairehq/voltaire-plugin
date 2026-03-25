# Voltaire — Claude Code Plugin

Fix your paywall conversion rate with AI — directly inside Claude Code.

Voltaire connects your Stripe revenue data to your codebase, benchmarks your conversion rate against industry data, and applies targeted fixes through Claude Code.

---

## Install

```bash
claude mcp add voltaire --transport http https://mcp.hivoltaire.com/mcp
```

On first use, a browser window opens to connect your Voltaire account. No API key to copy.

---

## Usage

Once installed, just run:

```
/voltaire
```

Or say it naturally — "fix my paywall", "optimize my conversion rate" — and Claude picks it up automatically.

---

## What happens

1. Voltaire reads your revenue metrics and behavioral data from Stripe
2. Benchmarks your conversion rate against your category (SaaS, mobile, etc.)
3. Analyzes your codebase to find where the paywall is and what's wrong with it
4. Proposes a fix — and waits for your confirmation before touching anything
5. Applies the fix: SDK install, paywall repositioning, copy, timing
6. Logs what was changed so future sessions don't repeat the same work

---

## Requirements

- [Claude Code](https://claude.ai/code)
- A Stripe account

Free plan: SDK install + data tracking + paywall analysis. Pro plan ($49/mo): weekly AI recommendations ranked by revenue impact.

---

## Links

- [hivoltaire.com](https://hivoltaire.com)
- [Docs](https://hivoltaire.com/docs)
- hello@hivoltaire.com
