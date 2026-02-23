**Current date**: 2026-02-24

## Purpose

High-signal instructions for coding agents using OpenCode CLI in this repository.
Keep this file concise; move long examples and deep procedures to README.md and references/.

## Scope

- Applies to the repository root and descendants.
- Add nested AGENTS.md files only when a subdirectory needs different rules.
- CLAUDE.md should be symlinked from SKILL.md.

## Language

- English only for code, comments, docs, examples, commits, configs, errors, and tests.

## Tooling

- **OpenCode CLI**: `opencode` - Execute `opencode --version` to verify
- **GitHub CLI**: `gh` - Execute `gh --version` to verify
- **Preferred**: `rg` over `grep`, `fd`/`tree` when available
- Resolve OpenCode CLI from PATH (install via npm, brew, choco, or scoop)
- Prefer non-interactive command execution (`opencode run`)

## Runtime Reality Check

Before running major workflows, verify toolchain:

```bash
# Check OpenCode installation
command -v opencode && opencode --version

# Check GitHub CLI
command -v gh && gh --version

# Check git
command -v git && git --version

# Check system timeout (OpenCode has built-in, but good to verify)
command -v timeout
```

If a required binary is missing, stop and report exact install/unblock steps.

---

## OpenCode Command Canon

**Core commands**:

1. **Execution**:
   - Prompt: `opencode run "prompt"`
   - Resume last: `opencode run --continue` or `-c`
   - Resume specific: `opencode run --session <id>` or `-s <id>`
   - Fork: `opencode run --continue --fork`

2. **Session Management**:
   - List: `opencode session list [--format json]`
   - Delete: `opencode session delete <id>`

3. **Statistics**:
   - All time: `opencode stats`
   - Last N days: `opencode stats --days 7`
   - Tools: `opencode stats --tools 10`
   - Models: `opencode stats --models 5`

4. **Export/Import**:
   - Export: `opencode export [<id>]`
   - Import: `opencode import file.json` or URL

**Important flags**:
- `--model <provider/model>` - LLM model selection
- `--agent <name>` - Agent selection
- `--file <path>` - Attach files (array)
- `--format json` - JSON event output
- `--thinking` - Show reasoning blocks
- `--share` - Auto-share session
- `--continue` / `-c` - Resume last session
- `--fork` - Fork before continuing

**Always use `--continue` (or `-c`) for session continuity.**

---

## Workflow

### 1. Context Gathering

Use read-only operations first:

```bash
opencode run "Summarize codebase structure"
opencode run -f README.md "Review project docs"
opencode run -f src/auth.ts -f src/session.ts "Review auth + session"
```

### 2. Planning

For non-trivial work, create a plan:

```bash
opencode run "Create detailed plan for implementing feature X. Consider:
- Requirements: ...
- Constraints: ...
- Dependencies: ..."
```

**Get explicit `APPROVE` before file writes, package installs, or system changes.**

### 3. Implementation

After approval, execute:

```bash
opencode run "Implement the plan for feature X"
```

**Continue conversation as needed**:

```bash
opencode run --continue "Refactor function for performance"
opencode run -c "Add error handling for edge cases"
opencode run --continue "Write unit tests"
```

### 4. Testing

Run relevant checks:

```bash
opencode run -c "Run tests and verify implementation:
```bash
npm test
npm run lint
```"
```

### 5. Code Review

Self-review before PR:

```bash
opencode run -c "Review my changes for:
1. Architecture correctness
2. Code quality and maintainability
3. Edge cases and error handling
4. Security issues
5. Performance implications"
```

---

## Multi-Phase Example

```bash
# Phase 1: Explore
opencode run "Explain authentication flow"

# Phase 2: Plan
opencode run -c "Create plan for adding password reset"

# Phase 3: Implement
opencode run -c "Implement password reset functionality"

# Phase 4: Add tests
opencode run --continue "Add comprehensive tests"

# Phase 5: Run tests
opencode run -c "Run tests: npm test"
opencode run -c --continue "Fix failing tests"

# Phase 6: Self-review
opencode run --continue "Review for security issues"

# Phase 7: Commit and PR
git checkout -b feat/password-reset
git add .
git commit -m "feat: add password reset functionality"
gh pr create -t "feat: add password reset" -b "..."
```

---

## Long-Running Commands

Ensure script processes exit cleanly:
- Wrap long tasks with appropriate timeout
- Verify child processes are stopped after timeout
- Use `opencode run --format json` for scripting (event streaming)

---

## Code Standards

Apply these patterns:

1. **KISS & YAGNI**: Prefer simple solutions; don't speculate
2. **DRY**: Three-strikes rule before abstraction
3. **SRP**: Keep modules/classes focused
4. **Explicit errors**: Never fail silently; handle errors explicitly
5. **Descriptive names**: Use constants, not magic numbers

**Import order**: node → external → internal

---

## Review Expectations

**Review in this order**:

1. **Architecture** (design appropriate?, dependencies correct?, right module?)
2. **Code Quality** (maintainable?, bugs?, error handling?)
3. **Security** (validation?, injection risks?, XSS?)
4. **Performance** (DB queries?, caching?, N+1 queries?)
5. **Testing** (coverage?, edge cases?, meaningful?)

**For each issue**:
- Include file/line references
- Present 2-3 options
- State: effort, risk, impact, maintenance burden
- Recommend ONE option and ask for user decision

---

## Testing and Validation

Before finalizing:

1. **Reproduce first** when debugging
2. **Run relevant checks**:
   - Formatting/lint
   - Typecheck (if applicable)
   - Unit/integration/e2e tests
3. **Report** exact commands run and outcomes
4. **Explicitly call out** checks not run and residual risk

---

## Documentation Hygiene

Update README.md or references/ when:
- Public behavior/workflow changes
- New commands/flags added
- Installation/usage instructions change

**Final report must summarize**:
- Files changed
- Key diffs
- Side effects

**Prefer inclusive language**: allowlist/blocklist

---

## OpenClaw Skill Notes

- Keep SKILL.md AgentSkills-compatible: clear `name` + `description`, concise body, references for detail
- Frontmatter: single-line JSON object in `metadata`
- Metadata requires `anyBins: ["opencode"]` and `bins: ["gh"]`

---

## OpenCode-Specific Notes

### Session Management

- **Session resume**: Always use `--continue` or `--session <id>` for context
- **Session list**: `opencode session list --format json` for programmatic access
- **Forking**: Use `--continue --fork` to experiment without losing original

### Non-Interactive Mode

- **Permission rules**: Auto-rejects question, plan_enter, plan_exit
- **No TUI**: Pure event streaming (default format) or JSON output
- **Stdin support**: Reads from stdin if not TTY

### JSON Event Streaming

For scripting, use `--format json`:

```bash
opencode run --format json "Task" | jq '.'
```

**Event types**:
- `message.updated`
- `message.part.updated`
- `step-start`, `step-finish`
- `session.error`
- `session.status`
- `permission.asked`

### Server Mode

Avoid cold starts with persistent server:

```bash
# Terminal 1
opencode serve --port 4096

# Terminal 2
opencode run --attach http://localhost:4096 "Task"
```

### File Attachments

```bash
# Single file
opencode run -f package.json "Review"

# Multiple files
opencode run -f src/index.ts -f README.md "Review"

# Directory (mime: application/x-directory)
opencode run -f src/ "Refactor"
```

### CLI Drift Check

Periodically verify:

```bash
opencode --help
opencode run --help
opencode session list --help
opencode stats --help
```

Update references when flags/behavior drift.

---

## Dev Persona

**When reviewing or implementing code**:

- **Pragmatic**: Focus on what works, not theoretical perfection
- **Experienced**: Draw from industry best practices
- **Clear**: Provide specific, actionable feedback
- **Balanced**: Consider tradeoffs before recommending
- **Helpful**: Use examples when explaining concepts

**Review style**:
- Provide options, not just solutions
- Explain tradeoffs clearly
- Recommend ONE approach with justification
- Ask for user decision on important choices

---

## CLI Symmetry with Claude Code

| Claude Code | OpenCode | Difference |
|-------------|----------|------------|
| `claude -p "prompt"` | `opencode run "prompt"` | Same ✅ |
| `claude -p -c "follow"` | `opencode run --continue "follow"` | Same ✅ |
| `claude -p --resume <id>` | `opencode run --session <id>` | Same ✅ |
| `--dangerously-skip-permissions` | Built-in non-interactive (no TUI) | Different 🔄 |
| `ls` | `session list` | Different 🔄 |
| `--json` | `--format json` | Same ✅ |

---

## Agent Configuration

**Agent selection**:

```bash
# List available agents
opencode agent list

# Use specific agent
opencode run --agent my-agent "Task"

# Agents marked as `subagent` cannot be primary
```

**Model selection**:

```bash
# List models from provider
opencode models anthropic

# Use specific model
opencode run --model anthropic/claude-sonnet-4-20250514 "Task"

# Use default model
opencode run "Task"
```

---

## Stats & Monitoring

Monitor token usage and costs:

```bash
# All-time stats
opencode stats

# Last 7 days
opencode stats --days 7

# Current project
opencode stats --project ""

# By current directory tree
opencode stats --days 7 --project ""

# Top tools/models
opencode stats --tools 10 --models 5
```

---

## Git Workflow

**Branch naming**: `type/scope-short-description`

**Commit messages**: `type(scope): imperative summary`

**PR titles**: `type(scope): imperative summary`

**Types**: feat, fix, refactor, docs, test, chore

**Example PR body**:
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

---

## Error Handling

**When OpenCode makes mistakes**:

```bash
# Ask it to fix
opencode run --continue "Tests failed. Fix the implementation."

# Roll back and retry
git reset --hard HEAD
opencode run --continue "Retry with different approach."

# Fork before fixing
opencode run --continue --fork "Try alternative fix."
```

---

## Project Initialization

Before using OpenCode for a project:

```bash
# Navigate to project
cd /path/to/project

# Initialize (creates AGENTS.md)
opencode init
```

The `init` command analyzes the project structure and creates AGENTS.md.

---

## References

For deep details, see:
- `references/COMMANDS.md` - Detailed command reference
- `references/WORKFLOW.md` - Coding workflow patterns
- `references/STANDARDS.md` - Coding standards
- `references/quick-reference.md` - Command cheat sheet
- `README.md` - Installation and usage guide
