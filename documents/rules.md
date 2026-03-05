# Rules

Similar to CLAUDE.md file

## Official documentation

[Claude code rules](https://code.claude.com/docs/en/memory#organize-rules-with-claude%2Frules%2F)

## How to setup

```
your-project/
├── .claude/
│   ├── CLAUDE.md           # Main project instructions
│   └── rules/
│       ├── code-style.md   # Code style guidelines
│       ├── testing.md      # Testing conventions
│       └── security.md     # Security requirements
```


## How it works

- rules are not triggered automatically, only triggers when you read something in the given directory
- if you create/modify any files there, it might not apply UNLESS you read something from there before
