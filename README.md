# Chattee

[Chattee](https://chattee.ai) is an AI app builder that builds, deploys and hosts full web apps. This plugin connects your AI assistant to the Chattee MCP server and teaches it how to work with Chattee. You describe an app; your assistant hands the brief to Chattee, shows you the whole plan, and the app is built once you say yes. A small change you ask for can go straight in, and nothing goes Live until you ask.

One install brings:

- **The Chattee MCP server** (`https://chattee.ai/mcp/connect`). You sign in with your Chattee account through OAuth; no token sits in any file.
- **The Chattee skill**: the build loop, the approval rule, what each outcome means, and every tool.
- **Three commands**, in Claude Code and Cursor: build an app, check where a project stands, publish it.

| | Claude Code | Cursor |
|---|---|---|
| Build | `/chattee:build <what to build>` | `/chattee-build` |
| Status | `/chattee:status` | `/chattee-status` |
| Publish | `/chattee:publish` | `/chattee-publish` |

You need a Chattee account ([create one](https://chattee.ai/get-started?source=plugin)). A new account gets a 7-day trial with 30 credits for planning; building an app needs a paid plan and spends its credits, as building in the browser does.

Limits: Chattee builds and hosts its own projects and does not edit the repository you are working in. A build takes 5–90 minutes.

## Claude Code

Inside Claude Code:

```text
/plugin marketplace add chattee-ai/plugins
/plugin install chattee@chattee
```

Then open `/mcp`, choose **chattee** and sign in: Chattee's consent page opens in your browser.

Install the plugin **instead of** `claude mcp add`, not alongside it: with both, every Chattee tool shows up twice. If you added Chattee by hand before, remove that entry first. The plugin's tools are named `mcp__plugin_chattee_chattee__<tool>`.

Signing in with a personal access token instead of OAuth? Then keep the manual setup with a header, as the [docs](https://chattee.ai/docs/mcp#how-do-i-add-the-chattee-mcp-server-to-claude-code) show; the plugin signs in with OAuth only.

## Cursor

Install **Chattee** from the Cursor Marketplace once it is listed there. Until then, copy this repository to `~/.cursor/plugins/local/chattee` and reload the window. Cursor shows the plugin's commands in `/` when **Include third-party Plugins, Skills, and other configs** is on in its settings.

Cursor finds the server, sees that it offers OAuth and asks you to sign in; approve on Chattee's consent page.

## VS Code and GitHub Copilot

In Copilot CLI:

```text
copilot plugin marketplace add chattee-ai/plugins
copilot plugin install chattee@chattee
```

In VS Code, with `chat.plugins.enabled` on, run **Chat: Install Plugin from Source** from the Command Palette and give it `https://github.com/chattee-ai/plugins`.

Then ask Copilot, for example, "Plan a one-page site on Chattee for a dog-walking service and show me the plan." The skill tells Copilot to show you the whole plan and to wait for your approval before it is built. We have checked that Copilot CLI installs the plugin and finds the server, and have not yet tested signing in from Copilot or VS Code.

## Other assistants

The server works without the plugin. These are the setups the docs describe:

| Assistant | How it connects |
|---|---|
| Claude.ai, Claude Desktop | A [custom connector](https://chattee.ai/docs/mcp#how-do-i-add-chattee-as-a-custom-connector-in-claudeai-and-claude-desktop), signed in with OAuth |
| ChatGPT | [Developer mode](https://chattee.ai/docs/mcp#how-do-i-connect-the-chattee-mcp-server-to-chatgpt), signed in with OAuth |
| Codex CLI | [One command or three lines of TOML](https://chattee.ai/docs/mcp#how-do-i-add-the-chattee-mcp-server-to-codex-cli), with a personal access token |
| Claude Agent SDK | [A server entry in your options](https://chattee.ai/docs/mcp#how-do-i-use-the-chattee-mcp-server-from-the-claude-agent-sdk), with a personal access token |

This repository also carries manifests for Codex, Gemini CLI and Antigravity. We have checked the files against those clients' formats and have not yet tested signing in from each of them. Any other client that speaks Streamable HTTP and OAuth, such as Zed or Devin Desktop, takes the server by its URL:

```text
https://chattee.ai/mcp/connect
```

## Just the skill

With Node.js 22.20 or newer, one command installs the skill into the agents you choose:

```text
npx skills add https://chattee.ai
```

It is the same file as [skill.md](https://chattee.ai/docs/mcp/skill.md), fetched from chattee.ai. The `skills` tool is Vercel's and reports installs anonymously; set `DISABLE_TELEMETRY=1` to turn that off. Claude.ai takes the skill as a [zip](https://chattee.ai/docs/mcp/chattee-skill.zip), uploaded under Customize → Skills.

The skill alone does not connect the server; the sections above do.

## What it sends, and to whom

The plugin holds no code and runs nothing on your machine: it is a server address, a skill and three prompts, all plain text. It talks to one service, `https://chattee.ai/mcp/connect`, over HTTPS and signed in as you.

- **Sent to Chattee:** what your assistant puts into a tool call: your brief, messages, answers and decisions, and any file or setting value it attaches. The plugin and the server cannot read your machine themselves; your assistant decides what goes into a call, so check a call that carries a file before you allow it.
- **Returned to your assistant:** your Chattee projects' plans, run states, files, logs, links and credit balance, and the query results you ask for.
- **Processed by:** Chattee, operated by Anwendo GmbH (a German company), with its hosting provider, and Anthropic, whose AI models plan and build: what you send for a build reaches them.
- **Kept:** your projects and their chats in your Chattee account and, for 30 days, a log of each tool call: when, which tool, project and client, and the outcome, not the content.

You can revoke the connection at any time on the [AI assistants page](https://chattee.ai/app/ai-assistants).

## Updating

Each release is a version in [CHANGELOG.md](CHANGELOG.md). Update the marketplace in your client's plugin manager, then the plugin.

## Privacy and support

- Privacy policy: https://chattee.ai/p/privacy-policy
- Terms: https://chattee.ai/p/terms
- Documentation: https://chattee.ai/docs/mcp
- Support: https://chattee.ai/p/contact
- Security: report a vulnerability to info@chattee.ai ([SECURITY.md](SECURITY.md))

## License

MIT — see [LICENSE](LICENSE). It covers the files in this repository; the Chattee service is proprietary.
