# OpenCode Cheat Sheet

Quick reference for OpenCode commands, keybinds, and configuration.

---

## Installation

```bash
# Install script (recommended)
curl -fsSL https://opencode.ai/install | bash

# npm
npm install -g opencode-ai

# Homebrew (macOS/Linux)
brew install anomalyco/tap/opencode

# Arch Linux
sudo pacman -S opencode

# Chocolatey (Windows)
choco install opencode

# Docker
docker run -it --rm ghcr.io/anomalyco/opencode
```

---

## Starting OpenCode

```bash
# Launch TUI in current directory
opencode

# Launch TUI for a specific project
opencode /path/to/project

# Continue last session
opencode -c

# Continue a specific session
opencode -s <session-id>

# Fork a session when continuing
opencode -c --fork

# Use a specific model
opencode -m anthropic/claude-sonnet-4-5
```

---

## TUI Slash Commands

| Command | Alias | Keybind | Description |
|---|---|---|---|
| `/init` | | | Initialize project, create `AGENTS.md` |
| `/help` | | | Show help dialog |
| `/new` | `/clear` | `ctrl+x n` | Start a new session |
| `/sessions` | `/resume`, `/continue` | `ctrl+x l` | List and switch sessions |
| `/compact` | `/summarize` | `ctrl+x c` | Compact current session context |
| `/undo` | | `ctrl+x u` | Undo last message + file changes |
| `/redo` | | `ctrl+x r` | Redo previously undone message |
| `/share` | | | Share current session (copies link) |
| `/unshare` | | | Unshare current session |
| `/models` | | `ctrl+x m` | List available models |
| `/themes` | | `ctrl+x t` | Switch themes |
| `/editor` | | `ctrl+x e` | Open external editor for message |
| `/export` | | `ctrl+x x` | Export conversation to Markdown |
| `/details` | | | Toggle tool execution details |
| `/thinking` | | | Toggle thinking/reasoning blocks |
| `/connect` | | | Add a provider and API key |
| `/exit` | `/quit`, `/q` | `ctrl+x q` | Exit OpenCode |

### Examples

```
# Initialize a new project
/init

# Start a fresh conversation
/new

# Undo the last change (can be run multiple times)
/undo

# Redo what was just undone
/redo

# Share your conversation
/share

# Compact context when it gets large
/compact
```

---

## TUI Input Features

### File References

Use `@` to fuzzy search and attach files to your prompt:

```
How is auth handled in @src/api/auth.ts?
```

### Bash Commands

Prefix with `!` to run shell commands (output is added to conversation):

```
!ls -la
!git status
!npm test
```

### Mode Switching

Press `Tab` to toggle between **Build mode** and **Plan mode**:

```
<TAB>    # Switch to Plan mode (read-only suggestions)
<TAB>    # Switch back to Build mode (makes changes)
```

### Image Input

Drag and drop images into the terminal to add them to the prompt.

---

## Keyboard Shortcuts

The default leader key is `ctrl+x`. Press the leader key first, then the shortcut.

### Global

| Keybind | Action |
|---|---|
| `ctrl+p` | Open command palette |
| `ctrl+x n` | New session |
| `ctrl+x l` | List sessions |
| `ctrl+x m` | List models |
| `ctrl+x a` | List agents |
| `ctrl+x t` | Switch theme |
| `ctrl+x s` | OpenCode status |
| `ctrl+x q` | Exit |
| `ctrl+c` | Exit / Clear prompt |
| `ctrl+z` | Suspend terminal |
| `Tab` | Cycle agent (Build/Plan) |
| `Shift+Tab` | Cycle agent reverse |
| `ctrl+t` | Cycle model variant |
| `F2` | Cycle recent models |

### Session / Conversation

| Keybind | Action |
|---|---|
| `ctrl+x c` | Compact session |
| `ctrl+x u` | Undo last message |
| `ctrl+x r` | Redo message |
| `ctrl+x e` | Open external editor |
| `ctrl+x x` | Export conversation |
| `ctrl+x y` | Copy messages |
| `ctrl+x b` | Toggle sidebar |
| `ctrl+x h` | Toggle concealed content |
| `ctrl+x g` | Session timeline |
| `PageUp` | Scroll up one page |
| `PageDown` | Scroll down one page |
| `Home` | Scroll to top |
| `End` | Scroll to bottom |

### Input / Editing

| Keybind | Action |
|---|---|
| `Enter` | Submit message |
| `Shift+Enter` | New line |
| `ctrl+a` | Move to start of line |
| `ctrl+e` | Move to end of line |
| `ctrl+k` | Delete to end of line |
| `ctrl+u` | Delete to start of line |
| `ctrl+w` | Delete previous word |
| `ctrl+d` | Delete character under cursor |
| `alt+f` | Move forward one word |
| `alt+b` | Move backward one word |
| `Up` | Previous prompt history |
| `Down` | Next prompt history |

---

## CLI Commands

### Non-Interactive Run

```bash
# Run a one-off prompt
opencode run "Explain how closures work in JavaScript"

# Run with a specific model
opencode run -m anthropic/claude-sonnet-4-5 "Refactor this function"

# Attach a file to the prompt
opencode run -f src/index.ts "Review this file"

# Continue last session non-interactively
opencode run -c "What was the last thing we discussed?"

# Output as JSON events
opencode run --format json "List all TODO comments"

# Share the session after running
opencode run --share "Summarize the codebase"

# Attach to a running server
opencode serve &
opencode run --attach http://localhost:4096 "Explain async/await"
```

### Session Management

```bash
# List all sessions
opencode session list

# List last 5 sessions
opencode session list -n 5

# List sessions as JSON
opencode session list --format json

# Delete a session
opencode session delete <sessionID>

# Export a session as JSON
opencode export <sessionID>

# Export with redacted sensitive data
opencode export --sanitize <sessionID>

# Import a session from file or URL
opencode import session.json
opencode import https://opncd.ai/s/abc123
```

### Authentication

```bash
# Login to a provider
opencode auth login

# Login to a specific provider
opencode auth login -p anthropic

# List authenticated providers
opencode auth list

# Logout from a provider
opencode auth logout
```

### Model Management

```bash
# List all available models
opencode models

# List models for a specific provider
opencode models anthropic

# Refresh model cache
opencode models --refresh

# Show verbose model info (costs, metadata)
opencode models --verbose
```

### MCP Server Management

```bash
# Add an MCP server (interactive)
opencode mcp add

# List configured MCP servers
opencode mcp list

# Authenticate with an OAuth MCP server
opencode mcp auth <server-name>

# List OAuth-capable servers
opencode mcp auth list

# Remove OAuth credentials
opencode mcp logout <server-name>

# Debug OAuth issues
opencode mcp debug <server-name>
```

### Agent Management

```bash
# Create a new agent (interactive)
opencode agent create

# Create non-interactively
opencode agent create \
  --description "Reviews code for security issues" \
  --mode subagent \
  --permissions bash,read,grep,glob \
  --path .opencode/agents

# List all agents
opencode agent list
```

### Server & Web

```bash
# Start headless API server
opencode serve

# Start on a specific port
opencode serve --port 4096 --hostname 0.0.0.0

# Start with web interface
opencode web

# Attach TUI to remote server
opencode attach http://10.20.30.40:4096
```

### GitHub Integration

```bash
# Install GitHub agent in repo
opencode github install

# Run GitHub agent (used in CI)
opencode github run

# Fetch and checkout a PR branch
opencode pr 123
```

### Other Commands

```bash
# Show token usage and cost stats
opencode stats
opencode stats --days 7
opencode stats --models 5

# Install a plugin
opencode plugin opencode-helicone-session

# Upgrade to latest version
opencode upgrade

# Upgrade to specific version
opencode upgrade v0.1.48

# Uninstall OpenCode
opencode uninstall

# Dry run uninstall (see what would be removed)
opencode uninstall --dry-run

# Print database path
opencode db path

# Run a database query
opencode db "SELECT * FROM sessions LIMIT 5"
```

### Global Flags

```bash
opencode --help          # Display help
opencode --version       # Print version
opencode --print-logs    # Print logs to stderr
opencode --log-level DEBUG  # Set log level
opencode --pure          # Run without external plugins
```

---

## Custom Commands

### Create via Markdown

Create `.opencode/commands/<name>.md`:

```markdown
---
description: Run tests with coverage
agent: build
model: anthropic/claude-sonnet-4-5
---

Run the full test suite with coverage report and show any failures.
Focus on the failing tests and suggest fixes.
```

Then run: `/test`

### Create via Config (opencode.json)

```json
{
  "command": {
    "test": {
      "template": "Run the full test suite with coverage.",
      "description": "Run tests with coverage",
      "agent": "build"
    }
  }
}
```

### Arguments in Commands

```markdown
---
description: Create a new component
---

Create a new React component named $ARGUMENTS with TypeScript.
```

Run: `/component Button`

### Positional Arguments

```markdown
---
description: Create a file
---

Create a file named $1 in the directory $2 with the content: $3
```

Run: `/create-file config.json src "{ \"key\": \"value\" }"`

### Shell Output in Commands

```markdown
---
description: Review recent changes
---

Recent git commits:
!`git log --oneline -10`

Review these changes and suggest improvements.
```

### File References in Commands

```markdown
---
description: Review component
---

Review the component in @src/components/Button.tsx.
Check for performance issues and suggest improvements.
```

---

## Configuration

### Config Locations (by precedence, later overrides earlier)

1. Remote config (`.well-known/opencode`)
2. Global: `~/.config/opencode/opencode.json`
3. Custom: `OPENCODE_CONFIG` env var
4. Project: `opencode.json` in project root
5. `.opencode/` directories
6. Inline: `OPENCODE_CONFIG_CONTENT` env var

### Minimal opencode.json

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "anthropic/claude-sonnet-4-5",
  "autoupdate": true
}
```

### TUI Config (tui.json)

```json
{
  "$schema": "https://opencode.ai/tui.json",
  "theme": "opencode",
  "mouse": true,
  "scroll_speed": 3,
  "diff_style": "auto",
  "keymap": {
    "leader": "ctrl+x",
    "leader_timeout": 2000
  }
}
```

### Permissions

```json
{
  "permission": {
    "edit": "ask",
    "bash": "ask",
    "write": "ask"
  }
}
```

### Formatters

```json
{
  "formatter": true
}
```

### LSP Servers

```json
{
  "lsp": true
}
```

### Compaction

```json
{
  "compaction": {
    "auto": true,
    "prune": true,
    "reserved": 10000
  }
}
```

---

## Key Environment Variables

| Variable | Description |
|---|---|
| `OPENCODE_CONFIG` | Path to custom config file |
| `OPENCODE_TUI_CONFIG` | Path to custom TUI config file |
| `OPENCODE_CONFIG_DIR` | Path to custom config directory |
| `OPENCODE_CONFIG_CONTENT` | Inline JSON config content |
| `OPENCODE_DISABLE_AUTOUPDATE` | Disable automatic updates |
| `OPENCODE_DISABLE_AUTOCOMPACT` | Disable auto context compaction |
| `OPENCODE_SERVER_PASSWORD` | Enable basic auth for serve/web |
| `OPENCODE_SERVER_USERNAME` | Override basic auth username |
| `OPENCODE_AUTO_SHARE` | Automatically share sessions |
| `OPENCODE_DISABLE_MOUSE` | Disable mouse capture in TUI |
| `EDITOR` | Editor for `/editor` and `/export` |

---

## Project Structure

```
project/
  opencode.json          # Project config (server/runtime)
  tui.json               # Project TUI config
  AGENTS.md              # Project rules (created by /init)
  .opencode/
    agents/              # Custom agent definitions
    commands/            # Custom slash commands
    plugins/             # Local plugins
    skills/              # Agent skills
    tools/               # Custom tools

~/.config/opencode/
  opencode.json          # Global config
  tui.json               # Global TUI config
  agents/                # Global agents
  commands/              # Global commands
  plugins/               # Global plugins
```

---

## Quick Workflow Examples

### Start a new project

```bash
cd /path/to/project
opencode
```
```
/init
```

### Ask questions about code

```
How is authentication handled in @src/api/auth.ts?
```

### Plan then build a feature

```
<TAB>                          # Switch to Plan mode
Add soft-delete for notes with an undo screen
<TAB>                          # Switch to Build mode
Sounds good! Go ahead and make the changes.
```

### Undo and retry

```
/undo                          # Revert changes
Can you try a different approach using...
```

### Non-interactive scripting

```bash
opencode run "Add error handling to all API endpoints" --share
```

### Review a PR

```bash
opencode pr 42
```
```
Review the changes in this PR for security and performance issues.
```

---

## Links

- Docs: https://opencode.ai/docs
- GitHub: https://github.com/anomalyco/opencode
- Discord: https://opencode.ai/discord
- Config Schema: https://opencode.ai/config.json
- TUI Schema: https://opencode.ai/tui.json
