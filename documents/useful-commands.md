# Useful Slash Commands

Type `/` inside a Claude Code session to see all commands, or type `/` followed by
letters to filter. Some commands are only visible depending on your platform or plan.

## Session Management

| Command                 | Description                                                                |
|-------------------------|----------------------------------------------------------------------------|
| `/clear`                | Clear conversation history and free up context. Aliases: `/reset`, `/new` |
| `/compact [focus]`      | Summarize conversation to free context, with optional focus instructions   |
| `/resume [session]`     | Resume a previous session by ID/name, or open a picker. Alias: `/continue`|
| `/fork [name]`          | Create a fork of the current conversation at this point                    |
| `/rename [name]`        | Rename the session. Omit name to auto-generate one                         |
| `/exit`                 | Exit the CLI. Alias: `/quit`                                               |

## Project & Memory

| Command                 | Description                                                                |
|-------------------------|----------------------------------------------------------------------------|
| `/init`                 | Analyze the codebase and generate a starter `CLAUDE.md`                    |
| `/memory`               | Browse loaded CLAUDE.md files, toggle auto-memory, edit memory entries     |
| `/add-dir <path>`       | Add another working directory to the current session                       |

Example — bootstrap a new project:
```
/init
```
Claude scans your repo and creates a `CLAUDE.md` with build commands, test instructions,
and conventions it discovers. If one already exists, it suggests improvements.

## Diagnostics & Status

| Command                 | Description                                                                |
|-------------------------|----------------------------------------------------------------------------|
| `/doctor`               | Diagnose and verify your Claude Code installation and settings             |
| `/status`               | Show version, model, account, and connectivity info                        |
| `/cost`                 | Show token usage statistics for the current session                         |
| `/usage`                | Show plan usage limits and rate limit status                                |
| `/context`              | Visualize current context usage as a colored grid                          |

Example — check why something feels off:
```
/doctor
```
Runs health checks on your installation, auth, and settings.

## Code Review & Security

| Command                 | Description                                                                |
|-------------------------|----------------------------------------------------------------------------|
| `/review`               | Review a PR for quality, correctness, security, and test coverage          |
| `/security-review`      | Analyze pending changes on the current branch for security vulnerabilities |
| `/diff`                 | Interactive diff viewer showing uncommitted changes and per-turn diffs     |
| `/pr-comments [PR]`     | Fetch and display comments from a GitHub PR                                |

Example — review PR #42:
```
/review 42
```
Requires the `gh` CLI to be installed and authenticated.

## Configuration

| Command                 | Description                                                                |
|-------------------------|----------------------------------------------------------------------------|
| `/config`               | Open the settings interface. Alias: `/settings`                            |
| `/permissions`          | View or update tool permissions. Alias: `/allowed-tools`                   |
| `/model [model]`        | Select or change the AI model mid-session                                  |
| `/theme`                | Change the color theme (includes dark, light, and colorblind variants)     |
| `/vim`                  | Toggle between Vim and Normal editing modes                                |
| `/fast [on|off]`        | Toggle fast mode (same model, faster output)                               |
| `/terminal-setup`       | Configure Shift+Enter and other keybindings for your terminal              |
| `/keybindings`          | Open or create your keybindings configuration file                         |
| `/hooks`                | Manage hook configurations for tool events                                 |
| `/sandbox`              | Toggle sandbox mode (platform-dependent)                                   |

## Auth

| Command                 | Description                                                                |
|-------------------------|----------------------------------------------------------------------------|
| `/login`                | Sign in to your Anthropic account                                          |
| `/logout`               | Sign out from your Anthropic account                                       |

## Integrations

| Command                 | Description                                                                |
|-------------------------|----------------------------------------------------------------------------|
| `/ide`                  | Manage IDE integrations and show connection status                         |
| `/mcp`                  | Manage MCP server connections and OAuth authentication                     |
| `/chrome`               | Configure Chrome browser integration                                       |
| `/install-github-app`   | Set up Claude GitHub Actions for a repository                              |
| `/install-slack-app`    | Install the Claude Slack app via OAuth                                     |
| `/agents`               | Manage subagent configurations                                             |
| `/skills`               | List available skills                                                      |
| `/plugin`               | Manage Claude Code plugins                                                 |

## Misc

| Command                 | Description                                                                |
|-------------------------|----------------------------------------------------------------------------|
| `/help`                 | Show help and available commands                                           |
| `/copy`                 | Copy the last response to clipboard (with code block picker)               |
| `/export [filename]`    | Export the conversation as plain text                                       |
| `/feedback [report]`    | Submit feedback about Claude Code. Alias: `/bug`                           |
| `/release-notes`        | View the full changelog                                                    |
| `/stats`                | Visualize daily usage, session history, streaks, and model preferences     |
| `/insights`             | Generate a report analyzing your Claude Code sessions                      |
| `/tasks`                | List and manage background tasks                                           |
| `/rewind`               | Rewind conversation and/or code to a previous point. Alias: `/checkpoint`  |
| `/plan`                 | Enter plan mode directly from the prompt                                   |
| `/output-style [style]` | Switch output style: Default, Explanatory, or Learning                    |
| `/desktop`              | Continue session in the Claude Code Desktop app (macOS/Windows)            |
| `/remote-control`       | Make session available for remote control from claude.ai. Alias: `/rc`     |

## Quick Prefixes

These are not slash commands but useful shortcuts at the start of your input:

| Prefix | Description                                                |
|--------|------------------------------------------------------------|
| `!`    | Run a bash command directly, output added to context       |
| `@`    | Trigger file path autocomplete                             |

Example — quick shell command without leaving the session:
```
! git status
```
