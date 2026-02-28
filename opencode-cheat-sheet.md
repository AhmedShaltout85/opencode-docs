# OpenCode Command Reference

A complete reference guide for all OpenCode CLI commands, TUI slash commands, flags, and environment variables — with examples for each.

---

## Table of Contents

- [Core Commands](#core-commands)
- [Session Management](#session-management)
- [Auth Commands](#auth-commands)
- [Agent Commands](#agent-commands)
- [Model Commands](#model-commands)
- [MCP Commands](#mcp-commands)
- [GitHub Agent Commands](#github-agent-commands)
- [Global Flags](#global-flags)
- [TUI Slash Commands](#tui-slash-commands)
- [Custom Commands](#custom-commands)
- [Environment Variables](#environment-variables)

---

## Core Commands

### `opencode`
Launch the interactive TUI (Terminal User Interface).

```bash
opencode
```

---

### `opencode run`
Run a prompt non-interactively (headless mode). Useful for scripting and automation.

```bash
# Basic usage
opencode run "Refactor this file to use async/await"

# Run against a specific file
opencode run "Add JSDoc comments to all functions" -f src/utils.js

# Output the response as JSON
opencode run "List all TODO comments in this repo" --format json

# Use a specific model
opencode run "Write unit tests for auth.ts" --model anthropic/claude-sonnet-4-6
```

---

### `opencode serve`
Start a headless backend server. Other clients (TUI, web, or remote) can attach to it.

```bash
# Start on the default port
opencode serve

# Start on a custom port
opencode serve --port 4242

# Start with HTTP basic auth
OPENCODE_SERVER_PASSWORD=mysecret opencode serve --port 4242
```

---

### `opencode web`
Start a backend server and automatically open the web UI in your browser.

```bash
# Start web UI on default port
opencode web

# Start on a custom port
opencode web --port 8080

# Bind to a specific hostname
opencode web --hostname 0.0.0.0 --port 8080
```

---

### `opencode attach`
Attach the TUI to an already-running remote `opencode serve` instance.

```bash
# Attach to a local server
opencode attach http://localhost:4242

# Attach to a remote server
opencode attach https://myserver.example.com:4242
```

---

### `opencode acp`
Start an ACP (Agent Communication Protocol) server using stdin/stdout newline-delimited JSON. Useful for integrating OpenCode into other tools or editors.

```bash
opencode acp
```

---

### `opencode init`
Analyze the current project and create or update an `AGENTS.md` file with context about the codebase for agents.

```bash
# Run in the current directory
opencode init

# Run in a specific directory
opencode init /path/to/my/project
```

---

### `opencode pr`
Check out a GitHub Pull Request branch and start OpenCode in that context — great for reviewing or working on PRs.

```bash
# Open PR #42 from the current repo
opencode pr 42

# With a specific model
opencode pr 42 --model openai/gpt-4o
```

---

### `opencode upgrade`
Update OpenCode to the latest or a specific version.

```bash
# Upgrade to latest
opencode upgrade

# Upgrade to a specific version
opencode upgrade 0.4.2
```

---

### `opencode uninstall`
Uninstall OpenCode and remove all associated files and configuration.

```bash
opencode uninstall
```

---

## Session Management

### `opencode session list`
List all past sessions with their IDs, timestamps, and summaries.

```bash
opencode session list
```

---

### `opencode session stats`
Show token usage and cost statistics across sessions.

```bash
# Stats for all sessions
opencode session stats

# Stats for a specific session
opencode session stats --session abc123
```

---

### `opencode session export`
Export session data as a JSON file.

```bash
# Export the most recent session
opencode session export

# Export a specific session by ID
opencode session export --session abc123

# Export to a specific file
opencode session export --session abc123 --output my-session.json
```

---

### `opencode import`
Import a session from a JSON file or a share URL.

```bash
# Import from a local file
opencode import my-session.json

# Import from a share URL
opencode import https://opencode.ai/share/abc123
```

---

## Auth Commands

### `opencode auth login`
Add or configure an API key for a provider (e.g., Anthropic, OpenAI, Google).

```bash
# Interactive login — prompts you to choose a provider and enter your key
opencode auth login

# Login for a specific provider
opencode auth login anthropic
opencode auth login openai
opencode auth login google
```

---

### `opencode auth list`
List all authenticated providers and their current status.

```bash
opencode auth list
```

---

### `opencode auth logout`
Remove stored credentials for a provider.

```bash
# Logout from a specific provider
opencode auth logout anthropic

# Logout from all providers
opencode auth logout --all
```

---

## Agent Commands

### `opencode agent create`
Interactively create a new custom agent with a name, model, system prompt, and tools.

```bash
opencode agent create
```

---

### `opencode agent list`
List all available agents, including built-in and custom ones.

```bash
opencode agent list
```

---

## Model Commands

### `opencode models`
List all available models across all configured providers.

```bash
opencode models
```

---

### `opencode models <providerID>`
Filter the model list to a specific provider.

```bash
opencode models anthropic
opencode models openai
opencode models google
```

---

### `opencode models --refresh`
Force a refresh of the cached model list from all providers.

```bash
opencode models --refresh
```

---

## MCP Commands

MCP (Model Context Protocol) servers extend OpenCode with additional tools and capabilities.

### `opencode mcp add`
Add a local or remote MCP server to your configuration.

```bash
# Add a local stdio-based MCP server
opencode mcp add my-tools --command "node /path/to/mcp-server.js"

# Add a remote HTTP MCP server
opencode mcp add my-remote-tools --url https://mcp.example.com
```

---

### `opencode mcp list`
List all configured MCP servers and their status.

```bash
opencode mcp list
```

---

### `opencode mcp debug <name>`
Debug a specific MCP server connection and inspect its tools and responses.

```bash
opencode mcp debug my-tools
```

---

### `opencode mcp oauth login`
Authenticate with an MCP server that uses OAuth.

```bash
opencode mcp oauth login my-oauth-server
```

---

### `opencode mcp oauth list`
List all OAuth-capable MCP servers and their current authentication status.

```bash
opencode mcp oauth list
```

---

### `opencode mcp oauth logout`
Remove OAuth credentials for an MCP server.

```bash
opencode mcp oauth logout my-oauth-server
```

---

### `opencode mcp oauth debug`
Debug OAuth connection issues for an MCP server.

```bash
opencode mcp oauth debug my-oauth-server
```

---

## GitHub Agent Commands

### `opencode github install`
Install the OpenCode GitHub Actions agent into your repository. This adds a workflow file that enables OpenCode to respond to issues and PRs automatically.

```bash
# Run from inside your repo
opencode github install
```

---

### `opencode github run`
Manually trigger the GitHub agent workflow.

```bash
opencode github run
```

---

## Global Flags

These flags can be used with most commands.

| Flag | Description | Example |
|---|---|---|
| `-p "prompt"` | Run non-interactively with a prompt | `opencode -p "Fix the linting errors"` |
| `--model <model>` | Use a specific model | `opencode --model anthropic/claude-sonnet-4-6` |
| `--continue` | Continue from the most recent session | `opencode --continue` |
| `--attach <url>` | Attach to a running `opencode serve` instance | `opencode --attach http://localhost:4242` |
| `--port <port>` | Port for `web` and `serve` commands | `opencode serve --port 8080` |
| `--hostname <host>` | Hostname for `web` and `serve` commands | `opencode web --hostname 0.0.0.0` |
| `-f json` / `--format json` | Output response as JSON | `opencode run "list issues" -f json` |
| `-q` / `--quiet` | Suppress spinner output (good for scripts) | `opencode run "..." -q` |

### Examples

```bash
# Run a prompt quietly, outputting JSON, with a specific model
opencode run "Find all unused imports" --model openai/gpt-4o -f json -q

# Launch TUI and continue previous session
opencode --continue

# Launch TUI connected to a remote backend
opencode --attach http://myserver.com:4242
```

---

## TUI Slash Commands

These commands are typed directly inside the OpenCode TUI chat interface.

### `/init`
Analyze the current project and create or update `AGENTS.md`.

```
/init
```

---

### `/help`
Display a list of available slash commands.

```
/help
```

---

### `/connect`
Add or update provider credentials without leaving the TUI.

```
/connect
```

---

### `/models`
Open the model selector to switch the active model mid-session.

```
/models
```

---

### `/undo`
Undo the last change made by the agent to your files.

```
/undo
```

---

### `/redo`
Redo an undone change.

```
/redo
```

---

### `/share`
Generate a shareable link for the current session.

```
/share
```

---

### `/export`
Export the current conversation to a Markdown file.

```
/export
/export my-session.md
```

---

### `/editor`
Open your external `$EDITOR` (e.g., vim, nano, VS Code) to compose a longer, multi-line prompt.

```
/editor
```

---

## Custom Commands

You can create custom slash commands by placing `.md` files in a `commands/` directory at your project root or in `~/.config/opencode/commands/` for global commands.

**Example: `.opencode/commands/test.md`**
```markdown
Run the test suite and summarize any failures.

$ npm test
```

This creates a `/test` command available in the TUI.

**Template features:**

```markdown
# Using arguments
Review the following file for security issues: $ARGUMENTS

# Inject bash output
Current git status:
!git status

# Include file contents
Review this file:
@src/auth.ts
```

```bash
# Usage in TUI:
/test
/review src/auth.ts
```

---

## Environment Variables

| Variable | Description | Example |
|---|---|---|
| `OPENCODE_CONFIG` | Path to a specific config file | `OPENCODE_CONFIG=~/myconfig.json opencode` |
| `OPENCODE_CONFIG_DIR` | Path to the config directory | `OPENCODE_CONFIG_DIR=~/.config/opencode opencode` |
| `OPENCODE_SERVER_PASSWORD` | Password for HTTP basic auth on `web`/`serve` | `OPENCODE_SERVER_PASSWORD=secret opencode serve` |
| `OPENCODE_SERVER_USERNAME` | Username for HTTP basic auth (defaults to `opencode`) | `OPENCODE_SERVER_USERNAME=admin opencode serve` |
| `OPENCODE_DISABLE_AUTOUPDATE` | Disable automatic update checks | `OPENCODE_DISABLE_AUTOUPDATE=1 opencode` |

### Examples

```bash
# Start a password-protected web server
OPENCODE_SERVER_USERNAME=admin OPENCODE_SERVER_PASSWORD=supersecret opencode web --port 8080

# Use a custom config directory
OPENCODE_CONFIG_DIR=/etc/opencode opencode serve

# Disable auto-updates in CI
OPENCODE_DISABLE_AUTOUPDATE=1 opencode run "Run code review" -q
```

---

*Generated with OpenCode Command Reference — February 2026*
