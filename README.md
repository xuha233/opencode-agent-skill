# OpenCode Agent - OpenClaw Control Bridge

🚀 **OpenClaw Agent 控制 OpenCode CLI 的桥梁**

---

## 📖 What is this?

OpenCode Agent 是一个 OpenClaw Skill（技能），让 OpenClaw Agent（小午）能够控制 **OpenCode CLI** —— 一个免费、开源的 AI 编程助手（Claude Code 的替代品）。

### 核心定位

- **外部视角**（用户）：一个能让小午帮你写代码、审查代码、修复 Bug 的工具
- **内部视角**（Agent）："控制 OpenCode CLI 的桥梁" — 决策何时调用、如何调用、解析输出、处理错误

---

## 🎯 Features

### 面向用户（你能做什么）

- ✍️ **写代码**："实现用户注册功能"、"添加 API 错误处理"
- 👀 **审查代码**："审查这段代码的安全性"、"检查代码质量"
- 🔧 **重构优化**："优化性能"、"改进代码结构"
- 💡 **解释代码**："解释这个文件做什么的"、"分析架构"
- 🐛 **修复 Bug**："修复登录失败问题"、"调试这个错误"

### 面向 Agent（小午能做什么）

- 🎯 **意图识别**：自动判断是否为编程任务
- 🤔 **上下文感知**：收集相关文件、检查会话历史
- 🧠 **命令决策**：选择正确的命令和标志（`--continue`、`--fork`、`--file`）
- 📤 **输出解析**：解析 OpenCode 的响应，提取关键信息
- 🔄 **错误恢复**：命令失败时自动处理、重试或降级

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

Just ask OpenClaw Agent (小午) to code:

> "实现一个用户登录功能"

小午会：
1. 识别这是编程任务 ✅
2. 收集相关文件（`src/auth/login.ts`）
3. 调用 OpenCode：`opencode run "实现用户登录功能" --file src/auth/login.ts`
4. 解析输出并呈现给你
5. 如果需要继续，使用 `--continue`

### Example Conversations

```
You: 实现用户注册功能

小午: ✅ 正在实现用户注册...
[OpenCode 实现代码...]

You: 添加邮箱验证

小午: ✅ 继续添加邮箱验证...
[OpenCode 继续实现...]

You: 审查一下代码

小午: ✅ 审查代码...
[OpenCode 审查代码安全性、质量...]
```

---

## Control Bridge - Agent Internal Guide

> 这部分是给 OpenClaw Agent（小午）看的内部文档

### Activation Conditions

激活 Skill 当用户请求匹配以下模式：

1. **Implementation**: "implement", "create", "写代码", "实现"
2. **Code Review**: "review", "audit", "review", "审查"
3. **Refactoring**: "refactor", "optimize", "重构", "优化"
4. **Explanation**: "explain", "what does", "解释"
5. **Bug Fix**: "fix bug", "debug", "修复", "调试"

### Command Decision Logic

```
用户请求 → 意图识别 → 上下文检查 → 命令选择

示例：
用户："继续添加错误处理"
→ 意图：Implementation ✅
→ 上下文：继续任务
→ 命令：opencode run --continue "添加错误处理"
```

### Core Commands for Agent

| Scenarios | Command |
|-----------|---------|
| 新任务 | `opencode run "prompt"` |
| 继续任务 | `opencode run --continue "prompt"` |
| Fork 实验 | `opencode run --continue --fork "prompt"` |
| 审查文件 | `opencode run --file file.ts "review"` |
| 深度推理 | `opencode run -m claude-sonnet-4 "prompt"` |

### Integration Flow

```
1. 意图识别 → 是否为编程任务？
2. 收集上下文 → 相关文件、会话历史
3. 构建命令 → 选择命令 + 标志
4. 执行命令 → opencode run "prompt" --file ...
5. 解析输出 → 提取代码、建议、错误
6. 呈现用户 → 摘要、代码块、重点
7. 错误处理 → 重试、降级、用户反馈
```

**详情参考**: `references/AGENTS_GUIDE.md`

---

## Architecture

### Documentation Layers

```
┌─────────────────────────────────────────────────────────┐
│ SKILL.md                                                 │
├─────────────────────────────────────────────────────────┤
│ • Activation conditions (何时激活)                       │
│ • Command decision logic (用哪个命令)                    │
│ • Context awareness rules (如何收集上下文)             │
│ • Error handling (出错怎么办)                           │
└─────────────────────────────────────────────────────────┘
         ↓
┌─────────────────────────────────────────────────────────┐
│ CLAUDE.md                                                │
├─────────────────────────────────────────────────────────┤
│ • Workflow standards (工作流标准)                        │
│ • Code review patterns (审查模式)                        │
│ • Dev persona (开发者人设)                               │
└─────────────────────────────────────────────────────────┘
         ↓
┌─────────────────────────────────────────────────────────┐
│ references/                                              │
├─────────────────────────────────────────────────────────┤
│ • AGENTS_GUIDE.md (Agent 控制指南)                      │
│ • COMMANDS.md (详细命令参考)                            │
│ • WORKFLOW.md (工作流模式)                              │
│ • STANDARDS.md (编码标准)                               │
│ • quick-reference.md (命令速查表)                        │
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

**You**: "实现用户登出功能"

**Agent (小午)**:
1. ✅ 意图识别：Implementation
2. 📁 文件上下文：`src/auth/session.ts`
3. 🔧 命令：
   ```bash
   opencode run "Implement user logout with JWT invalidation and session management" --file src/auth/session.ts
   ```
4. 📤 解析输出：提取代码
5. 💬 呈现用户：
   ```
   ✅ 实现用户登出功能

   新增/修改的文件：
   - src/auth/logout.ts (新建)
   - src/api/auth/logout.ts (新建)
   - src/auth/session.ts (修改：添加 invalidateSession)

   主要功能：
   - 清除 JWT token
   - 使 session 失效
   - 返回成功响应
   ```

### Example 2: Code Review

**You**: "审查一下登录代码"

**Agent (小午)**:
1. ✅ 意图识别：Code Review
2. 📁 文件上下文：`src/auth/login.ts`, `src/auth/session.ts`
3. 🔧 命令：
   ```bash
   opencode run "Review authentication login and session management for security, quality, best practices" --file src/auth/login.ts --file src/auth/session.ts
   ```
4. 📤 解析输出：提取审查发现
5. 💬 呈现用户：
   ```
   ✅ 代码审查完成

   🔴 严重问题：
   1. SQL 注入风险 (src/auth/login.ts:45) - 查询未参数化
   2. JWT token 未过期检查

   🟡 建议改进：
   1. 添加输入验证
   2. 增加失败重试限制
   3. 改进错误消息

   建议优先处理 🔴 严重问题。

   需要我修复这些问题吗？
   ```

### Example 3: Bug Fix

**You**: "登录总是失败"

**Agent (小午)**:
1. ✅ 意图识别：Bug Fix
2. 📁 文件上下文：`logs/error.log`, `src/auth/login.ts`
3. 🔧 命令：
   ```bash
   opencode run "Investigate: User reports login always fails. Find root cause and propose fix" --file logs/error.log --file src/auth/login.ts
   ```
4. 📤 解析输出：提取根本原因
5. 💬 呈现用户：
   ```
   🔍 问题分析

   根本原因：
   密码比较逻辑错误 (src/auth/login.ts:78)
   ❌ 错误：`if (password !== user.password)`
   ✅ 应该：`bcrypt.compare(password, user.password)`

   需要修复吗？
   ```

### Example 4: Multi-Phase Workflow

**You**: "先实现注册，然后加邮箱验证，然后测试"

**Agent (小午)**:
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
| **Open Source** | ❌ No | ✅ Yes | OpenCode 完全开源 |
| **Price** | 💰 Subscription | ✅ Pay-as-you-go | 按使用量付费 |
| **LLM Providers** | Anthropic only | ✅ 15+ providers | 支持 GPT、DeepSeek、Claude 等 |
| **Session Resume** | `--resume` | `--continue` / `-c` | 语义相同 |
| **Fork Sessions** | ❌ Not supported | ✅ `--fork` | OpenCode 独有 |
| **Server Mode** | ❌ Not supported | ✅ `opencode serve` | 持久化后端，更快 |
| **GitHub Integration** | ✅ Yes | ✅ Yes | 两者都支持 |

**Cost Comparison** (估算):

| Task | Claude Code ($10/mo) | OpenCode (Pay-as-you-go) |
|------|---------------------|-------------------------|
| 100 prompts (simple) | $10 | ≈ $2-5 |
| 50 prompts (complex) | $10 | ≈ $8-15 |

**Recommendation**:
- 频繁使用 → OpenCode 更划算
- 偶尔使用 → Claude Code 更简单

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
