# OpenCode Command Cheat Sheet

Quick reference for common OpenCode CLI commands.

## RUN Command

**Primary execution command**

### Basic Usage

```bash
# Simple prompt
opencode run "Implement feature X"

# Multi-line (stdin)
echo "Long prompt with
multiple lines" | opencode run

# Multi-line (heredoc)
opencode run << EOF
Explain this code:
$(cat src/main.ts)
EOF
```

### Session Resume

```bash
# Continue last session (most recent root)
opencode run --continue
opencode run -c "Add error handling"

# Continue specific session
opencode run --session abc123def456
opencode run -s abc123 "Refactor function"

# Fork before continuing
opencode run --continue --fork "Try different approach"
opencode run -s abc123 --fork
```

### Files & Directories

```bash
# Single file
opencode run --file package.json "Review dependencies"

# Multiple files
opencode run -f src/index.ts -f README.md "Review"

# Directory
opencode run -f src/ "Refactor this module"
```

### Model & Agent

```bash
# Specific model
opencode run --model anthropic/claude-sonnet-4-20250514 "Task"
opencode run -m anthropic/claude-opus-4-20250514 "Complex task"

# Specific agent
opencode run --agent my-agent "Use custom agent"

# Both
opencode run -m anthropic/claude-sonnet-4-20250514 --agent my-agent "Task"
```

### Advanced Flags

```bash
# Custom title
opencode run --title "Feature X Implementation" "Implement X"

# Auto-share
opencode run --share "Share this session"

# Show thinking
opencode run --thinking "Explain reasoning"

# Model variant
opencode run --variant max "Maximum reasoning effort"
```

### Server Mode

```bash
# Terminal 1: Start server
opencode serve --port 4096

# Terminal 2: Attach
opencode run --attach http://localhost:4096 "Task"
opencode run --attach http://localhost:4096 --dir /remote/path "Check"

# Remote server
opencode run --attach http://remote:4096 --dir /path "Remote task"
```

### JSON Output (Scripting)

```bash
# Raw events
opencode run --format json "Task" | jq '.'

# Filter events
opencode run --format json "Task" | jq '.type'

# Tool outputs only
opencode run --format json "Task" | jq 'select(.part.tool)'

# Text only
opencode run --format json "Task" | jq 'select(.part.type == "text") | .part.text'
```

### Predefined Commands

```bash
# Run init command
opencode run --command init "Update AGENTS.md"

# Run custom command from config
opencode run --command my-command "args here"
```

---

## SESSION Commands

### List Sessions

```bash
# Table format (default)
opencode session list

# Table - limit to 10
opencode session list --max-count 10
opencode session list -n 20

# JSON format
opencode session list --format json

# JSON - limit 5
opencode session list -n 5 --format json
```

### Delete Sessions

```bash
# Delete by ID
opencode session delete abc123def456ghi789

# Delete last session (two steps)
opencode session list --format json | jq -r '.[0].id'
opencode session delete <id>
```

---

## STATS Commands

### General Stats

```bash
# All-time stats
opencode stats

# Last 7 days
opencode stats --days 7

# Today only
opencode stats --days 0

# Last 30 days
opencode stats --days 30
```

### Tool Usage

```bash
# All tools
opencode stats --tools

# Top 10 tools
opencode stats --tools 10

# Top 5 tools
opencode stats --tools 5
```

### Model Usage

```bash
# Top 5 models
opencode stats --models 5

# Top 10 models
opencode stats --models 10

# All models
opencode stats --models true
```

### Project Stats

```bash
# Current project only
opencode stats --project ""

# Specific project ID
opencode stats --project proj-id-123

# All projects
opencode stats
```

### Combined Filters

```bash
# Last 7 days, current project, top 10 tools, top 5 models
opencode stats --days 7 --project "" --tools 10 --models 5

# Today, all tools, all models
opencode stats --days 0 --tools true --models true
```

---

## EXPORT Commands

### Export Sessions

```bash
# Export latest (interactive)
opencode export

# Export by ID
opencode export abc123def456

# Export to file
opencode export abc123def456 > session.json

# Export and parse
opencode export abc123 | jq '.messages[].parts[].tool' | grep "bash"
```

---

## IMPORT Commands

### Import Sessions

```bash
# From file
opencode import session.json

# From share URL
opencode import https://opncd.ai/share/abc123
opencode import https://opencode.ai/share/xyz789

# Import from URL and save
opencode import https://opncd.ai/share/abc > imported.json
```

---

## Common Workflows

### Feature Implementation

```bash
# Phase 1: Explore
opencode run "Explain architecture"

# Phase 2: Plan
opencode run -c "Create plan for feature X"

# Phase 3: Implement
opencode run -c "Implement feature X"

# Phase 4: Test
opencode run -c "Add tests and run"

# Phase 5: Review
opencode run -c "Review for bugs"

# Phase 6: Commit
git add .
git commit -m "feat: add feature X"
gh pr create -t "feat: add feature X" -b "..."
```

### Bug Fix

```bash
# Investigate
opencode run -f bug-report.txt "Understand this bug"

# Root cause
opencode run -c "What causes this?"

# Fix
opencode run -c "Fix the bug with minimal changes"

# Test
opencode run -c "Add regression test"

# Commit
git commit -m "fix: resolve issue"
```

### Code Review

```bash
# Checkout PR
gh pr checkout 123

# Review
opencode run -c "Review for architecture, quality, security"

# Fix issues
opencode run -c "Address review findings"

# Re-review
opencode run -c "Re-review after fixes"

# Approve
gh pr review --approve
```

### Refactoring

```bash
# Current state
opencode run "Analyze code quality issues"

# Plan refactor
opencode run -c "Create refactor plan"

# Fork before refactor
opencode run -c -m claude-sonnet-4 --fork "Refactor module"

# Compare original vs forked
opencode session list
opencode export <original-id> > original.json
opencode export <forked-id> > forked.json
```

---

## Session Management Patterns

### Continuous Conversation

```bash
# Start
opencode run "Begin analysis"

# Continue multiple times
opencode run -c "Dig deeper"
opencode run -c "Add implementation details"
opencode run -c "Review for edge cases"
opencode run -c "Finalize design"
```

### Fork Experimentation

```bash
# Base session
opencode run "Implement algorithm A"

# Fork for algorithm B
opencode run --continue --fork "Replace with algorithm B"

# Fork for algorithm C
opencode run -s <fork-b-id> --fork "Try algorithm C"

# Compare all three
opencode session list --max-count 10
```

### Restore Old Session

```bash
# Find session
opencode session list --format json | jq -r '.[] | select(.title | contains("feature")) | .id'

# Restore
opencode run --session <id> "Add more features"

# Continue with different model
opencode run -s <id> --model anthropic/claude-opus-4-20250514 "Refine"
```

---

## Server Mode Workflow

### Persistent Server

```bash
# Terminal 1 - Start server once
opencode serve --port 4096

# Terminal 2+ - All commands connect here
opencode run --attach http://localhost:4096 "Task 1"
opencode run --attach http://localhost:4096 -c "Task 2"
opencode run --attach http://localhost:4096 -c "Task 3"
```

### Remote Development

```bash
# Remote server (hosted on VPS)
opencode serve --port 4096

# Local client connects to remote
opencode run --attach https://my-server.com:4096 --dir /remote/project "Check code"

# Run tests on remote
opencode run --attach https://my-server.com:4096 --dir /remote/project "Test this"
```

### Load Balancing

```bash
# Server 1: 4096
# Server 2: 4097
# Server 3: 4098

# Distribute load
opencode run --attach http://server1:4096 "Task A"
opencode run --attach http://server2:4097 "Task B"
opencode run --attach http://server3:4098 "Task C"
```

---

## JSON Streaming Scripts

### Monitor Execution

```bash
#!/bin/bash
# monitor-opencode.sh

opencode run --format json "$@" | while read -r event; do
  TYPE=$(echo "$event" | jq -r '.type')

  case $TYPE in
    "message.part.updated")
      TOOL=$(echo "$event" | jq -r '.part.tool // empty')
      if [ -n "$TOOL" ]; then
        STATUS=$(echo "$event" | jq -r '.part.state.status')
        echo "[$STATUS] $TOOL"
      fi
      ;;
    "session.error")
      ERROR=$(echo "$event" | jq -r '.properties.error.data.message')
      echo "ERROR: $ERROR"
      ;;
    "session.status")
      STATUS=$(echo "$event" | jq -r '.properties.status.type')
      echo "Session: $STATUS"
      ;;
  esac
done
```

### Extract Tool Calls

```bash
# All bash commands executed
opencode run --format json "Task" | jq -r '.part.tool | select(. == "bash")'

# All file reads
opencode run --format json "Task" | jq 'select(.part.tool == "read") | .part.state.input.filePath'

# All file writes
opencode run --format json "Task" | jq 'select(.part.tool == "write") | .part.state.input.filePath'
```

---

## Quick Reference Table

| Task | Command |
|------|---------|
| Run prompt | `opencode run "prompt"` |
| Continue last | `opencode run --continue` or `-c` |
| Continue session | `opencode run --session <id>` or `-s <id>` |
| Fork session | `opencode run --continue --fork` |
| List sessions | `opencode session list` |
| List sessions (JSON) | `opencode session list --format json` |
| Show stats | `opencode stats` |
| Show last 7 days | `opencode stats --days 7` |
| Show top tools | `opencode stats --tools 10` |
| Show top models | `opencode stats --models 5` |
| Export session | `opencode export <id>` |
| Import session | `opencode import file.json` |
| Start server | `opencode serve --port 4096` |
| Attach to server | `opencode run --attach http://localhost:4096` |

---

## Flag Shortcuts

| Long | Short | Description |
|------|-------|-------------|
| `--continue` | `-c` | Continue last session |
| `--session` | `-s` | Session ID |
| `--model` | `-m` | Model to use |
| `--file` | `-f` | Attach file(s) |
| `--max-count` | `-n` | Limit sessions |

---

## Tips

1. **Use `--format json` for scripting** - Easier to parse
2. **Fork for experiments** - Don't break original session
3. **Use server mode** - Faster for repeated use
4. **Export regularly** - Backup important sessions
5. **Filter stats** - Focus on relevant time/project
6. **Resume smart** - Use `--continue` for continuity, `--session` for specific
7. **Combine flags** - `--days 7 --project "" --tools 10`
8. **Check session list** - Use JSON for programmatic access
