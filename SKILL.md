---
name: opencode-agent
description: "Open source coding agent using OpenCode CLI for implementation and reviews. Primary mode: direct CLI with session resume. Activates dev persona for pragmatic, experienced developer guidance."
metadata: {"openclaw":{"emoji":"🚀","requires":{"bins":["gh"],"anyBins":["opencode"],"env":[]}}}
---

# OpenCode Agent Skill 🚀

Open source coding agent using OpenCode CLI. Primary mode: direct CLI execution with session resume support.

## When to Use

Trigger this skill when the user wants:
- Code review, standards review, or architecture review
- Implementation or refactoring
- GitHub workflows, commits, and PRs

## Primary Mode: Direct CLI (Non-Interactive)

OpenCode CLI supports non-interactive execution:

```bash
# Prompt execution
opencode run "Implement feature X"
opencode run "Explain this code"

# Resume last session (continues most recent root session)
opencode run --continue
opencode run -c "Fix the review findings"

# Resume specific session
opencode run --session <session-id>
opencode run -s <session-id> "Add error handling"

# Fork before continuing (creates new session with '(fork #N)' suffix)
opencode run --continue --fork
opencode run -s <id> --fork "Try a different approach"
```

## Core Commands

### RUN - Primary execution command

```bash
# Syntax
opencode run [message..] [OPTIONS]
opencode run --command <name> [message..] [OPTIONS]

# Positional Arguments
message..    Text messages to send (array)

# Options
--command    Predefined command (message used as args)
--continue, -c     Continue the last session (most recent root session)
--session, -s <id> Continue specific session ID
--fork       Fork session before continuing (requires --continue or --session)
--share      Auto-share session
--model, -m <provider/model>  LLM model to use
--agent <name>      Select agent
--format <default|json>  Output format (default: formatted, json: raw events)
--file, -f <path>    Attach file(s) to message (array)
--title <string>    Session title (uses truncated prompt if empty)
--attach <url>       Attach to running server (e.g., http://localhost:4096)
--dir <path>         Directory to run in (remote path if attaching)
--port <port>        Local server port (random if not specified)
--variant <string>   Model variant (e.g., high, max, minimal)
--thinking          Show thinking blocks (default: false)
```

**Session Selection Logic**:
- `--continue`: Finds most recent session where `parent_id IS NULL` (root sessions only)
- `--session <id>`: Direct session ID lookup
- Neither: Creates new session with title from `--title` or truncated prompt

**Fork Behavior**:
- Creates new session with parent ID set to original session
- Title suffix: `"(fork #N)"` where N is fork count
- Preserves message history from parent

### SESSION - Session management

```bash
# List sessions
opencode session list
opencode session list --max-count 10
opencode session list --max-count 20 --format json

# Delete session
opencode session delete <session-id>
```

**List Options**:
- `--max-count, -n <number>` - Limit to N most recent sessions
- `--format <table|json>` - Output format (default: table)

**Table Output Fields**:
- Session ID (first 8 chars for display)
- Title (truncated if needed)
- Updated (today's time or full datetime)

**JSON Output Fields**:
- `id`: Full session ID
- `title`: Session title
- `updated`: Last update timestamp
- `created`: Creation timestamp
- `projectId`: Project ID
- `directory`: Working directory

**Delete Behavior**:
- Uses SQL cascade delete (removes session, messages, parts, todos)
- Requires valid session ID

### STATS - Usage statistics

```bash
# Show stats (all time)
opencode stats

# Last 7 days
opencode stats --days 7

# Top 10 tools
opencode stats --tools 10

# Top 5 models
opencode stats --models 5

# All models
opencode stats --models true

# Current project only
opencode stats --project ""

# Specific project
opencode stats --project <project-id>
```

**Stats Options**:
- `--days <number>` - Show stats for last N days (default: all time)
- `--tools <number>` - Show top N tools (default: all)
- `--models <number|true>` - Show model stats (true=all, number=top N)
- `--project <string>` - Filter by project (empty=curr)

**Stats Output**:
- Total sessions, messages
- Total cost and token usage (input, output, reasoning, cache)
- Tool usage breakdown
- Model usage breakdown (by provider/model)
- Date range, days, cost/per/day, tokens/session

### EXPORT - Export session data

```bash
# Export latest session (interactive)
opencode export

# Export specific session
opencode export <session-id>
```

**Export JSON Schema**:
```json
{
  "info": {
    "id": "session-id",
    "title": "Session title",
    "projectId": "project-id",
    "directory": "/path/to/project",
    "time": {
      "created": 1234567890,
      "updated": 1234567890
    }
  },
  "messages": [
    {
      "info": {
        "id": "msg-id",
        "role": "user|assistant",
        "agent": "agent-name",
        "modelID": "model-name",
        "providerID": "provider-name",
        "time": { "created": 1234567890 },
        "cost": 0.01,
        "tokens": { "input": 100, "output": 200, "reasoning": 0 }
      },
      "parts": [
        {
          "id": "part-id",
          "type": "text|tool|step-start|step-finish|reasoning",
          "text": "...",
          "tool": "tool-name",
          "state": { "status": "completed|running|error", "input": {}, "output": "" }
        }
      ]
    }
  ]
}
```

### IMPORT - Import session data

```bash
# Import from file
opencode import session.json

# Import from share URL
opencode import https://opncd.ai/share/abc123
opencode import https://opencode.ai/share/abc123
```

**Import Behavior**:
- Generates new IDs for sessions/messages (preserves relationships)
- Imports session, messages, parts
- URL format: `https://opncd.ai/share/<slug>` or `https://opencode.ai/share/<slug>`
- Share URL `/api/share/<slug>/data` endpoint returns flat array (session, message, part...)

## Multi-Phase Workflow (Session Resume)

```bash
# Phase 1: Understand
opencode run "Explain how authentication works"

# Phase 2: Plan
opencode run -c "Create a plan for adding password reset"

# Phase 3: Implement
opencode run -c "Implement password reset feature"

# Phase 4: Test
opencode run -c "Run tests and fix issues"

# Phase 5: Review
opencode run -c "Review my changes for security issues"

# Phase 6: Commit and PR
git add .
git commit -m "feat: add password reset"
git checkout -b feat/password-reset
gh pr create -t "feat: add password reset" -b "..."
```

**Session Resume**: Use `--continue` (or `-c`) to maintain conversation context across phases.

## JSON Event Streaming (Scripting)

OpenCode supports JSON event output for scripting:

```bash
# Get raw JSON events
opencode run --format json "Implement X"

# Parse events
opencode run --format json "Implement X" | jq '.'
```

**Event Types**:
- `tool_use`: Tool execution
- `step_start`, `step_finish`: Step lifecycle
- `text`: Assistant response text
- `reasoning`: Thinking blocks (with `--thinking` flag)
- `error`: Session errors
- `session.status`: Session status changes

**Event Structure**:
```json
{
  "type": "event-type",
  "timestamp": 1703275200000,
  "sessionID": "abc123...",
  "part": { ... },
  "error": { ... }
}
```

## Server Mode (Persistent Backend)

Avoid cold start times with persistent server:

```bash
# Terminal 1: Start server
opencode serve --port 4096

# Terminal 2: Attach and connect
opencode run --attach http://localhost:4096 "Implement X"

# Use remote directory
opencode run --attach http://remote:4096 --dir /remote/path "Check file"
```

**Server Advantage**:
- Persistent LLM connections
- Faster startup
- Remote development support
- Shared session state

## File Attachments

Attach files/directories to prompts:

```bash
# Single file
opencode run --file package.json "Review dependencies"

# Multiple files
opencode run --file src/index.ts --file README.md "Review code + docs"

# Directory (mime: application/x-directory)
opencode run --file src/ "Refactor this module"
```

**File Types**:
- Files: `text/plain`
- Directories: `application/x-directory`
- URLs: Converted via `pathToFileURL()`

## Configuration & Initialization

```bash
# Initialize project (creates AGENTS.md)
opencode init

# Configuration files location
~/.local/share/opencode/config.json  # Main config
~/.local/share/opencode/auth.json    # Provider credentials
~/.env                              # Project-level env vars

# List configured providers
opencode models anthropic  # List Anthropic models

# Use specific model
opencode run --model anthropic/claude-sonnet-4-20250514 "Task"
```

## Non-Negotiable Rules

1. **Use OpenCode CLI** — Direct CLI execution is the primary mode
2. **Feature branch** — Always use a feature branch for changes
3. **PR before done** — Always create a PR before completion
4. **GitHub hygiene** — Precise titles, structured bodies, explicit test commands
5. **Self-audit before completion** — Run implementation and review checklists

## Self-Audit Policy

Self-audit is required when any of these are true:
- Code or config changed
- Tests changed or should have changed
- Review is requested
- Docs changed with executable commands

## Tooling + Workflow References

Read these before any work:
- `references/WORKFLOW.md` - Branch, PR, review, multi-phase workflows
- `references/STANDARDS.md` - Coding standards and limits
- `references/COMMANDS.md` - Detailed command reference
- `references/quick-reference.md` - Command cheat sheet

## Requirements

- **OpenCode CLI** (`opencode`) - Install via npm, brew, choco, or scoop:
  - `npm install -g opencode-ai`
  - `brew install anomalyco/tap/opencode`
  - `choco install opencode`
  - `scoop install opencode`
- **GitHub CLI** (`gh`) - For PR workflows
- LLM provider API keys (configure with `opencode auth login`)

## Quick Reference

| Task | Command |
|------|---------|
| Run prompt | `opencode run "prompt"` |
| Resume last | `opencode run --continue` |
| Resume session | `opencode run --session <id>` |
| Fork & continue | `opencode run --continue --fork` |
| List sessions | `opencode session list` |
| Show stats | `opencode stats --days 7` |
| Export session | `opencode export <id>` |
| Import session | `opencode import file.json` |

## Dev Persona

When reviewing or implementing code:
- Be pragmatic and experienced
- Prioritize simplicity over cleverness
- Use examples when explaining concepts
- Ask clarifying questions before making assumptions
- Focus on maintainability and readability

## License

MIT
