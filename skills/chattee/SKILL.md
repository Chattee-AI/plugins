---
name: chattee
description: Builds, deploys and publishes web apps on Chattee via its MCP server. Use when the user names Chattee or chattee.ai, or wants a hosted app built and published for them, not code in their repo.
---

# Chattee

Chattee builds, deploys and hosts full web apps. Through its MCP server you have one built and published for the user, while the plan and the money stay their decisions.

This skill names the server's tools bare (`create_project`). Your client may prefix them: `mcp__chattee__create_project` after `claude mcp add`, `mcp__plugin_chattee_chattee__create_project` from the Claude Code plugin. Call the tool whose name ends in the one written here.

## Not connected yet?

If no Chattee tools are available, tell the user how to connect, and stop. Never imitate the tools or describe results you did not get.

- Claude Code: `claude mcp add --transport http chattee --scope user https://chattee.ai/mcp/connect`, then `claude mcp login chattee`. A Claude Code that answers `unknown command 'login'` is older than that command: sign in with `/mcp` inside Claude Code, or update it with `claude update`.
- Claude.ai and Claude Desktop: Customize → Connectors → Add custom connector, with `https://chattee.ai/mcp/connect` as the URL.
- Any other client: https://chattee.ai/mcp has the setup for each one.

## When to use Chattee

Use Chattee when the user:

- names Chattee or chattee.ai, or a project they built there;
- asks for a web app to be built, hosted and published for them (a site, a portal, a booking tool) rather than code changed in the repository you are working in.

Do not use it for code in the user's own repository: that is your work, not Chattee's. A build spends the user's credits and takes 5–90 minutes, so start one only when the user asked for a hosted app.

Chattee builds the whole app: a React front end, a Node back end where it needs one, an installable mobile app for phones when asked for one, a managed Postgres database, sign-in, file storage and email, deployed on chattee.ai with a Draft preview and, once published, a Live URL.

## Writing the brief

`create_project` takes the brief in the user's own words. Pass on what they said; never invent requirements. A good brief names:

- who uses the app, and what they do in it;
- the data it keeps, and who may see which part of it;
- sign-in and roles, if any;
- the emails it sends, and to whom;
- the look, if the user has a preference;
- what is out of scope ("no admin area").

Attach the user's files with `attachments` instead of describing them. When the brief leaves something open, Chattee's planner asks: relay its questions, never guess. One app is one project; later changes go into its chat with `send_message`, `mode: "plan"` for a feature and `mode: "edit"` for a small, direct change.

## The server's rules

Chattee MCP server is a remote Model Context Protocol server that lets AI assistants such as Claude Code, Claude.ai, Cursor, Codex and ChatGPT create, change, deploy and publish full web apps on Chattee.

Workflow:
1. `create_project` with the user's brief (or `send_message` into an existing chat, with `mode`).
2. Call `wait_for_run` repeatedly until it returns a decision point. Runs take 5–90 minutes; a timed-out wait (`outcome: timed_out_waiting`) or a dropped stream just means call `wait_for_run` again. Never stop calling it before a decision point, and never call `stop_run` because a wait timed out.
3. When the state is `awaiting_approval`, show the FULL plan and the builder link to the user. Call `approve_plan` only after they said yes, or `refine_plan` with their feedback. Never approve on their behalf.
4. Keep calling `wait_for_run`. When the state is `completed`, relay the closing summary and the Draft preview link.
5. `publish_project` only when the user asks; follow it with `get_release_status` (with `wait_seconds`) until `published`, and relay the Live URL.

Decisions that are the user's alone: answers to the agent's questions (`awaiting_input` — relay the question and its options verbatim, then `answer_question`), and money (`paused_budget`, `payment_required`, `insufficient_credits`). At a money outcome, relay the credits used and the builder link and stop: plans and credits are bought in the builder. Only when the user names a new total for a run at `paused_budget`, call `continue_run` with `run_budget_credits` set to exactly that total; set `persist_run_budget_default` only after a separate, explicit yes to keeping it as their default. Never choose or raise an amount yourself. A run paused at a limit ends by itself if nobody decides in time (`get_credits` says how long), and `stop_run` ends it at once; its work is kept either way, and `send_message` continues it as a new run with a fresh credit limit.

Modes: `mode: "plan"` for features and anything that may add modules or resources (the agent proposes, the user approves); `mode: "edit"` for a small direct change (no approval, credits spent at once).

Every result carries links (builder chat, project, Draft preview, Live URL). Always pass them on to the user.

A large chat that sat idle continues in a NEW chat instead of being resumed (its whole conversation would otherwise be written to the cache again): when a result's `chat_uid` differs from the one you sent to (`new_chat_reason` is set), use that chat for `wait_for_run` and every later message, and tell the user the conversation continues there.

Beyond the build loop: `list_templates` / `create_project(template_uid)` start from a template and `export_project_as_template` makes one; `list_files` / `read_file` look at the code and `write_file` / `delete_file` make a small, certain change (for anything larger, `send_message` with `mode: "edit"`, so the agent builds and verifies); `list_env_vars` / `set_env_vars` hold the app's own settings (secrets are never shown back; `env: "live"` is production and needs the `publish` scope); `list_resources` / `provision_resource` / `query_database` cover the managed database, storage, auth and email; `get_run_logs` explains a failed run; `get_credits` says what a build can spend; `delete_project` needs the project's exact name. A missing scope is reported as `insufficient_scope` with what to ask the user for; nothing was changed in that case.

Reference, limits and troubleshooting: https://chattee.ai/docs/mcp (as Markdown: https://chattee.ai/docs/mcp.md).

## What each outcome means

| State or outcome | What it means | What you do |
|---|---|---|
| `awaiting_approval` | The plan is ready. | Show the FULL plan and the builder link. `approve_plan` only after the user said yes; otherwise `refine_plan` with their feedback. |
| `awaiting_input` | The agent asked a question. | Relay it verbatim with its options, then `answer_question` with the user's answer. Never guess. |
| `paused_budget` | The run reached its budget. | Relay the credits used and the builder link, and stop. Only if the user names a new total, `continue_run` with `run_budget_credits` set to it; `persist_run_budget_default` only on a separate yes. |
| `insufficient_credits`, `payment_required` | The account needs credits or a paid plan. | Relay the link: the user buys in the builder. Nothing continues until then. |
| `paused_timeout`, `orphaned` | The run hit its time limit, or lost its process. | `continue_run` without a budget, within the existing allowance. |
| `timed_out_waiting` | Your wait ended before a decision point. | Call `wait_for_run` again; the run goes on. Never `stop_run` because a wait timed out. |
| `failed`, `deploy_failed` | The run or its deploy failed. | Read why (`get_run_logs`); `retry_run` once, or `approve_plan` again after a fixed deploy failure; otherwise tell the user. |
| `completed` | The run finished. | Relay the closing summary and the Draft preview link. `publish_project` only when the user asks. |

A tool called without its scope answers `insufficient_scope` and changes nothing: tell the user which scope to grant. The server takes 120 calls per minute per user, and `rate_limited` says how long to wait. A wait returns after at most 240 seconds.

## The 35 tools

Most tools that change data also take an optional `idempotency_key` (left out of the inputs below): a repeat with the same key returns the first result for 24 hours.

### Account and workspaces

| Tool | What it does | Scope | Inputs |
|---|---|---|---|
| `list_workspaces` | Lists the workspaces you belong to; the first by name is the default. | `projects:read` | – |
| `get_credits` | Shows the credit balance the next build draws on, and the credit limit per run. | `usage:read` | `project_uid` |

### Projects

| Tool | What it does | Scope | Inputs |
|---|---|---|---|
| `list_projects` | Lists a workspace's projects with deploy status and links; `query` searches by name. | `projects:read` | `page`, `page_size`, `query`, `workspace_uid` |
| `get_project` | Shows modules, Draft and Live status, unpublished changes, the chat holding a run, and links. | `projects:read` | `project_uid` |
| `create_project` | Creates a project from a prompt (planning starts at once) or from a template. | `projects:write` | `attachments`, `name`, `prompt`, `run_budget_credits`, `template_uid`, `workspace_uid` |
| `delete_project` | Deletes a project for good; needs its exact name as confirmation. | `projects:delete` | `confirm`, `project_uid` |

### Chats

| Tool | What it does | Scope | Inputs |
|---|---|---|---|
| `list_chats` | Lists a project's chats by recent activity. | `projects:read` | `project_uid` |
| `create_chat` | Opens a new chat or returns the empty one; never hands out the busy chat. | `projects:write` | `project_uid`, `title` |
| `get_chat` | Reads the transcript as typed rows (text, plan, question, closing summary) plus the run's state; pages back with `before`. | `projects:read` | `before`, `chat_uid`, `limit`, `response_format` |
| `delete_chat` | Deletes a chat; refused while it has a run to resolve. | `projects:delete` | `chat_uid` |

### Build runs

| Tool | What it does | Scope | Inputs |
|---|---|---|---|
| `send_message` | Starts a run (`mode: plan` or `edit`) or forwards feedback into the one running; a large chat that sat idle continues in a new one. | `projects:write` | `attachments`, `chat_uid`, `message`, `mode`, `run_budget_credits` |
| `wait_for_run` | Waits for a decision point (plan, question, pause, completion, failure) or the timeout, streaming progress. | `projects:read` | `chat_uid`, `response_format`, `timeout_seconds` |
| `approve_plan` | Relays your yes to the plan; deploys and starts the build. | `projects:write` | `build_budget_credits`, `chat_uid` |
| `refine_plan` | Relays your feedback on the plan; the agent revises it. | `projects:write` | `attachments`, `chat_uid`, `feedback` |
| `answer_question` | Relays your answer to a question the agent asked; never a guess. | `projects:write` | `answers`, `chat_uid`, `tool_use_id` |
| `stop_run` | Stops the run, or ends one paused at its credit limit. | `projects:write` | `chat_uid` |
| `continue_run` | Resumes a paused run; raises its budget only to a total you name. | `projects:write` | `chat_uid`, `persist_run_budget_default`, `run_budget_credits` |
| `retry_run` | Runs a stopped or failed run again on the same conversation. | `projects:write` | `chat_uid` |
| `get_run_logs` | Reads the run's tool calls, the deploy log or a container's log; says when an answer was cut. | `projects:read` | `chat_uid`, `module`, `source`, `tail` |

### Publishing

| Tool | What it does | Scope | Inputs |
|---|---|---|---|
| `get_release_status` | Draft versus Live, the release serving Live, history and blockers; waits for a release with `wait_seconds`. | `projects:read` | `project_uid`, `response_format`, `wait_seconds` |
| `publish_project` | Publishes the Draft to Live, or restores an earlier release. | `publish` | `change_summary`, `project_uid`, `restore_release_uid` |

### Templates

| Tool | What it does | Scope | Inputs |
|---|---|---|---|
| `list_templates` | Lists the templates you can start from: yours, published ones, or both. | `projects:read` | `scope` |
| `get_template` | Shows a template's description, tags, modules and preview. | `projects:read` | `template_uid` |
| `export_project_as_template` | Freezes a deployed project as a private template with its own empty database. | `templates` | `description`, `name`, `project_uid`, `tags` |
| `update_template` | Renames, describes, tags, publishes or unpublishes a template. | `templates` | `description`, `name`, `published`, `tags`, `template_uid` |
| `delete_template` | Deletes a template for good; unpublish it first. | `templates` | `template_uid` |

### Files

| Tool | What it does | Scope | Inputs |
|---|---|---|---|
| `list_files` | Shows a file tree, or searches names and contents. | `projects:read` | `depth`, `path`, `project_uid`, `query`, `response_format` |
| `read_file` | Reads a window of a text file, at most 200,000 characters per call. | `projects:read` | `limit_lines`, `offset_line`, `path`, `project_uid` |
| `write_file` | Writes a whole Draft file, creating it if missing; counts as an unpublished change. | `files:write` | `content`, `path`, `project_uid` |
| `delete_file` | Deletes one Draft file. | `files:write` | `path`, `project_uid` |

### Configuration and data

| Tool | What it does | Scope | Inputs |
|---|---|---|---|
| `list_env_vars` | Lists per-module variables; a secret's value comes back as `null`. | `config` | `env`, `project_uid` |
| `set_env_vars` | Merges variables; removes one only with `remove: true`; `apply` recreates the containers. | `config` | `apply`, `env`, `project_uid`, `vars` |
| `list_resources` | Lists the managed database, file storage, auth and email, with their health. | `config` | `env`, `project_uid` |
| `provision_resource` | Adds a managed resource outside a plan. | `config` | `project_uid`, `reason`, `resource_id` |
| `query_database` | Shows the schema or runs one read-only statement against Draft or Live (`env` is required; Live reads are logged). | `config` | `env`, `max_rows`, `project_uid`, `sql` |

## Reference

- The docs: https://chattee.ai/docs/mcp (as Markdown: https://chattee.ai/docs/mcp.md)
- Every tool as JSON: https://chattee.ai/docs/mcp/tools.json
- Tokens and connected apps: https://chattee.ai/app/ai-assistants
