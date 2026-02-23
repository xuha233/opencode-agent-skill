---
name: opencode-agent
description: "OpenClaw Agent control bridge for OpenCode CLI. Activates on coding tasks: implementation, review, refactoring, analysis. Uses non-interactive mode with session resume."
metadata: {"openclaw":{"emoji":"🚀","requires":{"bins":["gh"],"anyBins":["opencode"],"env":[]}}}
---

# OpenCode Agent - Control Bridge for OpenCode CLI

**Purpose**: This skill provides the interface for OpenClaw Agent to control OpenCode CLI for coding tasks.

---

## Activation Conditions

Activate this skill when user request matches **any** coding task pattern:

### Implementation Tasks
Keywords: `implement`, `build`, `create`, `add function`, `write code`, `generate`
Examples:
- "Implement user registration"
- "Create a REST API endpoint"
- "Add error handling"

### Code Review Tasks
Keywords: `review`, `audit`, `check`, `analyze code`
Examples:
- "Review this code for bugs"
- "Security audit of auth module"
- "Check code quality"

### Refactoring Tasks
Keywords: `refactor`, `improve`, `optimize`, `clean up`
Examples:
- "Refactor this function"
- "Optimize database queries"
- "Improve code structure"

### Explanation Tasks
Keywords: `explain`, `what does`, `how does`, `understand`
Examples:
- "Explain how authentication works"
- "What does this function do?"
- "Analyze the architecture"

### Bug Fix Tasks
Keywords: `fix bug`, `debug`, `resolve error`, `investigate issue`
Examples:
- "Fix the login bug"
- "Debug this error"
- "Investigate performance issue"

---

## Command Decision Logic

```
User Request → Intent Recognition → Context Check → Command Selection

┌─────────────────────────────────────────────────────────────────┐
│ DECISION FLOW                                                   │
└─────────────────────────────────────────────────────────────────┘

Step 1: Intent Recognition
├─ Is coding task? ──NO──→ Use other skill
└─ YES ───────────────────→ Continue to Step 2

Step 2: Task Type
├─ Implementation? ──→ Step 3a
├─ Review? ──────────→ Step 3b
├─ Refactor? ─────────→ Step 3c
├─ Explain? ──────────→ Step 3d
├─ Bug Fix? ──────────→ Step 3e
└─ Other ─────────────→ Step 3a (default)

Step 3a: Implementation Context
├─ New project / unrelated task? ──→ opencode run "prompt"
├─ Continue existing task? ────────→ Check Step 4
└─ Risky change (large refactor)? ─→ opencode run --continue --fork "prompt"

Step 3b: Code Review Context
├─ Single file? ───────────────────→ opencode run --file file.ts "review"
├─ Multiple files? ─────────────────→ opencode run --file f1.ts --file f2.ts "review"
├─ After implementation? ────────────→ opencode run --continue "review changes"
└─ Specific session? ───────────────→ opencode run --session <id> "review files"

Step 3c: Refactoring Context
├─ Small change (safe)? ────────────→ opencode run --continue "refactor X"
├─ Large/Ambiguous change? ─────────→ opencode run --continue --fork "try refactoring"
└─ Multiple approaches? ──────────────→ Fork each approach separately

Step 3d: Explanation Context
├─ Understand codebase? ─────────────→ opencode run "explain architecture"
├─ Specific file? ───────────────────→ opencode run --file file.ts "explain this"
└─ Session recall? ──────────────────→ Check Step 4

Step 3e: Bug Fix Context
├─ Error log provided? ──────────────→ opencode run --file error.log "investigate"
├─ Context files available? ──────────→ opencode run --file src/ts "fix bug"
└─ Multiple fix attempts? ────────────→ opencode run --continue --fork "try fix B"

Step 4: Session Context
├─ User says "continue"? ─────────────→ opencode run --continue "prompt"
├─ User provides session ID? ─────────→ opencode run --session <id> "prompt"
├─ Latest session relevant? ─────────→ Check via opencode session list
└─ Need clean slate? ─────────────────→ opencode run "prompt"
```

---

## Core Commands

### Primary: `opencode run`

**Syntax:**
```bash
opencode run "prompt" [OPTIONS]
```

**Key Flags for Agent:**

| Flag | When to Use | Context |
|------|------------|---------|
| `--continue, -c` | Task continuation | User says "continue", "add", "more" |
| `--session, -s <id>` | Specific session recovery | User provides ID or references old task |
| `--fork` | Alternative approaches | Risky changes, multiple implementations |
| `--file, -f <path>` | Context attachment | References files, multi-file review, directory |
| `--model, -m <provider/model>` | Model selection | Deep reasoning (Claude Sonnet), fast/cheap (DeepSeek) |
| `--format json` | Programmatic output | Need to parse events, extract structured data |

**Recommended Patterns:**

1. **New Implementation:**
   ```bash
   opencode run "Implement user registration with password hashing and JWT tokens"
   ```

2. **Continue Task:**
   ```bash
   opencode run --continue "Add email verification feature"
   ```

3. **Fork for Alternative:**
   ```bash
   opencode run --continue --fork "Try implementing with OAuth2 instead"
   ```

4. **With Context Files:**
   ```bash
   opencode run "Review authentication flow for security issues" --file src/auth.ts --file src/session.ts
   ```

5. **Specific Model:**
   ```bash
   opencode run --model anthropic/claude-sonnet-4-20250514 "Implement complex algorithm"
   ```

### Session Management

**List Sessions:**
```bash
opencode session list --format json
```
Returns JSON: `[{id, title, updated, created, projectId, directory}]`

**Check Latest:**
```bash
opencode session list | head -2
```

**Delete Session:**
```bash
opencode session delete <session-id>
```

### Statistics

**Get Current Usage:**
```bash
opencode stats --format json
```

**Last N Days:**
```bash
opencode stats --days 7 --format json
```

### Export/Import

**Export Session:**
```bash
opencode export <session-id> > backup.json
```

**Import Session:**
```bash
opencode import <file.json or URL>
```

---

## Context Awareness Rules

### Rule 1: Is This a Coding Task?

**Positive Indicators ✅:**
- "implement", "build", "create", "write code", "generate"
- "review", "audit", "check", "analyze code"
- "refactor", "improve", "optimize"
- "fix bug", "debug", "resolve error"
- "explain", "what does", "how does"
- Mentions specific programming concepts (API, database, function, class)

**Negative Indicators ❌:**
- "write documentation" → Use file editing tools
- "create blog post" → Use writing skills
- "summarize this article" → Use reading skills
- "translate to Chinese" → Use translation skills (not code)

### Rule 2: Should I Add Files?

**Add Files When:**
- User explicitly mentions files: "Review file.ts"
- Multiple related files: "Review auth module" → Attach all auth files
- Directory reference: "Analyze this API" → Attach entire dir/
- Error logs provided: Always attach
- Code snippets in conversation: Create temp file, then attach

**File Attachment Order:**
1. User-specified files (highest priority)
2. Related context files (same directory, naming patterns)
3. Error logs / stack traces
4. Test files (if testing mentioned)

**Maximum Rule:** Attach up to 10 files at once. For more, ask user.

### Rule 3: Continue vs New Session

**Use `--continue` When:**
- User says "continue", "add", "more", "next"
- Follow-up to previous message
- Implementing multi-phase plan
- Refactoring based on review findings

**Use `--session <id>` When:**
- User provides specific session ID
- Reference to earlier task (by time/description)
- Switching between parallel work streams

**Start New Session When:**
- Completely unrelated task
- Fresh start requested ("start over", "new task")
- After major context break

### Rule 4: Fork or Modify?

**Fork When:**
- Risky/large refactoring
- Trying multiple approaches
- Experimental changes
- Unsure of correct solution

**Modify When:**
- Small, safe changes
- Follow-up to clear plan
- Bug fixes to existing code
- Adding simple features

---

## Workflows

### Standard Implementation Flow

```bash
# Phase 1: Explore (Read-only)
opencode run "Explain the current authentication implementation" --file src/auth.ts

# Phase 2: Plan
opencode run --continue "Create a detailed plan for adding password reset feature"

# Phase 3: Implement
opencode run --continue "Implement the password reset feature"

# Phase 4: Test
opencode run --continue "Write tests for password reset"

# Phase 5: Review
opencode run --continue "Review the implementation for security issues"

# Phase 6: Commit
git checkout -b feat/password-reset
git add .
git commit -m "feat: add password reset"
gh pr create -t "feat: add password reset" -b "..."
```

### Code Review Flow

```bash
# Attach all relevant files
opencode run -f src/auth/login.ts -f src/auth/session.ts -f src/api/auth.ts "Review authentication module for security issues, error handling, and code quality"

# Fix findings
opencode run --continue "Address the review findings:
1. Fix SQL injection risk in login query
2. Add input validation for email/username
3. Handle edge case of expired JWT tokens"

# Re-review
opencode run --continue "Re-review after fixes"
```

### Bug Fix Flow

```bash
# Investigate
opencode run -f logs/error.log -f src/auth/login.ts "Analyze this error: User reports 'Login fails with valid credentials'"
opencode run --continue "What's the root cause?"

# Fix
opencode run --continue "Implement the fix for root cause identified"

# Verify
opencode run --continue "Verify the fix resolves the issue"
```

### A/B Testing with Forks

```bash
# Approach A
opencode run "Implement user authentication with JWT tokens"

# Fork for Approach B (OAuth2)
opencode run --continue --fork "Replace JWT with OAuth2 flow"

# Fork for Approach C (Session-based auth)
opencode run -s <fork-b-id> --fork "Try session-based authentication"

# Compare
opencode run -s <original-id> "List pros/cons of JWT approach"
opencode run -s <fork-b-id> "List pros/cons of OAuth2 approach"
```

---

## Output Interpretation

### Default Output (Plain Text)
**Format:** Human-readable text
**Agent Action:** Present to user, summarize if long

### JSON Output (`--format json`)
**Format:** Event stream
**Event Types:**
- `message.updated` - Agent/model info
- `message.part.updated` - Text/tool/reasoning
- `step-start`, `step-finish` - Tool execution
- `session.status` - Session state (idle/busy/error)

**Agent Action:** Parse and extract:
- Text content for user
- Tool calls for logging
- Errors for troubleshooting

### Session List Output
**Table Format:** Human-readable
**Agent Action:** Show latest 5-10 sessions

**JSON Format:** Structured data
**Agent Action:** Parse for automation, date filtering

---

## Error Handling

### Command Fails
1. **Check OpenCode installed:**
   ```bash
   command -v opencode && opencode --version
   ```
2. **Check configured:**
   ```bash
   opencode auth list
   ```
3. **Check session exists:**
   ```bash
   opencode session list
   ```
4. **Retry with clearer prompt**

### Timeout
- **Message:** "OpenCode is still processing, waiting..."
- **Action:** Wait 60s, then check progress again
- **Fallback:** Ask user if they want to continue waiting

### Rate Limits
- **Message:** "Rate limit reached, retrying in 60s..."
- **Action:** Wait 60s, then retry command

### API Errors
- **Message:** "API error: [error message]"
- **Action:** Check auth credentials, or try different model

---

## Examples

### Example 1: Simple Implementation

**User:** "Implement a user logout function"

**Agent Reasoning:**
1. Intent: Implementation ✅
2. Context: Existing auth module
3. Command: `opencode run "Implement user logout with JWT token invalidation and session management"`

**Agent Action:**
```bash
opencode run "Implement user logout function:
- Clear JWT token
- Invalidate session
- Add error handling
- Return proper response" --file src/auth/session.ts
```

### Example 2: Code Review

**User:** "Review the login code"

**Agent Reasoning:**
1. Intent: Code Review ✅
2. Context: Auth module (search for login files)
3. Files: src/auth/login.ts, src/auth/session.ts

**Agent Action:**
```bash
opencode run "Review authentication login and session management for security issues, error handling, code quality" --file src/auth/login.ts --file src/auth/session.ts
```

### Example 3: Bug Fix

**User:** "Login fails with valid credentials"

**Agent Reasoning:**
1. Intent: Bug Fix ✅
2. Context: Error log + login code

**Agent Action:**
```bash
opencode run "Investigate: User reports 'Login fails with valid credentials'. Find root cause and propose fix" --file logs/error.log --file src/auth/login.ts
```

### Example 4: Multi-Task

**User:** "Implement registration, then add email verification, then test"

**Agent Reasoning:**
1. Intent: Implementation (multi-phase)
2. Use `--continue` for phases

**Agent Action:**
```bash
# Phase 1
opencode run "Implement user registration with password hashing"

# Phase 2
opencode run --continue "Add email verification after registration"

# Phase 3
opencode run --continue "Write tests for registration + verification workflow"
```

---

## Configuration

### OpenCode Config Files
- **Main:** `~/.local/share/opencode/config.json`
- **Auth:** `~/.local/share/opencode/auth.json`

### Agent Behavior Recommendations

1. **Always use `--continue`** for related tasks
2. **Fork before risky changes**
3. **Prefer `--format json`** for programmatic parsing
4. **Check `opencode session list`** before new session
5. **Report token usage** via `opencode stats` periodically
6. **Add files liberally** — more context = better outcome
7. **Model selection:** DeepSeek for fast/cheap, Claude Sonnet for deep reasoning

---

## Quick Reference

| Scenario | Command |
|----------|---------|
| New task | `opencode run "prompt"` |
| Continue task | `opencode run --continue "prompt"` |
| Fork alternative | `opencode run --continue --fork "prompt"` |
| Review single file | `opencode run --file file.ts "review"` |
| Review multiple files | `opencode run -f f1.ts -f f2.ts "review"` |
| Bug investigation | `opencode run -f error.log "investigate"` |
| Deep reasoning | `opencode run -m anthropic/claude-sonnet-4 "task"` |
| List sessions | `opencode session list --format json` |
| Get stats | `opencode stats --format json` |
| Export session | `opencode export <id> > backup.json` |
