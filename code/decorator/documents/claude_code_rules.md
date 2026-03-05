==
Claude code tutorial
===

# Claude Code Rules Feature

Rules are modular instruction files stored in `.claude/rules/` that Claude Code
automatically loads at the start of every session (v2.0.64+). They let you split
project guidance into focused, topic-based files instead of one large `CLAUDE.md`.

## Structure

```
.claude/
├── CLAUDE.md
└── rules/
    ├── code-style.md
    ├── testing.md
    └── security.md
```

- **Project-level:** `<project>/.claude/rules/`
- **User-level:** `~/.claude/rules/` (lower priority, applies across all projects)

## Rules vs CLAUDE.md

| Aspect          | CLAUDE.md      | .claude/rules/       |
|-----------------|----------------|----------------------|
| Structure       | Single file    | Multiple files       |
| Ideal for       | Small projects | Large projects/teams |
| Maintainability | Can get unwieldy | Easier to manage   |

Both load automatically and work together.

## Best Practices

- Keep each rule file under 500 lines.
- Organize by topic (style, testing, security).
- Remove or update obsolete rules regularly.
- Use symlinks to share rules across projects.

===
Created by claude
===
