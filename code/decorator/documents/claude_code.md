===
Claude code tutorial
===

# Claude Code

Claude Code is Anthropic's official agentic CLI tool that brings Claude directly into
your terminal. It understands your codebase, executes commands, edits files, and handles
complex software engineering tasks through natural language conversation.

## Key Features

- **Agentic Coding:** Describe what you want in plain English and Claude Code handles
  file edits, shell commands, and multi-step workflows autonomously.
- **Codebase Awareness:** Automatically indexes and understands your project structure,
  dependencies, and conventions.
- **Terminal-Native:** Runs entirely in the terminal with no browser or IDE required.
- **Tool Use:** Reads/writes files, runs tests, executes git commands, searches code,
  and interacts with external services.
- **CLAUDE.md Support:** Project-level instruction files that guide Claude Code's
  behavior for your specific codebase.
- **MCP Integration:** Connects to Model Context Protocol servers for extended
  capabilities like IDE interaction and database access.

## Getting Started

```bash
# Install globally
npm install -g @anthropic-ai/claude-code

# Start a session in your project directory
cd your-project
claude
```

## Common Workflows

| Workflow          | Example Prompt                                  |
|-------------------|-------------------------------------------------|
| Bug fixing        | "Fix the null pointer error in auth.ts"         |
| New features      | "Add a dark mode toggle to the settings page"   |
| Refactoring       | "Convert this class component to a hook"        |
| Code review       | "Review my last commit for issues"              |
| Test writing      | "Write unit tests for the UserService class"    |
| Git operations    | "Create a PR with a summary of my changes"      |

## Configuration

- **CLAUDE.md** - Project instructions checked into your repo.
- **.claude/rules/** - Modular rule files for larger projects.
- **~/.claude/settings.json** - Global user preferences.
- **Hooks** - Shell commands that run in response to Claude Code events.

## Tips

- Be specific in your prompts for better results.
- Use `/help` inside a session to see available commands.
- Let Claude Code read files before asking it to modify them.
- Use plan mode for complex tasks to review the approach before execution.

===
Created by claude
===
