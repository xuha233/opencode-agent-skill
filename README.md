# OpenCode Agent OpenClaw Skill 🚀

OpenClaw skill for coding assistant using OpenCode CLI (open-source alternative to Claude Code). Fully aligned with actual OpenCode source code.

## Features

- **Session Resume Workflows** — Multi-phase issue → implement → PR → review → fix cycles with full context preservation
- **Direct CLI Execution** — Non-interactive mode with `opencode run`
- **Multi-Provider Support** — Use any LLM provider via OpenCode
- **Code Review Patterns** — Structured review workflow (architecture, quality, security, performance, testing)
- **Self-Auditing** — Mandatory implementation + review checklists
- **Dev Persona** — Pragmatic code reviews with clear feedback
- **Fork Workflows** — Experiment without losing original sessions
- **JSON Event Streaming** — Full event output for scripting
- **Server Mode** — Persistent backend for faster startup

## Installation

### 1. Install OpenCode CLI

Choose your preferred method:

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

### 2. Configure LLM Provider

```bash
# Login to provider (opens browser)
opencode auth login

# Check configured providers
opencode auth list
```

### 3. Clone Skill

```bash
# Clone to OpenClaw skills directory
cd ~/.openclaw/skills
git clone https://github.com/your-username/opencode-agent-openclaw-skill.git opencode-agent
```

Or manually copy to `~/.openclaw/skills/opencode-agent/`.

### 4. Initialize Project

For each project you work on:

```bash
cd /path/to/project
opencode init
```

Creates `AGENTS.md` with project-specific guidelines.

## Quick Start

### Basic Usage

```bash
# Execute prompt
opencode run "Implement feature X"

# Continue last session
opencode run --continue
opencode run -c "Fix the bug"

# Resume specific session
opencode run -s abc123def456 "Add error handling"
```

### Complete Workflow

```bash
# Phase 1: Explore
opencode run "Explain authentication flow"

# Phase 2: Plan
opencode run -c "Create plan for password reset"

# Phase 3: Implement
opencode run -c "Implement password reset"

# Phase 4: Test
opencode run -c "Run tests: npm test"

# Phase 5: Review
opencode run -c "Review for security issues"

# Phase 6: Commit
git checkout -b feat/password-reset
git add .
git commit -m "feat: add password reset"
gh pr create -t "feat: add password reset" -b "..."
```

## Core Commands

### RUN - Execution

```bash
opencode run [message..] [OPTIONS]
```

**Key Flags**:
- `--continue, -c` - Resume last session
- `--session, -s <id>` - Resume specific session
- `--fork` - Fork before continuing
- `--file, -f <path>` - Attach files
- `--model, -m <provider/model>` - LLM model
- `--format json` - JSON event output
- `--thinking` - Show reasoning blocks

### SESSION - Session Management

```bash
opencode session list [--format json] [--max-count N]
opencode session delete <session-id>
```

### STATS - Usage Statistics

```bash
opencode stats [--days 7] [--tools 10] [--models 5] [--project ""]
```

### EXPORT/IMPORT - Session Backup

```bash
opencode export [<session-id>] > session.json
opencode import <file.json or URL>
```

## Command Cheat Sheet

| Task | Command |
|------|---------|
 | Run prompt | `opencode run "prompt"` |
| Continue last | `opencode run --continue` |
| Continue session | `opencode run -s <id>` |
| Fork session | `opencode run -c --fork` |
| List sessions | `opencode session list` |
| Show stats | `opencode stats --days 7` |
| Export session | `opencode export <id>` |
| Import session | `opencode import file.json` |
| Start server | `opencode serve --port 4096` |
| Attach to server | `opencode run --attach http://localhost:4096` |

## Model Selection

```bash
# List Anthropic models
opencode models anthropic

# Use specific model
opencode run --model anthropic/claude-sonnet-4-20250514 "Task"

# Default model
opencode run "Task"
```

## Agent Selection

```bash
# List agents
opencode agent list

# Use specific agent
opencode run --agent my-agent "Task"

# Custom agent settings in config.json
~/.local/share/opencode/config.json
```

## Server Mode (Persistent Backend)

Avoid cold start times:

```bash
# Terminal 1: Start server
opencode serve --port 4096

# Terminal 2: Attach and run
opencode run --attach http://localhost:4096 "Task"
```

**Benefits**:
- Persistent LLM connections
- Faster startup (no cold start)
- Remote development support

## JSON Event Streaming

For scripting and integration:

```bash
# Get raw events
opencode run --format json "Task" | jq '.'

# Filter events
opencode run --format json "Task" | jq 'select(.type == "message.part.updated")'

# Tool outputs only
opencode run --format json "Task" | jq 'select(.part.tool) | .part.state.output'
```

**Event Types**:
- `message.updated`
- `message.part.updated`
- `step-start`, `step-finish`
- `session.error`
- `session.status`

## Fork Workflows

Experiment without losing original:

```bash
# Original approach
opencode run "Implement algorithm A"

# Fork for algorithm B
opencode run --continue --fork "Replace with algorithm B"

# Fork for algorithm C
opencode run -s <fork-id> --fork "Try algorithm C"

# Compare
opencode session list
```

## GitHub Integration

### Pull Request Workflow

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

### PR Body Template

```markdown
## What
Brief description (1-2 sentences).

## Why
Reason for the change.

## Tests
```bash
npm test
```

## AI Assistance
Generated with OpenCode Agent
Session: <session-id>
```

## Git Workflow

**Branch naming**: `type/scope-short-description`

**Commit messages**: `type(scope): imperative summary`

**Types**: feat, fix, refactor, docs, test, chore

**Examples**:
```
feat(auth): add password reset functionality
fix(api): handle null response from user endpoint
refactor(ui): simplify component hierarchy
```

## Configuration Files

| File | Location | Purpose |
|------|----------|---------|
| `config.json` | `~/.local/share/opencode/` | Main config |
| `auth.json` | `~/.local/share/opencode/` | Provider credentials |
| `AGENTS.md` | Project root | Project guidelines (created by `opencode init`) |
| `.env` | Project root | Environment variables |

## Documentation

- `SKILL.md` — Core skill documentation (dev persona, commands)
- `CLAUDE.md` — AI agent instructions (workflow, standards)
- `references/COMMANDS.md` — Detailed command reference
- `references/WORKFLOW.md` — Coding workflow patterns
- `references/STANDARDS.md` — Coding standards
- `references/quick-reference.md` — Command cheat sheet

## Dev Persona

When reviewing or implementing code:
- Be pragmatic and experienced
- Prioritize simplicity over cleverness
- Provide specific, actionable feedback
- Explain tradeoffs clearly
- Use examples when explaining concepts

**Review style**:
- Identify specific issues with line references
- Present 2-3 options
- Explain effort, risk, impact
- Recommend ONE approach
- Ask for user decision

## Troubleshooting

### OpenCode not found

```bash
# Check installation
where opencode  # Windows
which opencode  # macOS/Linux

# Reinstall
npm install -g opencode-ai
```

### Authentication errors

```bash
# Check credentials
opencode auth list

# Re-login
opencode auth login
```

### Session not found

```bash
# List sessions
opencode session list

# Use exact session ID from list
opencode run --session <id>
```

### Server connection fails

```bash
# Check if server running
netstat -an | grep 4096  # macOS/Linux
netstat -an | findstr 4096  # Windows

# Start server
opencode serve --port 4096
```

### Format issues

```bash
# Use correct JSON flag (--format NOT --json)
opencode run --format json "Task"
```

## Requirements

- **OpenCode CLI** (`opencode`) - Install via npm, brew, choco, or scoop
- **GitHub CLI** (`gh`) - For PR workflows
- LLM provider API keys - Configure with `opencode auth login`

## License

MIT

## Links

- 📖 [OpenCode Documentation](https://opencode.ai/docs/)
- 🔧 [OpenCode CLI Reference](https://opencode.ai/docs/cli)
- 💻 [OpenCode GitHub](https://github.com/anomalyco/opencode)
- 🔗 [Models Hub](https://models.dev/)
- 🐛 [Issue Tracker](https://github.com/anomalyco/opencode/issues)

## Comparison with Claude Code

| Feature | Claude Code | OpenCode | Notes |
|---------|------------|----------|-------|
| Open Source | No | Yes ✅ | OpenCode fully open |
| Price | Subscription | Pay-as-you-go ✅ | Usage-based pricing |
| LLM Providers | Anthropic only | Multi-provider ✅ | Any LLM provider supported|
| Session Resume | `--resume <id>` | `--session <id>` ✅ | Same semantics |
| Continuing last | `-c` | `--continue` / `-c` ✅ | Same |
| Non-interactive | `--dangerously-skip-permissions` | Built-in ✅ | No TUI in run mode |
| List sessions | `ls` | `session list` ✅ | Different command name |
| JSON output | `--json` | `--format json` ✅ | Same output |
| Fork sessions | No | `--fork` ✅ | OpenCode unique |
| Server mode | No | Yes ✅ | Persistent backend |
| Agent system | Yes | Yes ✅ | Similar capabilities |
| GitHub integration | Yes | Yes ✅ | Both support |

## Credit

Based on actual OpenCode CLI source code (v0.x):
- `packages/opencode/src/cli/cmd/run.ts`
- `packages/opencode/src/cli/cmd/session.ts`
- `packages/opencode/src/cli/cmd/stats.ts`
- `packages/opencode/src/cli/cmd/export.ts`
- `packages/opencode/src/cli/cmd/import.ts`

All commands and flags validated against source implementation.

## Support

- 📚 [Documentation](https://opencode.ai/docs/)
- 💬 [Discord Community](https://opencode.ai/discord)
- 🐛 [GitHub Issues](https://github.com/anomalyco/opencode/issues)
- 📧 [Email Support](mailto:support@opencode.ai)
