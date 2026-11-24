---
description: Publish a Chattee project's Draft to Live, after you confirm what goes live.
argument-hint: "[project]"
---

Publish the user's Chattee project to Live with the Chattee MCP tools, following the Chattee skill and the server's own rules.

The user's request: $ARGUMENTS

## Steps

1. Find the project the user means. Call `list_projects` with a `query` when you do not know it.
2. Call `get_release_status`: it says what would go Live and whether anything blocks it. Relay a blocker and stop.
3. Tell the user what will go Live, and ask them to confirm. Call `publish_project` only after an explicit yes.
4. Then call `get_release_status` with `wait_seconds` until the release is `published`, or has failed.
5. Relay the Live URL, or the error and the builder link.
