# Claude Code Presentation — Speaker Guide

**Presenter:** Attila Budai, Senior Software Engineer
**Duration:** ~65–75 minutes (including demos and Q&A)
**Audience:** Intermediate-to-senior developers
**Goal:** Every attendee leaves with a working mental model of Claude Code's architecture, vocabulary, and daily workflow — enough to start using it productively.

---

## Presentation Philosophy

This is a **foundations session**. The goal is not to impress with advanced features — it's to make sure everyone shares a common vocabulary and understands how the pieces connect. Advanced topics (hooks deep-dive, subagent orchestration, parallel worktrees, CI/CD integration) are explicitly deferred to a follow-up session.

Lead with context handling, not product features. Context is the single concept that separates people who struggle with Claude Code from people who use it effectively. If the audience remembers one thing, it should be: "context is finite, manage it deliberately."

---

## Slide-by-Slide Guide

### Slide 1: Title

**Duration:** 1–2 minutes

Open with a quick framing statement. Don't read the slide — the audience can see it.

**What to say:**
"I know most of you are already using Windsurf daily, many of you have spent time in Cursor, and about half of you have at least tried Claude Code. So this isn't an intro-to-AI-coding-tools talk. You already know what these tools can do at the surface level. What we're going to do today is go deeper — specifically into what happens under the hood, why Claude Code makes certain design choices differently, and the one concept that determines whether any of these tools works well or falls apart. That concept is context. And I promise — even if you've been using these tools for months — some of what you'll see today will change how you use all of them, not just Claude Code."

**Key point:** Set expectations that this is foundational knowledge that applies across tools, not a sales pitch. The audience's existing experience is an asset — they'll immediately connect the concepts to things they've already encountered.

---

### Slide 2: The Context Window — What Actually Gets Sent

**Duration:** 5–7 minutes (this is the most important slide)

This is your opening hook. Spend real time here. Most developers have a fundamentally wrong mental model of how LLM conversations work.

**What to say:**
"Before we talk about Claude Code itself, I need to reframe something. You've all heard of prompt engineering — crafting the perfect instruction to get the best output. In mid-2025, Shopify's CEO Tobi Lütke and Andrej Karpathy from OpenAI started pushing a different term: context engineering. The shift is simple but profound. Prompt engineering is what you write inside the context window. Context engineering is how you decide what fills the window. Prompt engineering is about clever wording. Context engineering is about designing the entire information environment the model operates in — memory, tools, retrieval, history, rules.

Everything we're going to cover today — CLAUDE.md, skills, rules, hooks, MCP, /compact, /clear — is context engineering. You're not just writing prompts. You're architecting what the model sees. And this diagram shows you why that matters."

**Walk through the infographic left to right:**

**Before diving in — set the academic frame (optional, use if audience is technical):**
"This isn't just anecdotal. Stanford's 'Lost in the Middle' paper from 2024 documented that LLMs exhibit a U-shaped attention curve — high attention at the beginning (primacy) and end (recency) of context, with significant degradation in the middle. A 2025 follow-up showed this mirrors human short-term memory patterns. Larger models reduce but don't eliminate the U-curve. This is the physics of context — it applies to every LLM, every tool."

**Community-validated observations you can reference:**
- A Reddit user (u/Main_Payment_6430) put it perfectly: "If you have a perfect CLAUDE.md at the top, but 4,000 lines of debugging chat at the bottom, the model will pay 10x more attention to the debugging chat." S-tier performance for the first ~45 minutes, then it degrades into a "lazy junior dev."
- u/g3_SpaceTeam named the three problems we'll discuss: context ordering (recency bias), context saturation (needle-in-haystack), and context decay (stale information).
- u/helk1d's post (818 upvotes) called it the "lost in the middle dumb zone" — when context gets big, the middle becomes invisible.

**The attention map — use this mental model:**
"Think of the 200k token context window like this: CLAUDE.md sits near the top and benefits from primacy — the model pays strong attention to what comes first. Your latest message sits at the bottom and gets recency attention — the model focuses heavily on what's most recent. Everything in the middle — your turn 1 messages, turn 2 tool results, old debugging output — falls into a low-attention zone. As context grows, this middle zone expands and swallows your earlier instructions. This is why debugging noise overwrites architectural rules, even when CLAUDE.md explicitly says otherwise."

1. **Turn 1 (~15%):** "On the first message, the system prompt takes about 16k tokens. That's fixed overhead — CLAUDE.md, rules, skill descriptions, MCP tool definitions. Your actual conversation sits on top. Notice the context window limit at the top — 200k tokens."

2. **Turn 2 (~30%):** "Here's the thing most people miss: Turn 2 doesn't just send your new message. It re-sends everything from Turn 1 plus your new message. The entire history is transmitted every single turn. Tool results — file contents, grep output, test results — are the biggest contributor at 68% of context."

3. **Turn 3 (~52%):** "By turn 3, we're already past 50%. This is where quality starts to degrade. Notice the 'derivation noise' callout — wrong turns, rejected approaches, old debugging output. The model pays more attention to this recent noise than to your CLAUDE.md instructions at the bottom."

4. **Turn 4 (~78%):** "At 78%, the model's attention is 10x more focused on recent chat than on CLAUDE.md. You're getting hallucinations, dropped instructions, shallow responses."

5. **Turn 5 — The Split:** "This is the critical decision point. Path A: keep going at 95% — you get phantom edits, Claude claims to have made changes but the output is empty. Path B: fresh start via Plan Mode Leapfrog — context drops to 18%, peak performance restored."

**Key points to emphasize:**
- "This is not a chatbot. Every turn re-sends everything."
- "68% of context is tool results — most of which are re-fetchable. Claude can always re-read a file."
- "The 50% mark is your action threshold. When you feel quality dropping, it's not your imagination."

**Bottom section — briefly:**
- Point out the context composition bar (68% tool results)
- Point out the "What people think happens" vs "What actually happens" diagram

**Transition:** "Now that you understand why context matters, let's look at the specific numbers."

---

### Slide 3: Context Handling — The #1 Operational Skill

**Duration:** 3–4 minutes

This slide reinforces slide 2 with concrete metrics. Don't repeat the same narrative — focus on the actionable numbers.

**What to say:**
"Here are the numbers you need to internalize."

Walk through the four metric cards:
- **~50% peak performance:** "This is your ceiling for best-quality output."
- **~30 min / 70-100k tokens:** "If you've been working for 30 minutes straight, quality is already degrading."
- **22-38% auto-compact overhead:** "Auto-compaction sounds helpful, but it consumes a quarter of your context window just for the buffer."
- **Up to 66% MCP idle consumption:** "This one surprises everyone. MCP servers eat context just by being registered, even if you never call them. We'll cover this in detail shortly."

**Point to the bottom bar:** "Three distinct problems — ordering bias means Claude pays more attention to recent content. Saturation means finding a specific instruction gets harder as context grows. Decay means old context becomes wrong as your code changes."

**Transition:** "So what is this tool that's so sensitive to context management? Let's define it."

---

### Slide 4: What is Claude Code?

**Duration:** 4–5 minutes

**What to say:**
"Claude Code is Anthropic's official agentic coding tool. We use the word 'agentic' a lot in this space, so let's define it precisely."

**Definition card:** Read the description briefly, then move to the key section.

**"What makes it agentic?" — spend time here:**

"Agentic means autonomous goal pursuit through a loop. Not just responding to prompts — actually pursuing a goal through multiple steps. Here's the loop:"

Walk through the four cards:

1. **Plan:** "You give Claude a goal — 'migrate our auth from cookies to JWTs.' Claude breaks this into steps: identify all cookie-based auth code, design the JWT structure, update middleware, update API routes, update the client. It identifies risks — 'this will break existing sessions, we need a migration path.'"

2. **Act:** "Claude uses tools to execute the plan — edits files, runs shell commands, commits to git, calls MCP servers. These aren't suggestions in a chat window. Claude is actually writing to your filesystem and running your commands."

3. **Observe:** "After each action, Claude reads the results. It runs your tests, reads the error output, checks if the file it just wrote actually compiles. This is where most other tools stop — they act and then wait for you to tell them what happened."

4. **Iterate:** "Based on what it observed, Claude adjusts. Tests failed? It reads the stack trace, identifies the issue, and fixes it. Linter errors? It corrects them. This loop continues autonomously until the goal is met or Claude needs your input."

**Bottom bar — connecting back to context:**
"And here's why this connects to everything we discussed about context. Every iteration of this loop generates tool results — file reads, test output, error logs — that fill the context window. A complex task like a 23-file migration might go through 50+ Plan-Act-Observe-Iterate cycles. If you're not managing context, quality collapses before the task is done. The agentic loop is powerful, but context is its fuel. Manage the fuel, and the loop keeps running. Let it run out, and you get phantom edits and hallucinations."

**Transition:** "Now that we know what agentic means, let's establish a shared vocabulary for everything else in the Claude Code ecosystem."

**Why this matters for the audience:**
"This loop is why context management matters so much. Every iteration of Plan-Act-Observe-Iterate generates tool results that fill the context window. A 23-file migration might go through 50+ iterations. If you're not managing context, quality collapses before the task is done."

**Transition:** "Now that we know what agentic means, let's establish a shared vocabulary for everything else in the Claude Code ecosystem."

---

### Slide 5: The Claude Code Dictionary

**Duration:** 5–7 minutes (reference slide — go through each term)

This is the slide that directly addresses your goal of building a shared vocabulary. Go through each term deliberately.

**What to say for each:**

1. **CLAUDE.md:** "Your project's constitution. It lives in the project root and gets injected into every single conversation turn. It's where you put project rules, conventions, and routing instructions. Keep it under 250 lines — remember, it's consuming context every turn."

2. **Skills:** "Expert knowledge files in .claude/skills/. The key difference from Rules: skills are loaded on demand, not automatically. Claude loads them when it thinks they're relevant. If Claude doesn't know a skill exists, it won't load it — which is why your CLAUDE.md should list available skills."

3. **Hooks:** "Mandatory enforcement. Shell scripts that run before or after Claude acts. Unlike CLAUDE.md, which Claude can interpret loosely, hooks execute outside Claude's decision loop. Claude literally cannot skip them."

4. **Rules:** "Files in .claude/rules/ that are always loaded into context — like modular extensions of CLAUDE.md. Use these for testing standards, security policies, code style. The difference from Skills: rules are always present, skills are on-demand."

5. **MCP:** "Model Context Protocol — Anthropic's open standard for connecting Claude to external tools. Think of it as USB-C for AI. One protocol, many services — databases, APIs, GitHub, Jira, Slack."

6. **Subagent:** "A child Claude session with its own context window. The parent delegates a task, the subagent does the work, and only a summary comes back. This is how you keep the parent's context clean during large tasks."

7. **Commands:** "Custom workflows you define in .claude/commands/. Write markdown, invoke with /slash syntax. Your team's playbooks, automated."

8. **Agents:** "Specialists defined in .claude/agents/. Like subagents, but pre-configured with specific roles and instructions. A code-reviewer agent, a security-audit agent, each with their own context."

9. **Worktrees:** "A git feature — multiple working copies of the same repo. Claude Code uses this for parallel execution. You can have 3, 5, even 100 Claude sessions each working on different branches simultaneously."

10. **Memory:** "Claude's cross-session learnings. Different from CLAUDE.md — memory persists between conversations and is managed by Claude itself. CLAUDE.md is your explicit configuration; memory is Claude's accumulated understanding."

**Key point:** "Notice the pattern: CLAUDE.md, Rules, and Skills are all markdown files — but they load differently. Rules are always in context. Skills are on-demand. CLAUDE.md is always in context. This distinction matters, and we'll see it live in the demo."

**Transition:** "Let's go deeper on MCP since it directly impacts your context budget."

---

### Slide 6: MCP — Model Context Protocol

**Duration:** 3–4 minutes

**What to say:**
"MCP is how Claude Code connects to the outside world. The analogy I like: USB-C for AI. Before MCP, every AI tool had its own way of connecting to databases, APIs, services. MCP standardizes this."

Walk through the three-column flow:
1. "Claude Code acts as the client — it discovers what tools are available from connected MCP servers."
2. "The MCP server is a lightweight process that translates between Claude and the external service."
3. "The external service is whatever you're connecting to — Postgres, GitHub, Jira, a custom API."

**Emphasize the warning box:** "Here's the catch that connects back to our context discussion. Each MCP server's tool definitions consume tokens just by being registered. You don't even have to use them. If you have 5 MCP servers connected, their tool definitions alone can eat 66% of your context window. The discipline is: install only what you need, and toggle servers off when you're not actively using them."

**Transition:** "Another context-management tool is delegation to subagents."

---

### Slide 7: Subagents & Delegation

**Duration:** 3–4 minutes

**What to say:**
"When Claude needs to do something complex, it has two choices. 'Do it myself' means using the main context window — every tool result, every file read stays in context and fills it up. 'Delegate' means spawning a subagent — a completely separate Claude session with its own context window."

Walk through the two columns. The key insight is the last row of each:
- Execute: "Best for small, focused tasks."
- Delegate: "Best for large tasks, parallel work, specialists."

**Key point:** "The magic is that only a summary returns to the parent. If a subagent reads 50 files and writes a report, the parent doesn't get those 50 file reads in its context — it gets a paragraph summary. This is how you do complex work without destroying your context."

**Bottom note:** "You can define reusable specialists in .claude/agents/ — a code-reviewer, a security auditor, each with their own instructions and role."

**Mention:** "We'll go deeper into subagent orchestration in the advanced session."

**Transition:** "Now let's look at the commands you'll use every day."

---

### Slide 8: Slash Commands & Daily Workflow

**Duration:** 3–4 minutes

**What to say:**
Go through each command. These are the daily workflow tools.

- **/compact:** "Your most-used context management command. When context is getting heavy — you feel responses getting slower or less accurate — run /compact. Claude summarizes the conversation, frees context, and continues from the summary."

- **/clear:** "The nuclear option. Wipes everything and starts fresh. More aggressive than /compact because nothing is carried over. Use when you're changing tasks entirely or context is beyond recovery."

- **/teleport:** "Transfers your session between surfaces. The use case: you're on the train, reviewing code on your phone through claude.ai/code. You get to the office, open your terminal, run /teleport, and your session continues locally with full filesystem access."

- **/init:** "Run this when you start using Claude Code in a new project. Claude analyzes your codebase and generates a CLAUDE.md with conventions, tech stack, and routing. It's a starting point — you'll refine it."

- **/review:** "Triggers a code review workflow. Claude examines recent changes against your project rules."

- **/your-command:** "The extensibility point. Any markdown file in .claude/commands/ becomes a slash command. Your team's deploy checklist, your release workflow, your documentation generator — all invokable by name."

**Transition:** "Let's see some of this in action."

---

### 🎯 DEMO 1: Install & First Run (~5 minutes)

**When:** After slide 8

**What to show:**

1. **Install:** `npm install -g @anthropic-ai/claude-code` (or show it already installed)
2. **Run:** `cd` into a real project and run `claude`
3. **Show the initial prompt** — point out how Claude picks up the project context
4. **Show `!` prefix:** Type `!ls` or `!git status` — explain this is direct shell passthrough, no Claude interpretation
5. **Show `Shift+Tab`:** Toggle through Normal → Auto-Accept → Plan Mode. Point out the indicator change.
6. **Show `/compact`** — if the session has some history, show the summarization. Otherwise, mention it and say you'll show it during the longer demo.

**What to say:** "Notice that Claude immediately knows the project structure. It read the directory, found the language, found the framework. If there's a CLAUDE.md, it loaded it. All of this is context — and it's already consuming tokens."

---

### Slide 9: The Claude Product Family

**Duration:** 3–4 minutes

Walk through the three columns briefly. The audience now has the vocabulary to understand the differences.

**Key insight to share:** "Notice the persistence row. Claude Chat has conversation history — that's it. Claude Code Web has project memory via CLAUDE.md. Claude Code CLI has the full stack: CLAUDE.md, hooks, skills, memory. This is why the CLI is the power tool."

**Don't linger here.** The comparison is a reference — people will come back to it later.

---

### Slide 10: Claude Code vs Cursor vs Windsurf

**Duration:** 3–4 minutes

**What to say:**
"This isn't about which tool is better — it's about understanding what each one actually is architecturally."

Highlight the key rows:
- **Agent mode:** "Core architecture vs bolt-on feature. This is the fundamental difference. Claude Code was built as an agent from day one. Cursor and Windsurf added agent features to what was originally an autocomplete tool."
- **Context control (highlighted row):** "This is the row I want you to pay attention to. With Claude Code, you have full autonomy over what enters your context — selective skill loading, MCP server toggles, /compact, /clear, rules vs skills separation. With Cursor, you have partial control, but it silently truncates context for cost and performance reasons. Developers have reported that Cursor's auto mode sometimes drops files from context without warning — you don't notice until the AI gives you wrong code. With Windsurf, everything in .windsurfrules loads on every single prompt. No selective loading, no toggles. You're trading control for simplicity. Given that we spent the first 15 minutes of this talk on why context is the #1 skill — this row is the competitive argument for Claude Code."
- **Extensibility:** "CLAUDE.md, hooks, MCP, subagents vs .cursorrules. Claude Code has a full configuration stack. Cursor has a single rules file."
- **Parallel work:** "Git worktrees with 3-100 agents vs single workspace. This is the scalability story."

**Read the bottom callout:** "Context autonomy matters. This ties directly back to everything we discussed about context degradation. If you can't control what goes into your context, you can't manage it. And if you can't manage it, quality degrades."

---

### Slide 11: Competitive Edge — Head to Head

**Duration:** 2–3 minutes

This is a visual summary of slide 10. Don't repeat the same information — just highlight what's new.

**Left column highlights:** "Terminal-native is underrated. Claude Code works over SSH, in tmux, on headless servers. You can have agents running on a remote build server."

**Right column:** "Be honest about the tradeoffs. Visual feedback and tab completion are real advantages. If you're making a quick variable rename, Cursor is faster."

---

### Slide 12: The Three Permission Modes

**Duration:** 3–4 minutes

**What to say:**
"Three modes, one keybinding. Shift+Tab toggles between them."

For each mode, give a concrete scenario:
- **Normal:** "You're working on a production codebase you don't fully understand yet. You want to see every file write before it happens."
- **Auto-Accept:** "You're building a greenfield project and you trust Claude's judgment. You don't want to approve every npm install and file creation."
- **Plan Mode:** "You're designing the architecture for a new feature. You want Claude to read the codebase and propose an approach, but you don't want it touching anything yet."

**Bottom section — permissions:** Walk through quickly. The key surprise for most people is that file reads are usually auto-allowed, but git pushes always require explicit approval regardless of mode.

---

### Slide 13: settings.json — Configuration Guide

**Duration:** 3–4 minutes

**What to say:**
"Four files, strict precedence."

Focus on the practical:
- "The project-level file is what you commit to git — team-shared conventions."
- "The local file is your personal overrides — gitignored."
- "The managed file is for enterprise admins — it cannot be overridden."

**Show the code block:** Point out the key settings:
- `"model": "opus"` — "You can set the default model per-project."
- `permissions.allow` — "Whitelist specific commands. `Bash(npm run *)` means any npm run command is auto-approved."
- `permissions.deny` — "Blacklist specific reads. `Read(.env)` prevents Claude from reading your secrets."

**Pro tip:** "Arrays like permissions.allow merge across all scopes. So your user settings and project settings combine — you don't have to duplicate everything."

---

### Slide 14: Claude Model Comparison Guide

**Duration:** 2–3 minutes

This is a visual reference. Don't read every cell.

**Key points:**
- "Haiku is fast and cheap — use it for subagent tasks where you need high volume."
- "Sonnet is the workhorse — 90% of your daily coding work should use this."
- "Opus is for deep strategy — 1 million token context window, extended thinking, but slower and more expensive."

**Cost tip:** "A multi-model strategy can reduce costs by 40-60%. Use Haiku for subagents, Sonnet for main work, Opus for planning and architecture."

---

### Slide 15: CLAUDE.md — Persistent Project Context

**Duration:** 4–5 minutes

Walk through the infographic carefully. This is the most important configuration concept.

**Left side — What it IS:**
1. "A routing table — like an old telephone operator. It tells Claude where to find things. 'If you need auth docs, see @docs/auth.md.'"
2. "A constitution — your project's rules and conventions."
3. "An index — links to deeper documentation. Claude follows the links when it needs more detail."
4. "A verification contract — error handling patterns, function contracts. Claude checks its work against these."

**Right side — What it ISN'T:**
- "Not a personality profile. Don't write 'Be helpful and thorough.' Claude already is."
- "Not a manual. Don't dump your entire project documentation in here. Keep it under 250 lines."
- "Not a memory system. It doesn't store timeline or dynamic state."
- "Not enforceable by itself. Claude interprets CLAUDE.md — it can and will interpret loosely. For enforcement, you need hooks."

**Key technical detail:** "CLAUDE.md contents are wrapped in system-reminder tags and injected into every conversation. But here's the thing: recent chat and debugging output gets 10x more attention than CLAUDE.md content at the bottom of context. This is why short, focused CLAUDE.md files work better than long ones."

---

### Slide 16: Project Configuration Hierarchy

**Duration:** 3–4 minutes

Walk through the layers of the infographic from top to bottom:

1. "CLAUDE.md at the top — always in context. Lists available skills and commands."
2. ".claude/rules/ — modular laws, always loaded. Testing standards, security rules, style guides."
3. ".claude/skills/ — expert knowledge, loaded on demand. Database migration patterns, auth debugging guides."
4. ".claude/commands/ — workflows, loaded on /invoke. Deploy checklists, review workflows."
5. ".claude/agents/ — specialists with separate context. Code reviewer, security auditor."
6. "settings.json and hooks — the enforcement layer. Cannot be ignored."

**Read the bottom bar:** "CLAUDE.md is the constitution — advisory. Skills are SOPs — optional. Hooks are the police — mandatory."

**Transition:** "Let's see this hierarchy in practice."

---

### 🎯 DEMO 2: CLAUDE.md & Rules (~5 minutes)

**When:** After slide 16

**What to show:**

1. **Show CLAUDE.md:** Open a real project's CLAUDE.md. Walk through its structure — the routing table section, the conventions section, available skills.

2. **Run /init on a fresh project:** If you have a small project without CLAUDE.md, run `/init`. Show the audience what Claude generates — it reads the codebase and produces an initial constitution.

3. **The rules vs skills demo (key moment):**
   - Create a simple rule file in `.claude/rules/` — for example, `always-typescript-strict.md` with content like "Always use TypeScript strict mode. Never use `any` type."
   - Ask Claude to write a TypeScript function. Show that it follows the rule.
   - Now move that same file to `.claude/skills/` instead.
   - Ask Claude the same question. Show that Claude doesn't follow it — because skills are demand-loaded, and Claude doesn't know this skill exists unless CLAUDE.md references it.
   - **Say:** "This is the most common mistake people make. They put important rules in skills/ thinking they'll always be active. They won't. Rules go in rules/. Skills go in skills/ with a reference from CLAUDE.md."

**What to say after the demo:** "This hierarchy exists because of context. If everything was always loaded, you'd burn through your context window before the first conversation turn. The tiered loading is a deliberate design choice."

---

### Slide 17: Plan Mode — Think Before You Act

**Duration:** 3–4 minutes

**What to say:**
Walk through the CAN/CANNOT columns quickly — the audience already knows this from the permission modes slide.

**Focus on the hybrid workflow section at the bottom:**
"The community consensus, and what I'd recommend, is a four-tool workflow:
1. Claude Code CLI for the heavy lifting — multi-file changes, feature implementation, refactoring, debugging.
2. Your IDE (Cursor, VS Code, JetBrains) for quick edits and visual navigation.
3. Terminal for parallel worktree sessions — multiple Claude agents working simultaneously.
4. Web/Mobile for long-running tasks and reviewing on the go."

**Key insight:** "Plan Mode is especially powerful for the 'Plan Mode Leapfrog' technique you saw in the context window slide. When context is degraded, switch to Plan Mode, create a detailed plan, then /clear and start a fresh session with 'Continue from the plan.' Fresh context, peak performance restored."

---

### Slide 18: Hooks — The Enforcement Layer

**Duration:** 4–5 minutes

**What to say:**
"We've talked about CLAUDE.md as advisory — Claude interprets and follows it. We've talked about skills as optional — loaded on demand. Hooks are different. Hooks are mandatory. They're shell scripts that execute outside Claude's decision loop. Claude cannot choose to ignore them."

**Walk through the three hook types:**
- **PreToolUse:** "Runs before Claude takes an action. Can block the action entirely. Example: prevent writes to production files."
- **PostToolUse:** "Runs after Claude acts. Can trigger corrections. Example: run a linter after every file write."
- **Stop:** "Runs before Claude's final response. The last quality gate. Example: verify all tests pass before Claude declares it's done."

**Show the code example:**
"Here's a concrete PreToolUse hook. It matches on Write actions, checks if the tool input contains 'production', and blocks with exit code 1. This runs every time Claude tries to write any file. Claude has no say in this — the hook executes at the OS level."

**Point to the analogy on the right:**
"CLAUDE.md = Constitution — advisory, Claude interprets. Skills = SOPs — optional, loaded on demand. Hooks = Police — mandatory, cannot be bypassed."

**Mention:** "We'll do a live hooks demo in the advanced session. For today, just understand the hierarchy and know that hooks are where you put guardrails you can't afford to have ignored."

---

### Slide 19: Putting It All Together — How Claude Code Works

**Duration:** 4–5 minutes

**What to say:**
"Now you've seen all the individual pieces. This diagram shows how they connect in a single conversation turn."

Walk through each layer, referencing what the audience learned:
1. **Context layer:** "CLAUDE.md, Rules, Skills — you now know how each loads. Codebase, Memory, MCP Data, Conversation — all feeding into the context window you saw filling up in slide 2."
2. **Reasoning layer:** "Understand → Plan → Execute or Delegate. Plan Mode lives here — Shift+Tab. Subagents are the delegation path."
3. **Tools layer:** "Read, Search, Edit, Bash, Git, MCP Tools, Browser. Each tool use generates results that go back into context."
4. **Hooks layer:** "The Quality Gate. PreToolUse, PostToolUse, Stop. Cannot be bypassed."
5. **Verify layer:** "Run tests, check if they pass. If no, fix and retry. This is the autonomous loop."
6. **Output layer:** "Code changes, git commits, tests passing, pull requests, plans, explanations."

**Point to the right side — the feedback loop:**
"Everything feeds back. Changed files update the codebase. Learnings go to memory. Messages go to conversation history. And the whole thing repeats next turn — with a bigger context window."

**Point to the bottom bar:** "Context peaks at ~50%, then quality degrades. This is why the entire session today started with context management."

---

### Slide 20: Key Takeaways

**Duration:** 2–3 minutes

Don't just read the bullets. Add a sentence of color to each:

1. **Context management:** "If you remember one thing from today — start new sessions early, use /compact often, and watch your context usage."

2. **Agentic-first:** "Claude Code isn't an autocomplete tool. Let it plan, implement, test, and commit. Trust the agent loop."

3. **MCP costs context:** "Connect only the MCP servers you need. Toggle them off when you're done."

4. **Subagents keep context clean:** "For big tasks, let Claude delegate. A summary is better than 50 file reads in your context."

5. **CLAUDE.md + Hooks:** "Constitution for guidance, police for enforcement. You need both."

6. **Right mode for the right task:** "Normal when you're learning, Auto-Accept when you're in flow, Plan when you're designing."

7. **CLI + IDE:** "Use both. They're complementary tools, not competitors."

**Close:** "Questions? And for those interested — the advanced session will cover hooks in depth with live demos, subagent orchestration patterns, parallel worktree workflows, and CI/CD integration."

---

## Demo Preparation Checklist

Before the presentation, prepare the following:

### Demo 1: Install & First Run
- [ ] A project directory with some code (ideally a repo the audience would recognize or find interesting)
- [ ] Claude Code already installed (don't wait for npm install during the presentation — show the command but have it ready)
- [ ] Verify `claude` starts correctly in the project
- [ ] Practice the `!ls`, `Shift+Tab` toggle, and `/compact` flow

### Demo 2: CLAUDE.md & Rules
- [ ] A repo with an existing CLAUDE.md (show the real thing)
- [ ] A separate small project without CLAUDE.md (for the /init demo)
- [ ] Prepare the rule file content: a simple, testable rule like "Always use TypeScript strict mode. Never use `any` type."
- [ ] Test the rules/ vs skills/ demo end-to-end. Verify that:
  - A rule in `.claude/rules/` is followed
  - The same content moved to `.claude/skills/` is ignored (unless explicitly loaded)
- [ ] Have a fresh Claude session ready (so context is clean for the demo)

### General
- [ ] Test your terminal font size — make sure the audience can read from the back of the room
- [ ] Use a dark terminal theme with high contrast
- [ ] Close unnecessary tabs, notifications, and Slack
- [ ] Have a backup plan if the API is slow — screenshots of expected output

---

## Timing Summary

| Section | Slides | Duration |
|---------|--------|----------|
| Opening & Context (slides 1–3) | 3 | 10–14 min |
| What is Claude Code + Dictionary (slides 4–5) | 2 | 8–11 min |
| MCP, Subagents, Slash Commands (slides 6–8) | 3 | 9–12 min |
| **Demo 1: Install & First Run** | — | 5 min |
| Product Family & Comparisons (slides 9–11) | 3 | 8–11 min |
| Permissions & Config (slides 12–13) | 2 | 6–8 min |
| Models, CLAUDE.md, Project Hierarchy (slides 14–16) | 3 | 9–12 min |
| **Demo 2: CLAUDE.md & Rules** | — | 5 min |
| Plan Mode & Hooks (slides 17–18) | 2 | 7–10 min |
| Synthesis & Takeaways (slides 19–20) | 2 | 6–8 min |
| **Q&A** | — | 5–10 min |
| **Total** | **20 slides + 2 demos** | **65–75 min** |

---

## Common Audience Questions (and How to Answer Them)

**"How does Claude Code compare to GitHub Copilot?"**
Copilot is primarily an autocomplete tool with chat as a secondary feature. Claude Code is agentic — it plans, executes multi-step tasks, and works autonomously. They're not direct competitors; Copilot is closer to Cursor's autocomplete mode.

**"Can I use Claude Code with my existing Cursor setup?"**
Yes. Claude Code works WITH Cursor, not instead of it. You can have Claude Code running in a terminal panel inside Cursor. Use Cursor for quick edits and tab completion, Claude Code for multi-file changes and autonomous work.

**"How much does it cost?"**
Pro subscription: $20/month. Max subscription: $100-200/month for heavier usage. For API-based usage, costs depend on model: Haiku at $1/$5 per million tokens (input/output), Sonnet at $3/$15, Opus at $5/$25.

**"Is my code sent to Anthropic's servers?"**
Yes, the code Claude reads is sent to the API for processing. It's not used for training. For enterprise concerns, point to Anthropic's data retention policies and the enterprise tier options.

**"What about security? Can Claude access my .env files?"**
By default, yes — Claude can read any file in your project. Use `permissions.deny` in settings.json to block specific files: `"deny": ["Read(.env)", "Read(*.secret)"]`. For enterprise, managed settings can enforce this organization-wide.

**"When should I start a new session vs use /compact?"**
Use /compact when you're continuing the same task but context is heavy. Use /clear (or start a new session) when you're switching tasks entirely. The Plan Mode Leapfrog technique — plan the remaining work, /clear, continue from the plan — gives you the best of both.

---

## What This Session Intentionally Defers

Mention these as "coming in the advanced session" to set expectations:

- **Hooks deep-dive:** Concrete PreToolUse/PostToolUse/Stop examples, live demo of hooks blocking actions
- **Subagent orchestration:** When to delegate, how to configure specialist agents, the bypassPermissions pattern
- **Parallel worktrees:** Setting up 3-5+ agents working simultaneously on different branches
- **CI/CD integration:** GitHub Actions with Claude Code, GitLab CI/CD, Slack bot integration
- **Custom MCP server development:** Building your own MCP servers for internal tools
- **The Ralph Loop:** Autonomous bug-fixing workflow technique
- **Advanced CLAUDE.md patterns:** Progressive disclosure, conditional references, verification contracts

---

## Research References & Sources

Keep these handy for Q&A or if someone challenges the context degradation claims:

**Academic Papers:**
- **"Lost in the Middle" (Stanford, 2024)** — https://arxiv.org/abs/2307.03172 — LLMs exhibit a U-shaped attention curve. Performance is highest for information at the beginning (primacy) and end (recency), and significantly degrades for information in the middle. Holds even for models explicitly designed for long context.
- **"Lost in the Middle: An Emergent Property" (2025)** — https://openreview.net/forum?id=XSHP62BCXN — The recency effect mirrors human short-term memory. The primacy effect is induced by autoregressive attention sinks. Larger models reduce but don't eliminate the U-curve.
- **Stanford ACE Framework (2025)** — https://arxiv.org/abs/2510.04618 — Shows "context collapse" when LLMs iteratively rewrite their own context (like auto-compaction).

**Community Sources (Reddit):**
- u/Main_Payment_6430: "Models have Recency Bias. If you have a perfect CLAUDE.md at the top, but 4,000 lines of debugging chat at the bottom, the model will pay 10x more attention to the debugging chat." S-tier for first ~45 min, then degrades.
- u/g3_SpaceTeam: Named three specific problems — context ordering, context saturation, context decay.
- u/helk1d (818 upvotes): "lost in the middle dumb zone" when context gets big.
- Context compaction KB: "Instructions buried at position 400K+ get missed"; "Attention is a much more limited resource than context."

**Context Engineering (the shift from prompt engineering):**
- Tobi Lütke (Shopify CEO) and Andrej Karpathy (former OpenAI) endorsed the term in June 2025
- Anthropic blog post on context engineering (September 2025) formalized it for agent development
- Key distinction: "Prompt engineering is what you do inside the context window. Context engineering is how you decide what fills the window."
