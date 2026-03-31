---
name: voltaire-undo
description: Revert the last fix Voltaire applied to the codebase. Triggered by "undo last fix", "revert voltaire change", "that fix didn't work".
argument-hint: [optional: fix number if multiple]
---

First, check if the Voltaire MCP tools are available by attempting to call `mcp__voltaire__get_stats`.

If the tool call fails, tell the user:
"Run `/mcp` in Claude Code, find **voltaire** in the list, and click **Authenticate**. A browser will open — log in with Google or GitHub. Once done, run `/voltaire-undo` again."
Then stop.

If the tools are available:

1. Call `mcp__voltaire__analyze_paywall` — read `previously_applied_fixes` to find what was last applied.

2. Show the user what will be undone:
   ```
   Last fix applied: [date]
   Change: [description from mark_applied]
   File(s): [if known]
   ```

3. Ask for confirmation: "This will revert the last Voltaire fix. Confirm? (yes/no)"

4. Wait for explicit confirmation. If no or unsure, stop.

5. If confirmed:
   - Locate the changed code using the fix description + date
   - Revert only that change — nothing else
   - If the change can't be automatically located (e.g. copy-only change), tell the user exactly what to revert manually

6. Call `mcp__voltaire__mark_applied` with:
   - `type: "revert"`
   - `description: "Reverted: [original fix description]"`

7. Confirm:
   ```
   ↩ Reverted: [fix description]
   Run /voltaire when ready to try a different fix.
   ```

**Important:** Only revert code changes Voltaire made. Never revert Stripe setup, SDK installation, or token config.
