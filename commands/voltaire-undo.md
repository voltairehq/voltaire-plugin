---
description: Review and revert the last Voltaire fix applied to the codebase
argument-hint: [optional: fix number to undo if multiple exist]
---

First, check if the Voltaire MCP tools are available by attempting to call `mcp__voltaire__get_stats`.

If the tool call fails, tell the user:
"Run `/mcp` in Claude Code, find Voltaire in the list, and click **Authenticate**. Once done, run `/voltaire-undo` again."
Then stop.

If the tools are available:

1. Call `mcp__voltaire__analyze_paywall` — look at `previously_applied_fixes` to find what was last applied.

2. Show the user what will be undone:
   ```
   Last fix applied: [date]
   Change: [description from mark_applied]
   File(s): [if known from the fix description]
   ```

3. Ask for confirmation before touching anything:
   "This will revert the last Voltaire fix. Confirm? (yes/no)"

4. **Wait for explicit confirmation.** If the user says no or is unsure, stop.

5. If confirmed:
   - Search the codebase for the changed code (use the fix description + date to locate it)
   - Revert the specific change — do not touch anything else
   - If the change cannot be automatically identified (e.g., it was a copy/messaging change with no clear marker), tell the user exactly what was changed and ask them to revert manually

6. After reverting, call `mcp__voltaire__mark_applied` with:
   - `type: "revert"`
   - `description: "Reverted: [original fix description]"`

7. Confirm:
   ```
   ↩ Reverted: [fix description]
   Run /voltaire when ready to apply a different fix.
   ```

**Important:** Only revert code changes Voltaire made. Never revert Stripe setup, SDK installation, or API key configuration — those are not reversible through this command and should not be undone without careful consideration.
