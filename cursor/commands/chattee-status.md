---
name: chattee-status
description: Show where a Chattee project stands — its run, its Draft and its Live release — with the links.
---

Report where the user's Chattee project stands, with the Chattee MCP tools. This command only reports: it starts, stops and continues nothing.

The user's request is the text they typed with this command.

## Steps

1. Find the project: the one the user names, else the one this conversation worked on. Call `list_projects` with a `query` when you do not know it.
2. Call `get_project`: it shows the Draft and Live status and the chat that holds a run, if any.
3. If a chat holds a run, call `wait_for_run` on it with `timeout_seconds: 0` to read its state without waiting.
4. If the user asks about Live, call `get_release_status`: it shows what is published and what is waiting.
5. Relay the state in plain words, with every link you received: the builder chat, the Draft preview, the Live URL.
