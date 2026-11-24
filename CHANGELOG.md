# Changelog

Every published version of the Chattee plugin, newest first. A heading is the version every plugin manifest states,
and a published version never changes.

## 1.1.0

- The plugin is the repository's root, so one repository serves every client that reads plugins from there. Beside
  the Claude Code and Cursor manifests it carries an Agent Plugins 1.0 manifest (VS Code, GitHub Copilot, Codex,
  ChatGPT, Kiro), a Gemini CLI extension, an Antigravity manifest and a Codex marketplace file.
- The Claude Code manifest names the icon, documentation, support, privacy and terms for Anthropic's directory.
- The README says what the plugin sends and to whom, what an account needs, how to install it in VS Code and GitHub
  Copilot, and how to install the skill alone (`npx skills add https://chattee.ai`).
- `SECURITY.md` names where a vulnerability is reported.
- The skill and the three commands are unchanged.

## 1.0.3

- The skill says that Chattee also builds an installable mobile app for phones when asked for one (the `mobile-pwa`
  module, served at `/m/` beside the website and the web app), and the server's `list_files`, `get_run_logs` and
  environment-variable tools name that module.

## 1.0.2

- The skill's Claude Code connect line says what to do when the CLI answers `unknown command 'login'`. That CLI is
  older than `claude mcp login` (Claude Code 2.1.186): sign in with `/mcp` inside Claude Code, or update it with
  `claude update`.

## 1.0.1

- The skill follows the server's credit limit per run: a run paused at its limit ends by itself if nobody decides in
  time (`get_credits` says how long), `stop_run` ends it at once, and the next message continues the work as a new
  run with a fresh credit limit.

## 1.0.0

- First release, for Claude Code and Cursor: the Chattee MCP server (`https://chattee.ai/mcp/connect`, sign-in with
  OAuth), the Chattee skill, and three commands — build an app, see where a project stands, publish it.
