# OpenCode Agent - OpenClaw Control Bridge

🚀 **Control Bridge for OpenClaw Agent to manage OpenCode CLI**

---

## What is this?

OpenCode Agent is an OpenClaw Skill that enables OpenClaw Agent to control **OpenCode CLI** — a free, open-source AI coding assistant (an alternative to Claude Code).

### Core Design

- **External Perspective (User)**: A tool that helps you write code, review code, and fix bugs
- **Internal Perspective (Agent)**: "Control Bridge for OpenCode CLI" — decides when to call, how to call, parse output, handle errors

---

## Features

### For Users (What you can do)

- ✍️ **Write Code**: "Implement user registration", "Add API error handling"
- 👀 **Code Review**: "Review code security", "Check code quality"
- 🔧 **Refactor**: "Optimize performance", "Improve code structure"
- 💡 **Explain Code**: "Explain what this file does", "Analyze architecture"
- 🐛 **Fix Bugs**: "Fix login failure", "Debug this error"

### For Agents (What the Agent can do)

- 🎯 **Intent Recognition**: Automatically detect coding tasks
- 🤔 **Context Awareness**: Gather relevant files, check session history
- 🧠 **Command Decision**: Select correct commands and flags (`--continue`, `--fork`, `--file`)
- 📤 **Output Parsing**: Parse OpenCode responses and extract key information
- 🔄 **Error Recovery**: Automatically handle failures, retry, or fallback

---

## Install

### Step 1: Install OpenCode CLI

Choose one method:

```bash
# NPM (cross-platform)
npm install -g opencode-ai

# Homebrew (macOS/Linux)
brew install anomalyco/tap/opencode

# Chocolatey (Windows)
choco install opencode

# Scoop (Windows)
scoop install opencode
```

### Step 2: Configure LLM Provider

```bash
# Login to AI provider (opens browser)
opencode auth login

# Check configured providers
opencode auth list
```

### Step 3: Install Skill in OpenClaw

```bash
# Clone to OpenClaw skills directory
cd ~/.openclaw/skills
git clone https://github.com/xuha233/opencode-agent-skill.git opencode-agent
```

Or manually copy to `~/.openclaw/skills/opencode-agent/`.

---

## Quick Start

### For Users

Simply ask OpenClaw Agent to code:

> "Implement user login functionality"

The Agent will:
1. Recognize this as a coding task ✅
2. Gather relevant files (`src/auth/login.ts`)
3. Call OpenCode: `opencode run "Implement user login" --file src/auth/login.ts`
4. Parse output and present to you
5. Use `--continue` if more work needed

### Example Conversations

```
You: Implement user registration

Agent: ✅ Implementing user registration...
[OpenCode generates code...]

You: Add email verification

Agent: ✅ Adding email verification...
[OpenCode continues implementing...]

You: Review the code

Agent: ✅ Reviewing code...
[OpenCode reviews security, quality...]
```

---

## Control Bridge - Agent Internal Guide

> Internal documentation for OpenClaw Agent

### Activation Conditions

Activate skill when user request matches these patterns:

1. **Implementation**: "implement", "build", "create", "write code", "generate"
2. **Code Review**: "review", "audit", "check", "analyze code"
3. **Refactoring**: "refactor", "improve", "optimize", "clean up"
4. **Explanation**: "explain", "what does", "how does", "understand"
5. **Bug Fix**: "fix bug", "debug", "resolve error", "investigate issue"

### Command Decision Logic

```
User Request → Intent Recognition → Context Check → Command Selection

Example:
User: "Add error handling"
→ Intent: Implementation ✅
→ Context: Continue task
→ Command: opencode run --continue "Add error handling"
```

### Core Commands for Agent

| Scenarios | Command |
|-----------|---------|
| New task | `opencode run "prompt"` |
| Continue task | `opencode run --continue "prompt"` |
| Fork alternative | `opencode run --continue --fork "prompt"` |
| Review single file | `opencode run --file file.ts "review"` |
| Deep reasoning | `opencode run -m claude-sonnet-4 "prompt"` |

### Integration Flow

```
1. Intent Recognition → Is this a coding task?
2. Gather Context → Relevant files, session history
3. Construct Command → Select command + flags
4. Execute Command → opencode run "prompt" --file ...
5. Parse Output → Extract code, suggestions, errors
6. Present to User → Summary, code blocks, key points
7. Error Handling → Retry, fallback, user feedback
```

**For details**: `references/AGENTS_GUIDE.md`

---

## Architecture

### Documentation Layers

```
┌─────────────────────────────────────────────────────────┐
│ SKILL.md                                                 │
├─────────────────────────────────────────────────────────┤
│ • Activation conditions (when to activate)               │
│ • Command decision logic (which command to use)          │
│ • Context awareness rules (how to gather context)       │
│ • Error handling (what to do when errors occur)         │
└─────────────────────────────────────────────────────────┘
         ↓
┌─────────────────────────────────────────────────────────┐
│ CLAUDE.md                                                │
├─────────────────────────────────────────────────────────┤
│ • Workflow standards (workflow patterns)                 │
│ • Code review patterns (review patterns)                 │
│ • Dev persona (developer character)                      │
└─────────────────────────────────────────────────────────┘
         ↓
┌─────────────────────────────────────────────────────────┐
│ references/                                              │
├─────────────────────────────────────────────────────────┤
│ • AGENTS_GUIDE.md (Agent control guide)                 │
│ • COMMANDS.md (Detailed command reference)             │
│ • WORKFLOW.md (Workflow patterns)                       │
│ • STANDARDS.md (Coding standards)                       │
│ • quick-reference.md (Command cheat sheet)              │
└─────────────────────────────────────────────────────────┘
```

---

## Configuration

### OpenCode Config Files

| File | Location | Purpose |
|------|----------|---------|
| `config.json` | `~/.local/share/opencode/` | Main configuration |
| `auth.json` | `~/.local/share/opencode/` | Provider credentials |

### Recommended Models

| Model | Best For | Cost |
|-------|----------|------|
| `deepseek/deepseek-chat` | Fast tasks, simple queries | Low |
| `anthropic/claude-sonnet-4-20250514` | Balanced tasks | Medium |
| `anthropic/claude-opus-4-20250514` | Deep reasoning, architecture | High |

---

## Examples

### Example 1: Simple Implementation

**You**: "Implement user logout functionality"

**Agent**:
1. ✅ Intent recognition: Implementation
2. 📁 File context: `src/auth/session.ts`
3. 🔧 Command:
   ```bash
   opencode run "Implement user logout with JWT invalidation and session management" --file src/auth/session.ts
   ```
4. 📤 Parse output: Extract code
5. 💬 Present to user:
   ```
   ✅ Implemented user logout

   Files created/modified:
   - src/auth/logout.ts (new)
   - src/api/auth/logout.ts (new)
   - src/auth/session.ts (modified: added invalidateSession)

   Main functionality:
   - Clear JWT token
   - Invalidate session
   - Return success response
   ```

### Example 2: Code Review

**You**: "Review the login code"

**Agent**:
1. ✅ Intent recognition: Code Review
2. 📁 File context: `src/auth/login.ts`, `src/auth/session.ts`
3. 🔧 Command:
   ```bash
   opencode run "Review authentication login and session management for security, quality, best practices" --file src/auth/login.ts --file src/auth/session.ts
   ```
4. 📤 Parse output: Extract review findings
5. 💬 Present to user:
   ```
   ✅ Code review completed

   🔴 Critical Issues:
   1. SQL injection risk (src/auth/login.ts:45) - Query not parameterized
   2. JWT token not checked for expiration

   🟡 Suggested Improvements:
   1. Add input validation
   2. Add failure retry limit
   3. Improve error messages

   Recommend prioritizing 🔴 critical issues.

   Should I fix these issues?
   ```

### Example 3: Bug Fix

**You**: "Login always fails"

**Agent**:
1. ✅ Intent recognition: Bug Fix
2. 📁 File context: `logs/error.log`, `src/auth/login.ts`
3. 🔧 Command:
   ```bash
   opencode run "Investigate: User reports login always fails. Find root cause and propose fix" --file logs/error.log --file src/auth/login.ts
   ```
4. 📤 Parse output: Extract root cause
5. 💬 Present to user:
   ```
   🔍 Problem Analysis

   Root Cause:
   Password comparison logic error (src/auth/login.ts:78)
   ❌ Wrong: `if (password !== user.password)`
   ✅ Should be: `bcrypt.compare(password, user.password)`

   Should I fix this?
   ```

### Example 4: Multi-Phase Workflow

**You**: "Implement registration, then add email verification, then test"

**Agent**:
```bash
# Phase 1
opencode run "Implement user registration with password hashing"

# Phase 2 (continue)
opencode run --continue "Add email verification after registration"

# Phase 3 (continue)
opencode run --continue "Write tests for registration + verification"
```

---

## Comparison: Claude Code vs OpenCode

| Feature | Claude Code | OpenCode | Notes |
|---------|------------|----------|-------|
| **Open Source** | ❌ No | ✅ Yes | OpenCode fully open source |
| **Price** | 💰 Subscription | ✅ Pay-as-you-go | Pay per use |
| **LLM Providers** | Anthropic only | ✅ 15+ providers | Supports GPT, DeepSeek, Claude, etc. |
| **Session Resume** | `--resume` | `--continue` / `-c` | Same semantics |
| **Fork Sessions** | ❌ Not supported | ✅ `--fork` | OpenCode exclusive |
| **Server Mode** | ❌ Not supported | ✅ `opencode serve` | Persistent backend, faster |
| **GitHub Integration** | ✅ Yes | ✅ Yes | Both support |

**Cost Comparison** (estimated):

| Task | Claude Code ($10/mo) | OpenCode (Pay-as-you-go) |
|------|---------------------|-------------------------|
| 100 prompts (simple) | $10 | ≈ $2-5 |
| 50 prompts (complex) | $10 | ≈ $8-15 |

**Recommendation**:
- Frequent use → OpenCode more cost-effective
- Occasional use → Claude Code simpler

---

## Development

### Based on OpenCode Source Code

This skill is based on actual OpenCode CLI source code analysis:

- `packages/opencode/src/cli/cmd/run.ts`
- `packages/opencode/src/cli/cmd/session.ts`
- `packages/opencode/src/cli/cmd/stats.ts`
- `packages/opencode/src/cli/cmd/export.ts`
- `packages/opencode/src/cli/cmd/import.ts`

All commands and flags validated against source implementation.

### Version History

- **v2.0.0** — Control Bridge redesign (current)
  - SKILL.md: Agent-centric tool guide
  - references/AGENTS_GUIDE.md: Detailed integration guide
  - README.md: User + Agent perspectives

- **v1.0.0** — Initial release
  - User-oriented documentation

---

## Troubleshooting

### OpenCode not found

```bash
# Check version
opencode --version

# Reinstall
npm install -g opencode-ai
```

### Auth errors

```bash
# Check credentials
opencode auth list

# Re-login
opencode auth login
```

### File not found

- Agent will search workspace
- If not found, will ask user for correct path

### Timeout

- Complex tasks may take time
- Agent will wait and report progress
- Can cancel with user command

---

## Documentation

### For Users

- **This README** — Quick start, examples, comparison

### For Agents

- **SKILL.md** — Activation conditions, command logic
- **CLAUDE.md** — Workflow standards, dev persona
- **references/AGENTS_GUIDE.md** — Detailed integration guide
- **references/COMMANDS.md** — Command reference
- **references/WORKFLOW.md** — Workflow patterns
- **references/STANDARDS.md** — Coding standards
- **references/quick-reference.md** — Command cheat sheet

---

## License

MIT

---

## Links

- 📦 **GitHub**: https://github.com/xuha233/opencode-agent-skill
- 📖 **OpenCode Docs**: https://opencode.ai/docs/
- 🔧 **OpenCode CLI Ref**: https://opencode.ai/docs/cli
- 💻 **OpenCode GitHub**: https://github.com/anomalyco/opencode
- 🔗 **Model Hub**: https://models.dev/
- 🐛 **Issue Tracker**: https://github.com/anomalyco/opencode/issues

---

## Credits

- **OpenCode CLI**: https://github.com/anomalyco/opencode
- **Design Philosophy**: Control bridge pattern — Agent → CLI → Execution
- **Documentation**: Based on actual source code analysis (v1.2.10)

---

## Feedback

Issues, feature requests, contributions: https://github.com/xuha233/opencode-agent-skill/issues

---

**🚀 Happy coding with OpenClaw + OpenCode!**

---

## Author & Maintainer

**Author**: 言午间

**Contact**: 3537183821@qq.com
