# OpenCode Commands Reference

Complete reference for all OpenCode CLI commands and flags based on actual source code.

## Table of Contents

- [RUN Command](#run-command)
- [SESSION Commands](#session-commands)
- [STATS Command](#stats-command)
- [EXPORT Command](#export-command)
- [IMPORT Command](#import-command)
- [Event Streams](#event-streams)

---

## RUN Command

**Purpose**: Execute OpenCode with a message or predefined command

**Source**: `packages/opencode/src/cli/cmd/run.ts`

### Syntax

```bash
opencode run [message..] [OPTIONS]
opencode run --command <name> [message..] [OPTIONS]
```

### Positional Arguments

| Argument | Type | Description |
|----------|------|-------------|
| `message..` | `string[]` | Text messages to send to OpenCode |

### Options

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--command` | | `string` | | Predefined command name (message used as args) |
| `--continue` | `-c` | `boolean` | `false` | Continue the last session (most recent root session) |
| `--session` | `-s` | `string` | | Session ID to continue |
| `--fork` | | `boolean` | `false` | Fork session before continuing (requires `--continue` or `--session`) |
| `--share` | | `boolean` | `false` | Auto-share session |
| `--model` | `-m` | `string` | | Model in format `provider/model` |
| `--agent` | | `string` | | Agent name to use |
| `--format` | | `string` | `default` | Output format: `default` (formatted) or `json` (raw events) |
| `--file` | `-f` | `string[]` | | File(s) to attach to message |
| `--title` | | `string` | | Session title (uses truncated prompt if empty) |
| `--attach` | | `string` | | Attach to running server (e.g., `http://localhost:4096`) |
| `--dir` | | `string` | | Directory to run in (remote path if attaching) |
| `--port` | | `number` | | Local server port (random if not specified) |
| `--variant` | | `string` | | Model variant (e.g., `high`, `max`, `minimal`) |
| `--thinking` | | `boolean` | `false` | Show thinking blocks |

### Session Selection Logic

**`--continue`**:
- Searches for most recent session where `parent_id IS NULL` (root sessions only)
- Excludes forked sessions (those with `parent_id` set)
- Searches within current project directory

**`--session <id>`**:
- Direct lookup by exact session ID
- No search required
- Can be root or forked session

**Neither**:
- Creates new session
- Title source: `--title` flag → truncated prompt (first 50 chars + "...")

### Fork Behavior

When `--fork` is specified with `--continue` or `--session`:

1. Creates new session with `parent_id = original_session_id`
2. Title suffix: `"(fork #N)"` where N is fork count
3. Preserves all message history from parent
4. Original session is not modified

**Use case**: Try different implementations while keeping original context

### File Attachments

`--file` flag supports multiple attachments:

```bash
# Files
opencode run --file package.json --file README.md "Review"

# Directories (mime: application/x-directory)
opencode run --file src/ "Refactor"

# URL conversion uses pathToFileURL()
```

Each file object:
```typescript
{
  type: "file",
  url: "file:///C:/path/to/file.txt",
  filename: "file.txt",
  mime: "text/plain" | "application/x-directory"
}
```

### Server Attachment

When `--attach` is specified:

```bash
# Attach to remote server
opencode run --attach http://remote:4096 --dir /remote/path "Task"

# Attach to local server
opencode run --attach http://localhost:4096 "Task"
opencode run --attach http://opencode.internal --dir /project "Check"
```

**Server connection**:
- Uses SDK client with HTTP fetch
- Directory interpretation:
  - Attaching: Remote server path
  - Not attaching: Local path (requires `process.chdir()`)

### Permission Rules (Non-Interactive)

Default rules for non-interactive mode:

```typescript
[
  { permission: "question", action: "deny", pattern: "*" },
  { permission: "plan_enter", action: "deny", pattern: "*" },
  { permission: "plan_exit", action: "deny", pattern: "*" }
]
```

**Effect**: Auto-rejects permission requests for questions, planning

### JSON Format Output

Event stream structure:

```json
{
  "type": "event-type",
  "timestamp": 1703275200000,
  "sessionID": "abc123...",
  "part": { /* part object */ },
  "error": { /* error object */ }
}
```

See [Event Streams](#event-streams) for event types.

---

## SESSION Commands

**Purpose**: Manage OpenCode sessions

**Source**: `packages/opencode/src/cli/cmd/session.ts`

### session list

**Syntax**:
```bash
opencode session list [OPTIONS]
```

**Options**:

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--max-count` | `-n` | `number` | | Limit to N most recent sessions |
| `--format` | | `string` | `table` | Output format: `table` or `json` |

**Behavior**:
- Lists sessions where `parent_id IS NULL` (roots only)
- Sorted by `time.updated DESC`
- Filters by current project directory

**Table Format**:
```
Session ID                Title                    Updated
───────────────────────── ────────────────────── ──────────────
abc123def456...           Implement feature X      Today 14:30
...
```

**JSON Format**:
```json
[
  {
    "id": "abc123def456...",
    "title": "Implement feature X",
    "updated": 1703275200000,
    "created": 1703271000000,
    "projectId": "proj-id",
    "directory": "/path/to/project"
  }
]
```

### session delete

**Syntax**:
```bash
opencode session delete <session-id>
```

**Arguments**:

| Argument | Type | Description |
|----------|------|-------------|
| `session-id` | `string` | Session ID to delete (required) |

**Behavior**:
- Validates session exists before deletion
- Uses SQL cascade delete (via foreign key constraints)
- Removes: session → messages → parts → todos (automatic)

---

## STATS Command

**Purpose**: Show token usage and cost statistics

**Source**: `packages/opencode/src/cli/cmd/stats.ts`

### Syntax

```bash
opencode stats [OPTIONS]
```

### Options

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--days` | `number` | | Show stats for last N days (default: all time) |
| `--tools` | `number` | | Show top N tools (default: all) |
| `--models` | `number\|true` | | Show model stats (true=all, number=top N) |
| `--project` | `string` | | Filter by project (empty=current) |

### Behavior

**Time filtering**:
- `--days undefined`: All time
- `--days 0`: Today only (midnight to now)
- `--days N`: Last N days from now

**Project filtering**:
- `--project undefined`: All projects
- `--project ""`: Current project (from `Instance.project`)
- `--project <id>`: Specific project ID

### Output Structure

```typescript
{
  totalSessions: number,
  totalMessages: number,
  totalCost: number,
  totalTokens: {
    input: number,
    output: number,
    reasoning: number,
    cache: { read: number, write: number }
  },
  toolUsage: Record<string, number>,  // tool name → count
  modelUsage: {
    "provider/model": {
      messages: number,
      tokens: { input, output, cache: { read, write } },
      cost: number
    }
  },
  dateRange: { earliest: number, latest: number },
  days: number,
  costPerDay: number,
  tokensPerSession: number,
  medianTokensPerSession: number
}
```

### Display Logic

**Tool usage**: Sorted by count (descending), limited by `--tools`

**Model usage**: Sorted by tokens (descending), limited by `--models`

**Date range**: From earliest session `time.created` to latest `time.updated`

---

## EXPORT Command

**Purpose**: Export session data as JSON

**Source**: `packages/opencode/src/cli/cmd/export.ts`

### Syntax

```bash
opencode export [session-id]
```

### Arguments

| Argument | Type | Description |
|----------|------|-------------|
| `session-id` | `string` | Session ID to export (optional) |

### Behavior

**Without session-id**:
- Interactive selection via `@clack/prompts`
- Shows top 10 recent sessions
- Format: `title • time • id-suffix`
- Press `cancel` to exit

**With session-id**:
- Direct export without prompts

### Export Schema

```json
{
  "info": {
    "id": "session-id",
    "title": "Session Title",
    "projectId": "project-id",
    "directory": "/path/to/project",
    "shareUrl": "https://opncd.ai/share/abc",
    "summary": "Session summary...",
    "permission": { "question": { "action": "allow", "pattern": "*" } },
    "time": {
      "created": 1703271000000,
      "updated": 1703275200000
    }
  },
  "messages": [
    {
      "info": {
        "id": "msg-id",
        "sessionID": "session-id",
        "role": "user|assistant",
        "agent": "agent-name",
        "modelID": "claude-sonnet-4-20250514",
        "providerID": "anthropic",
        "time": { "created": 1703271000000 },
        "cost": 0.05,
        "tokens": {
          "input": 1000,
          "output": 2000,
          "reasoning": 0,
          "cache": { "read": 500, "write": 100 }
        },
        "summary": "Quick summary"
      },
      "parts": [
        {
          "id": "part-id",
          "messageID": "msg-id",
          "sessionID": "session-id",
          "type": "text|tool|step-start|step-finish|reasoning|file|image",
          "text": "Text content",
          "tool": "bash",
          "state": {
            "status": "completed|running|error",
            "input": { "command": "ls -la" },
            "output": "drwxr-xr-x...",
            "error": null,
            "metadata": { /* ... */ }
          }
        }
      ]
    }
  ]
}
```

---

## IMPORT Command

**Purpose**: Import session data from JSON file or URL

**Source**: `packages/opencode/src/cli/cmd/import.ts`

### Syntax

```bash
opencode import <file>
```

### Arguments

| Argument | Type | Description |
|----------|------|-------------|
| `file` | `string` | JSON file path or share URL (required) |

### Behavior

**File path**:
- Reads local JSON file
- Must match export schema format
- Skips conflicts (uses `onConflictDoNothing()`)

**Share URL**:
- URL formats:
  - `https://opncd.ai/share/<slug>`
  - `https://opencode.ai/share/<slug>`
- Parse: extract slug from URL path
- API: `GET /api/share/<slug>/data`
- Returns flat array: `[session, message, message, part, part, ...]`

**Transformation**:
```typescript
// API response (flat)
[{ type: "session", data: {...} }, { type: "message", data: {...} }, ...]

// Transform to nested
{
  info: sessionData,
  messages: [{ info: messageData, parts: [partData...] }, ...]
}
```

**Import logic**:
1. Generate new session ID (preserves other fields)
2. Insert session (skip on conflict)
3. For each message:
   - Generate new message ID
   - Insert message (skip on conflict)
   - For each part:
     - Generate new part ID
     - Insert part (skip on conflict)

**Foreign keys**:
- `message.session_id` → new session ID
- `part.message_id` → new message ID
- `part.session_id` → new session ID

---

## Event Streams

**Purpose**: Real-time execution events for streaming/scripting

**Source**: `packages/opencode/src/cli/cmd/run.ts` (execute function)

### Subscription

When `--format json` is specified, events are emitted to stdout:

```bash
opencode run --format json "Task" | jq '.'
```

### Event Types

#### `message.updated`

Emitted when assistant message updated:

```json
{
  "type": "message.updated",
  "properties": {
    "sessionID": "abc...",
    "messageID": "msg-1",
    "info": {
      "role": "assistant",
      "agent": "default",
      "modelID": "claude-sonnet-4-20250514"
    }
  }
}
```

#### `message.part.updated`

Emitted for each part (tool, text, reasoning, etc.):

**Tool part**:
```json
{
  "type": "message.part.updated",
  "properties": {
    "sessionID": "abc...",
    "part": {
      "id": "part-1",
      "messageID": "msg-1",
      "sessionID": "abc...",
      "type": "tool",
      "tool": "bash",
      "state": {
        "status": "completed|running|error",
        "input": { "command": "ls" },
        "output": "...",
        "error": "...",
        "metadata": { "count": 10 }
      }
    }
  }
}
```

**Text part**:
```json
{
  "type": "message.part.updated",
  "properties": {
    "part": {
      "type": "text",
      "text": "Response text...",
      "time": { "start": 123, "end": 456 }
    }
  }
}
```

**Reasoning part**:
```json
{
  "type": "message.part.updated",
  "properties": {
    "part": {
      "type": "reasoning",
      "text": "Thinking process...",
      "time": { "start": 123, "end": 789 }
    }
  }
}
```

#### `step-start`, `step-finish`

Tool execution lifecycle events:

```json
{
  "type": "step-start",
  "properties": {
    "part": { /* tool part object */ }
  }
}

{
  "type": "step-finish",
  "properties": {
    "part": { /* tool part object */ }
  }
}
```

#### `session.error`

Session-level errors:

```json
{
  "type": "session.error",
  "properties": {
    "sessionID": "abc...",
    "error": {
      "name": "ErrorName",
      "message": "Error message",
      "data": { "message": "Detailed error" }
    }
  }
}
```

#### `session.status`

Session lifecycle events:

```json
{
  "type": "session.status",
  "properties": {
    "sessionID": "abc...",
    "status": {
      "type": "idle|busy|error"
    }
  }
}
```

**Event loop termination**: Loop exits when `status.type = "idle"`

#### `permission.asked`

Permission request events:

```json
{
  "type": "permission.asked",
  "properties": {
    "sessionID": "abc...",
    "requestID": "req-123",
    "permission": "question",
    "patterns": ["pattern-1", "pattern-2"]
  }
}
```

**Non-interactive mode**: Auto-rejects all permission requests

### Event Processing Flow

```
1. Subscribe to event stream
2. Loop:
   a. Check message.updated → Show agent/model
   b. Check part.updated
      - Tool completion/error → Show tool output
      - Task running → Show once (toggle map)
      - Text → Print to stdout
      - Reasoning (with --thinking) → Print
   c. Check session.error → Collect errors
   d. Check session.status (type=idle) → Exit loop
   e. Check permission.asked → Auto-reject
3. Execute prompt or command
```

### JSON Event Example

```bash
$ opencode run --format json "List files" | head -10
{"type":"message.updated","timestamp":1703275200000,"sessionID":"abc...","properties":{"messageID":"msg-1","info":{"role":"assistant","agent":"default"}}}
{"type":"message.part.updated","timestamp":1703275201000,"sessionID":"abc...","part":{"id":"part-1","type":"tool","tool":"bash","state":{"status":"running"}}}
{"type":"step-start","timestamp":1703275201500,"sessionID":"abc...","part":{"id":"part-1"}}
{"type":"step-finish","timestamp":1703275205000,"sessionID":"abc...","part":{"id":"part-1","state":{"status":"completed","output":"src/\nREADME.md\n"}}}
{"type":"message.part.updated","timestamp":1703275206000,"sessionID":"abc...","part":{"id":"part-2","type":"text","text":"Here are the files:","time":{"start":1703275206000}}}
{"type":"message.part.updated","timestamp":1703275207000,"sessionID":"abc...","part":{"id":"part-2","type":"text","time":{"end":1703275207000}}}
{"type":"session.status","timestamp":1703275208000,"sessionID":"abc...","status":{"type":"idle"}}
```

---

## Configuration Files

**Location**: `~/.local/share/opencode/`

| File | Purpose |
|------|---------|
| `config.json` | Main configuration (agents, commands, share settings) |
| `auth.json` | Provider credentials and API keys |
| Database | `~/.local/share/opencode/projects/*/db.sqlite` |

**Project-level**: `.env` in project root

---

## Model Selection

**Syntax**:
```bash
opencode models <provider>
opencode run --model <provider/model> Task
```

**Format**: `provider/model` (e.g., `anthropic/claude-sonnet-4-20250514`)

---

## Additional Resources

- **GitHub**: https://github.com/anomalyco/opencode
- **Docs**: https://opencode.ai/docs/
- **Issues**: https://github.com/anomalyco/opencode/issues
