===
Claude code tutorial
===

# CLAUDE.md Guide

CLAUDE.md files give Claude persistent instructions. They load at the start of every
session. More specific locations take precedence over broader ones.

## Where It Lives

| Scope              | Location                                                  | Shared with                     |
|--------------------|-----------------------------------------------------------|---------------------------------|
| **Managed policy** | `/Library/Application Support/ClaudeCode/CLAUDE.md`       | All users in organization       |
| **Project**        | `./CLAUDE.md` or `./.claude/CLAUDE.md`                    | Team members via source control |
| **User**           | `~/.claude/CLAUDE.md`                                     | Just you (all projects)         |
| **Local**          | `./CLAUDE.local.md`                                       | Just you (current project)      |

CLAUDE.md files in parent directories load automatically. Subdirectory CLAUDE.md files
load on demand when Claude reads files in those directories.

## Tips

- Keep each file under 200 lines for best adherence.
- Use `@path/to/file` syntax to import additional files.
- Split large instructions into `.claude/rules/` topic files.
- Use `CLAUDE.local.md` for personal preferences not checked into git.

===
Created by claude
===
