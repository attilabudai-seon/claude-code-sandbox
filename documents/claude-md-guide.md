# CLAUDE.md

CLAUDE.md is a configuration file that lives at the root of your project (or in ~/.claude/ for global preferences). Its contents are injected into every conversation turn as part of the user prompt, wrapped in <system-reminder> tags. It serves as persistent, always-available context that Claude reads before responding to any request.

[Official documentation](https://code.claude.com/docs/en/memory#claude-md-files)

## Where It Lives

| Scope              | Location                                                  | Shared with                     |
|--------------------|-----------------------------------------------------------|---------------------------------|
| **Managed policy** | `/Library/Application Support/ClaudeCode/CLAUDE.md`       | All users in organization       |
| **Project**        | `./CLAUDE.md` or `./.claude/CLAUDE.md`                    | Team members via source control |
| **User**           | `~/.claude/CLAUDE.md`                                     | Just you (all projects)         |
| **Local**          | `./CLAUDE.local.md`                                       | Just you (current project)      |

CLAUDE.md files in parent directories load automatically. Subdirectory CLAUDE.md files
load on demand when Claude reads files in those directories.

### The Loading Hierarchy

```
~/.claude/CLAUDE.md           -> Global preferences (always loaded)
~/.claude/rules/              -> Global rules (always loaded)
./CLAUDE.local.md             -> Personal project preferences (always loaded, gitignored)
project/CLAUDE.md             -> Project context (always loaded)
project/.claude/CLAUDE.md     -> Alt project location (always loaded)
project/.claude/rules/        -> Modular rules:
                                   - No path glob: loaded at start
                                   - With path glob: loaded when working with matching files
project/.claude/skills/       -> Skills (descriptions always in context, full content on invocation)
project/subdir/CLAUDE.md      -> Directory-specific (when working there)
```

### What It Is

CLAUDE.md is a configuration file that lives at the root of your project (or in `~/.claude/` for global preferences). Its contents are injected into every conversation turn as part of the user prompt, wrapped in `<system-reminder>` tags. It serves as persistent, always-available context that Claude reads before responding to any request.

Think of it as:
- A **routing table** that tells Claude where things are and how to behave
- A **constitution** that defines project-level rules and conventions
- An **index** that points to deeper documentation
- A **verification contract** — error handling patterns and function contracts (u/JWPapi)

### What It Isn't

- **Not a personality profile** — "Be helpful and thorough" wastes tokens and does nothing
- **Not a manual** — dumping your entire project documentation into it degrades performance
- **Not a memory system** — it's for static conventions, not dynamic state
- **Not enforceable by itself** — Claude reads it but can "interpret loosely" or ignore it entirely
- **Not a replacement for good code** — if a convention can be enforced by a linter, use the linter
- **Not always necessary** — some developers report that consistent code patterns alone are sufficient

### The Key Technical Detail

CLAUDE.md is injected as a user prompt with a `<system-reminder>` wrapper that includes the caveat "may or may not be relevant." This gives Claude discretion to ignore instructions it deems irrelevant. This is both a feature (prevents rigid behavior) and a frustration (rules get skipped). The model pays 10x more attention to recent debugging chat than to CLAUDE.md. Understanding this is critical — you cannot rely on it alone for enforcement.

---

## 2. The Golden Rules

These principles are the most consistently agreed-upon across all community discussions:

### Rule 1: Keep It Minimal
Under 100-250 lines. It's a router/index, not an encyclopedia. When 8,000+ tokens of rules are loaded, Claude starts ignoring half of them past message 15. Multiple users report that trimming CLAUDE.md from thousands of words to a few hundred dramatically improved compliance and reduced token waste. One user went from 5,000 words to 500 — result: 50% fewer tokens, no lost context, dramatically fewer errors.

### Rule 2: Use Trigger-Action Rules
Write "When you encounter X, do Y" — not "Be helpful" or "Write clean code." Claude responds to specific, actionable decision trees far better than personality descriptions or abstract principles.

### Rule 3: Only Include What Claude Can't Deduce from Code
Document gotchas — things the model would get wrong without hints. A production incident turned into a one-liner is infinitely more valuable than "always write tests." If Claude can figure it out by reading the codebase, don't put it in CLAUDE.md.

### Rule 4: Use Progressive Disclosure
Reference deeper docs with `@path/to/file.md` — load on demand, not all at once. Your root CLAUDE.md should be a table of contents, not the book.

### Rule 5: Enforce with Hooks
CLAUDE.md rules are suggestions. Hooks are law. If something absolutely must happen (tests before commit, format before save), put it in a hook, not in CLAUDE.md.

### Rule 6: Every Mistake Becomes a Rule
Every "why did Claude do that wrong" moment should become a CLAUDE.md update. It compounds fast. After every correction: "Update your CLAUDE.md so you don't make that mistake again"

### Rule 7: Context Management Is the Meta-Skill
"The most important skill is and always has been context management. CLAUDE.md is a part of that, prompting is part of that, code health is part of that, tools/MCPs are part of that."
---
