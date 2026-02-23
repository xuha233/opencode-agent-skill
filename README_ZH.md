# OpenCode Agent OpenClaw Skill 🚀

使用 OpenCode CLI 构建编程助手的 OpenClaw Skill（Claude Code 的开源替代品）。完全基于 OpenCode 源代码验证。

## 特性

- **会话恢复工作流** — 多阶段 探索 → 计划 → 实现 → PR → 审查 → 修复 循环，完整保留上下文
- **直接 CLI 执行** — 非交互模式，使用 `opencode run`
- **多提供商支持** — 通过 OpenCode 使用任何 LLM 提供商
- **代码审查模式** — 结构化审查流程（架构、质量、安全、性能、测试）
- **自我审计** — 强制的实现 + 审查检查清单
- **开发者人设** — 务实的代码审查，提供清晰反馈
- **Fork 工作流** — 在不丢失原始会话的情况下尝试实验
- **JSON 事件流** — 完整的事件输出，便于脚本集成
- **服务器模式** — 持久化后端，启动更快

## 安装

### 1. 安装 OpenCode CLI

选择你喜欢的安装方式：

```bash
# NPM（跨平台）
npm install -g opencode-ai

# Homebrew（macOS/Linux）
brew install anomalyco/tap/opencode

# Chocolatey（Windows）
choco install opencode

# Scoop（Windows）
scoop install opencode
```

### 2. 配置 LLM 提供商

```bash
# 登录提供商（会打开浏览器）
opencode auth login

# 查看已配置的提供商
opencode auth list
```

### 3. 克隆 Skill

```bash
# 克隆到 OpenClaw skills 目录
cd ~/.openclaw/skills
git clone https://github.com/xuha233/opencode-agent-skill.git opencode-agent
```

或者手动复制到 `~/.openclaw/skills/opencode-agent/`。

### 4. 初始化项目

对于每个你处理的项目：

```bash
cd /path/to/project
opencode init
```

这会创建包含项目特定指南的 `AGENTS.md`。

## 快速开始

### 基本用法

```bash
# 执行提示
opencode run "实现功能 X"

# 继续上次会话
opencode run --continue
opencode run -c "修复这个 bug"

# 恢复特定会话
opencode run -s abc123def456 "添加错误处理"
```

### 完整工作流

```bash
# 阶段 1：探索
opencode run "解释认证流程"

# 阶段 2：规划
opencode run -c "创建密码重置计划"

# 阶段 3：实现
opencode run -c "实现密码重置功能"

# 阶段 4：测试
opencode run -c "运行测试：npm test"

# 阶段 5：审查
opencode run -c "审查安全问题"

# 阶段 6：提交
git checkout -b feat/password-reset
git add .
git commit -m "feat: 添加密码重置功能"
gh pr create -t "feat: 添加密码重置功能" -b "..."
```

## 核心命令

### RUN - 执行

```bash
opencode run [message..] [OPTIONS]
```

**主要标志**：
- `--continue, -c` - 恢复上次会话
- `--session, -s <id>` - 恢复指定会话
- `--fork` - 继续前 Fork 会话
- `--file, -f <path>` - 附加文件
- `--model, -m <provider/model>` - LLM 模型
- `--format json` - JSON 事件输出
- `--thinking` - 显示推理过程

### SESSION - 会话管理

```bash
opencode session list [--format json] [--max-count N]
opencode session delete <session-id>
```

### STATS - 使用统计

```bash
opencode stats [--days 7] [--tools 10] [--models 5] [--project ""]
```

### EXPORT/IMPORT - 会话备份

```bash
opencode export [<session-id>] > session.json
opencode import <file.json or URL>
```

## 命令速查表

| 任务 | 命令 |
|------|---------|
 | 运行提示 | `opencode run "提示"` |
| 继续上次 | `opencode run --continue` |
| 继续会话 | `opencode run -s <id>` |
| Fork 会话 | `opencode run -c --fork` |
| 列出会话 | `opencode session list` |
| 显示统计 | `opencode stats --days 7` |
| 导出会话 | `opencode export <id>` |
| 导入会话 | `opencode import file.json` |
| 启动服务器 | `opencode serve --port 4096` |
| 连接服务器 | `opencode run --attach http://localhost:4096` |

## 模型选择

```bash
# 列出 Anthropic 模型
opencode models anthropic

# 使用特定模型
opencode run --model anthropic/claude-sonnet-4-20250514 "任务"

# 使用默认模型
opencode run "任务"
```

## Agent 选择

```bash
# 列出 agents
opencode agent list

# 使用特定 agent
opencode run --agent my-agent "任务"

# config.json 中的自定义 agent 设置
~/.local/share/opencode/config.json
```

## 服务器模式（持久化后端）

避免冷启动时间：

```bash
# 终端 1：启动服务器
opencode serve --port 4096

# 终端 2：连接并运行
opencode run --attach http://localhost:4096 "任务"
```

**优势**：
- 持久的 LLM 连接
- 更快的启动（无冷启动）
- 远程开发支持

## JSON 事件流

用于脚本和集成：

```bash
# 获取原始事件
opencode run --format json "任务" | jq '.'

# 过滤事件
opencode run --format json "任务" | jq 'select(.type == "message.part.updated")'

# 仅输出工具执行结果
opencode run --format json "任务" | jq 'select(.part.tool) | .part.state.output'
```

**事件类型**：
- `message.updated`
- `message.part.updated`
- `step-start`, `step-finish`
- `session.error`
- `session.status`

## Fork 工作流

在不丢失原始会话的情况下实验：

```bash
# 原始方法
opencode run "实现算法 A"

# Fork 尝试算法 B
opencode run --continue --fork "替换为算法 B"

# Fork 尝试算法 C
opencode run -s <fork-id> --fork "尝试算法 C"

# 比较
opencode session list
```

## GitHub 集成

### Pull Request 工作流

```bash
# 检出 PR
gh pr checkout 123

# 审查
opencode run -c "审查架构、质量、安全问题"

# 修复问题
opencode run -c "解决审查 findings"

# 重新审查
opencode run -c "修复后重新审查"

# 批准
gh pr review --approve
```

### PR 描述模板

```markdown
## 变更内容
简短描述（1-2 句话）。

## 变更原因
变更的原因。

## 测试方式
```bash
npm test
```

## AI 辅助
使用 OpenCode Agent 生成
会话 ID：<session-id>
```

## Git 工作流

**分支命名**：`type/scope-short-description`

**提交信息**：`type(scope): 命令式总结`

**类型**：feat（新功能）、fix（修复）、refactor（重构）、docs（文档）、test（测试）、chore（维护）

**示例**：
```
feat(auth): 添加密码重置功能
fix(api): 处理用户端点的 null 响应
refactor(ui): 简化组件层次结构
```

## 配置文件

| 文件 | 位置 | 用途 |
|------|------|------|
| `config.json` | `~/.local/share/opencode/` | 主配置 |
| `auth.json` | `~/.local/share/opencode/` | 提供商凭证 |
| `AGENTS.md` | 项目根目录 | 项目指南（由 `opencode init` 创建） |
| `.env` | 项目根目录 | 环境变量 |

## 文档

- `SKILL.md` — 核心 skill 文档（开发者人设、命令）
- `CLAUDE.md` — AI Agent 指令（工作流、标准）
- `references/COMMANDS.md` — 详细命令参考
- `references/WORKFLOW.md` — 编码工作流模式
- `references/STANDARDS.md` — 编码标准
- `references/quick-reference.md` — 命令速查表

## 开发者人设

审查或实现代码时：
- 务实且经验丰富
- 优先简单而非聪明
- 提供具体、可操作的反馈
- 清晰解释权衡
- 解释概念时使用示例

**审查风格**：
- 使用行引用标识具体问题
- 提供 2-3 个选项
- 解释工作量、风险、影响
- **推荐 ONE 种方法**
- 询问用户决策

## 故障排除

### 找不到 OpenCode

```bash
# 检查安装
where opencode  # Windows
which opencode  # macOS/Linux

# 重新安装
npm install -g opencode-ai
```

### 身份验证错误

```bash
# 检查凭证
opencode auth list

# 重新登录
opencode auth login
```

### 找不到会话

```bash
# 列出会话
opencode session list

# 使用列表中的确切会话 ID
opencode run --session <id>
```

### 服务器连接失败

```bash
# 检查服务器是否运行
netstat -an | grep 4096  # macOS/Linux
netstat -an | findstr 4096  # Windows

# 启动服务器
opencode serve --port 4096
```

### 格式问题

```bash
# 使用正确的 JSON 标志（--format 而不是 --json）
opencode run --format json "任务"
```

## 依赖要求

- **OpenCode CLI** (`opencode`) - 通过 npm、brew、choco 或 scoop 安装
- **GitHub CLI** (`gh`) - 用于 PR 工作流
- LLM 提供商 API 密钥 - 使用 `opencode auth login` 配置

## 许可证

MIT

## 相关链接

- 📖 [OpenCode 官方文档](https://opencode.ai/docs/)
- 🔧 [OpenCode CLI 参考手册](https://opencode.ai/docs/cli)
- 💻 [OpenCode GitHub 仓库](https://github.com/anomalyco/opencode)
- 🔗 [模型市场](https://models.dev/)
- 🐛 [问题追踪](https://github.com/anomalyco/opencode/issues)

## 与 Claude Code 对比

| 特性 | Claude Code | OpenCode | 备注 |
|------|------------|----------|------|
| 开源 | 否 | 是 ✅ | OpenCode 完全开源 |
| 价格 | 订阅 | 按需付费 ✅ | 基于使用量的定价 |
| LLM 提供商 | 仅 Anthropic | 多提供商 ✅ | 支持任何 LLM 提供商 |
| 会话恢复 | `--resume <id>` | `--session <id>` ✅ | 语义相同 |
| 继续上次 | `-c` | `--continue` / `-c` ✅ | 相同 |
| 非交互式 | `--dangerously-skip-permissions` | 内置 ✅ | run 模式无 TUI |
| 列出会话 | `ls` | `session list` ✅ | 命令名不同 |
| JSON 输出 | `--json` | `--format json` ✅ | 输出相同 |
| Fork 会话 | 不支持 | `--fork` ✅ | OpenCode 独有 |
| 服务器模式 | 不支持 | 是 ✅ | 持久化后端 |
| Agent 系统 | 支持 | 支持 ✅ | 功能相似 |
| GitHub 集成 | 支持 | 支持 ✅ | 两者都支持 |

## 作者与维护

**作者**: 言午间

**联系方式**: 3537183821@qq.com

---

## 致谢

基于 OpenCode CLI 实际源代码（v0.x）：
- `packages/opencode/src/cli/cmd/run.ts`
- `packages/opencode/src/cli/cmd/session.ts`
- `packages/opencode/src/cli/cmd/stats.ts`
- `packages/opencode/src/cli/cmd/export.ts`
- `packages/opencode/src/cli/cmd/import.ts`

所有命令和标志均已对照源实现验证。

## 支持

- 📚 [文档](https://opencode.ai/docs/)
- 💬 [Discord 社区](https://opencode.ai/discord)
- 🐛 [GitHub 问题](https://github.com/anomalyco/opencode/issues)
- 📧 [邮件支持](mailto:support@opencode.ai)
