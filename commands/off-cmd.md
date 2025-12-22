---
name: off-cmd
description: Disable SDD tools to save context
---

# Disable SDD Tools

Switch to minimal mode (1 wake tool) to save context tokens when not using SDD features.

## Steps

1. Call `mcp__plugin_foundry_foundry-ctl__set_sdd_mode mode="minimal"`

2. Wait for the response indicating restart status.

3. Display the result:
   - If status is "restarting": "SDD tools disabled. **Restart Claude Code** (`/exit` then relaunch) to see reduced context. Use `/on-cmd` to re-enable."
   - If status is "unchanged": "SDD tools are already disabled."
