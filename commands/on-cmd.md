---
name: on-cmd
description: Enable full SDD tools
---

# Enable SDD Tools

Enable the full SDD toolkit (17 tools) by switching from minimal mode.

## Steps

1. Call `mcp__plugin_foundry_foundry-ctl__set_sdd_mode mode="full"`

2. Wait for the response indicating restart status.

3. Display the result:
   - If status is "restarting": "SDD tools enabled. Server restarting (~1-2s). All 17 routers will be available."
   - If status is "unchanged": "SDD tools are already enabled."

4. After a brief pause, verify by calling `mcp__plugin_foundry_foundry-mcp__health action="liveness"` to confirm the server is responsive.
