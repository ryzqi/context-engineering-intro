# 🚀 Claude Code 完全使用指南

了解使用 Claude Code 构建任何内容所需的一切！本指南将带你从安装到高级上下文工程、子代理、钩子和并行代理工作流。

## 📋 前提条件

- 终端/命令行访问
- 已安装 Node.js（用于 Claude Code 安装）
- GitHub 账户（用于 GitHub CLI 集成）
- 文本编辑器（推荐使用 VS Code）

## 🔧 安装

**macOS/Linux:**

```bash
npm install -g @anthropic-ai/claude-code
```

**Windows (WSL recommended):**
参见 [install_claude_code_windows.md](./install_claude_code_windows.md) 中的详细说明

**验证安装:**

```bash
claude --version
```

---

## ✅ 提示 1：创建和优化 CLAUDE.md 文件

设置上下文文件，Claude 会在每次对话中自动拉取这些文件，其中包含项目特定信息、命令和指南。

```bash
mkdir your-folder-name && cd your-folder-name
claude
```

使用内置命令：

```
/init
```

或根据此仓库中的模板创建自己的 CLAUDE.md 文件。参见 `CLAUDE.md` 了解 Python 特定示例结构，其中包含：

- 项目认知和上下文规则
- 代码结构指南
- 测试要求
- 任务完成工作流
- 风格规范
- 文档标准

### 高级提示工程技术

**关键词**：Claude 对某些关键词有增强行为响应（信息密集型关键词）：

- **IMPORTANT**：强调不应被忽视的关键指令
- **Proactively**：鼓励 Claude 主动提出改进建议
- **Ultra-think**：可触发更彻底的分析（谨慎使用）

**基本提示工程技巧**：

- 避免提示"生产就绪"代码 - 这通常会导致过度工程化
- 提示 Claude 编写脚本检查其工作："实现后，创建验证脚本"
- 除非特别需要，避免向后兼容 - Claude 往往会不必要地保留旧代码
- 专注于清晰和具体的要求，而非模糊的质量描述

### 文件放置策略

Claude 自动从多个位置读取 CLAUDE.md 文件：

```bash
# 仓库根目录（最常见）
./CLAUDE.md              # 提交到 git，与团队共享
./CLAUDE.local.md        # 仅本地使用，添加到 .gitignore

# 父目录（用于单体仓库）
root/CLAUDE.md           # 通用项目信息
root/frontend/CLAUDE.md  # 前端特定上下文
root/backend/CLAUDE.md   # 后端特定上下文

# 引用外部文件以获得灵活性
echo "遵循最佳实践：~/company/engineering-standards.md" > CLAUDE.md
```

**专业提示**：许多团队保持 CLAUDE.md 简洁并引用共享标准文档。这使得：

- 可以在 AI 编码助手之间切换
- 无需更改每个项目即可更新标准
- 在团队之间共享最佳实践

*注意：虽然 Claude Code 自动读取 CLAUDE.md，但其他 AI 编码助手可以使用类似的上下文文件（例如 Cursor 的 .cursorrules）*

---

## ✅ 提示 2：设置权限管理

配置工具允许列表，以简化开发同时维护文件操作和系统命令的安全性。

**方法 1：交互式允许列表**
当 Claude 请求权限时，为常见操作选择"始终允许"。

**方法 2：使用 /permissions 命令**

```
/permissions
```

然后添加：

- `Edit`（用于文件编辑）
- `Bash(git commit:*)`（用于 git 提交）
- `Bash(npm:*)`（用于 npm 命令）
- `Read`（用于读取文件）
- `Write`（用于创建文件）

**方法 3：创建项目设置文件**
创建 `.claude/settings.local.json`：

```json
{
  "allowedTools": [
    "Edit",
    "Read",
    "Write",
    "Bash(git add:*)",
    "Bash(git commit:*)",
    "Bash(npm:*)",
    "Bash(python:*)",
    "Bash(pytest:*)"
  ]
}
```

**安全最佳实践**：

- 切勿允许 `Bash(rm -rf:*)` 或类似破坏性命令
- 使用特定命令模式而非 `Bash(*)`
- 定期审查权限
- 为不同项目使用不同的权限集

*注意：所有 AI 编码助手都有权限管理 - 有些内置，有些需要手动批准每个操作。*

---

## ✅ 提示 3：掌握自定义斜杠命令

斜杠命令是将您自己的工作流添加到 Claude Code 的关键。它们位于 `.claude/commands/` 中，使您能够创建可重用的、参数化的工作流。

### 内置命令

- `/init` - 生成初始 CLAUDE.md
- `/permissions` - 管理工具权限
- `/clear` - 在任务之间清除上下文
- `/agents` - 管理子代理
- `/help` - 获取 Claude Code 帮助

### 自定义命令示例

**仓库分析**：

```
/primer
```

执行全面的仓库分析，为 Claude Code 提供必要的上下文，以便您可以开始实施修复或新功能。

### 创建您自己的命令

1. 在 `.claude/commands/` 中创建 markdown 文件：

```markdown
# Command: analyze-performance

分析 $ARGUMENTS 中指定文件的性能。

## Steps:
1. 读取路径处的文件：$ARGUMENTS
2. 识别性能瓶颈
3. 建议优化方案
4. 创建基准测试脚本
```

2. 使用命令：

```
/analyze-performance src/heavy-computation.js
```

命令可以使用 `$ARGUMENTS` 接收参数并调用 Claude 的任何工具。

*注意：其他 AI 编码助手可以将这些命令用作常规提示 - 只需复制命令内容并粘贴参数。*

---

## ✅ 提示 4：集成 MCP 服务器

将 Claude Code 连接到 Model Context Protocol (MCP) 服务器以获得增强功能。在 [MCP 文档](https://docs.anthropic.com/en/docs/claude-code/mcp) 中了解更多信息。

**添加 Serena MCP 服务器** - 最强大的编码工具包：

确保首先[安装 uvx](https://docs.astral.sh/uv/getting-started/installation/#standalone-installer)。在带有 Windows 的 WSL 中操作方法如下：

```bash
sudo snap install astral-uv --classic
```

然后使用以下命令添加 Serena：

```bash
# 安装 Serena 用于语义代码分析和编辑
claude mcp add serena -- uvx --from git+  https://github.com/oraios/serena   serena start-mcp-server --context ide-assistant --project $(pwd)
```

[Serena](https://github.com/oraios/serena) 将 Claude Code 转变为功能齐全的编码代理，具有：

- 语义代码检索和分析
- 使用 Language Server Protocol (LSP) 的高级编辑功能
- 支持 Python、TypeScript/JavaScript、PHP、Go、Rust、C/C++、Java
- 订阅式编码助手的免费开源替代方案

**管理 MCP 服务器**：

```bash
# 列出所有配置的服务器
claude mcp list

# 获取特定服务器的详细信息
claude mcp get serena

# 移除服务器
claude mcp remove serena
```

**即将推出**：Archon V2（重大更新）- AI 编码助手的综合知识和任务管理骨干 - 首次实现真正的人机协作编码。

*注意：MCP 与每个主要的 AI 编码助手集成，服务器以非常相似的方式管理。*

---

## ✅ 提示 5：使用示例进行上下文工程

将您的开发工作流从简单提示转变为全面的上下文工程 - 为 AI 提供端到端实施所需的所有信息。

### 快速入门

PRP（产品需求提示）框架是一种简单的 3 步策略，用于上下文工程：

```bash
# 1. 使用示例和上下文定义需求
# 编辑 INITIAL.md 以包含示例代码和模式

# 2. 生成全面的 PRP
/generate-prp INITIAL.md

# 3. 执行 PRP 以实现您的功能
/execute-prp PRPs/your-feature-name.md
```

### 定义您的需求

您的 INITIAL.md 应始终包含：

```markdown
## FEATURE
构建用户认证系统

## EXAMPLES
- 认证流程：`examples/auth-flow.js`
- 类似 API 端点：`src/api/users.js` 
- 数据库模式模式：`src/models/base-model.js`
- 验证方法：`src/validators/user-validator.js`

## DOCUMENTATION
- JWT 库文档：https://github.com/auth0/node-jsonwebtoken  
- 我们的 API 标准：`docs/api-guidelines.md`

## OTHER CONSIDERATIONS
- 使用现有错误处理模式
- 遵循我们的标准响应格式
- 包括速率限制
```

### 关键 PRP 策略

**示例**：最强大的工具 - 提供代码片段、类似功能和要遵循的模式

**验证门**：确保全面测试并在所有测试通过后迭代

**无氛围编码**：在执行 PRP 之前和执行代码后进行验证！

您提供的示例越具体，Claude 就能更好地匹配您现有的模式和风格。

*注意：上下文工程适用于任何 AI 编码助手 - PRP 框架和基于示例的方法是通用原则。*

---

## ✅ 提示 6：利用子代理处理特定任务

子代理是专门的 AI 助手，它们在单独的上下文窗口中运行，具有专注的专业知识。它们使 Claude 能够将特定任务委托给专家，提高质量和效率。

### 了解子代理

每个子代理：

- 拥有自己的上下文窗口（不会污染主对话）
- 使用专门的系统提示运行
- 可以限制为特定工具
- 在委托任务上自主工作

### 本仓库中的示例子代理

**文档管理器** (`.claude/agents/documentation-manager.md`):

- 在代码更改时自动更新文档
- 确保 README 准确性
- 维护 API 文档
- 创建迁移指南

**验证门** (`.claude/agents/validation-gates.md`):

- 在更改后运行所有测试
- 迭代修复直到测试通过
- 强制执行代码质量标准
- 永远不会在测试失败时标记任务完成

### 创建您自己的子代理

1. 使用 `/agents` 命令或在 `.claude/agents/` 中创建文件：

```markdown
---
name: security-auditor
description: "安全专家。主动审查代码中的漏洞并提出改进建议。"
tools: Read, Grep, Glob
---

您是一位专注于识别和预防安全漏洞的安全审计专家...

## 核心职责
1. 审查代码中的 OWASP Top 10 漏洞
2. 检查暴露的机密或凭据
3. 验证输入清理
4. 确保适当的认证/授权
...
```

### 子代理最佳实践

**1. 专注专业知识**：每个子代理应有一个明确的专业领域

**2. 主动描述**：在描述中使用"proactively"以实现自动调用：

```yaml
description: "代码审查员。主动审查所有代码更改的质量。"
```

**3. 工具限制**：仅向子代理提供所需的工具：

```yaml
tools: Read, Grep  # 仅用于只读审查的代理
```

**4. 信息流设计**：了解信息如何从主代理 → 子代理 → 主代理流动。子代理描述至关重要，因为它告诉您的主 Claude Code 代理何时以及如何使用它。在描述中包含关于主代理应如何提示此子代理的明确说明。

**5. 一次性上下文**：子代理没有完整的对话历史 - 它们从您的主代理接收单个提示。在设计子代理时考虑此限制。

在 [子代理文档](https://docs.anthropic.com/en/docs/claude-code/sub-agents) 中了解更多信息。

*注意：虽然其他 AI 助手没有正式的子代理，但您可以通过创建专门的提示并在不同对话上下文之间切换来实现类似结果。*

---

## ✅ 提示 7：使用钩子进行自动化

钩子通过用户定义的 shell 命令提供对 Claude Code 行为的确定性控制，这些命令在预定义的生命周期事件中执行。

### 可用钩子事件

Claude Code 提供了几个可以挂钩的预定义操作：

- **PreToolUse**: 在工具执行之前（可以阻止操作）
- **PostToolUse**: 在工具成功完成之后
- **UserPromptSubmit**: 当用户提交提示时
- **SubagentStop**: 当子代理完成其任务时
- **Stop**: 当主代理完成响应时
- **SessionStart**: 在会话初始化时
- **PreCompact**: 在上下文压缩之前
- **Notification**: 在系统通知期间

在 [钩子文档](https://docs.anthropic.com/en/docs/claude-code/hooks) 中了解更多信息。

### 示例钩子：工具使用日志

此仓库在 `.claude/hooks/` 中包含一个简单的钩子示例：

**log-tool-usage.sh** - 记录所有工具使用情况以进行跟踪和调试：

```bash
#!/bin/bash
# 记录带有时间戳的工具使用情况
# 创建 .claude/logs/tool-usage.log
# 无需外部依赖
```

### 设置钩子

1. **在 `.claude/hooks/` 中创建钩子脚本**
2. **使其可执行**：`chmod +x your-hook.sh`
3. **在 `.claude/settings.local.json` 中添加到设置**：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": ".*",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/log-tool-usage.sh"
          }
        ]
      }
    ]
  }
}
```

钩子确保某些操作始终发生，而不是依赖 AI 记住 - 非常适合日志记录、安全验证和构建触发。

*注意：其他 AI 助手没有钩子（尽管 Kiro 有！），我几乎可以保证他们很快也会为其他人添加。*

---

## ✅ 提示 8：GitHub CLI 集成

设置 GitHub CLI 以使 Claude 能够与 GitHub 交互，处理问题、拉取请求和仓库管理。

```bash
# 安装 GitHub CLI
# 访问：https://github.com/cli/cli#installation

# 认证
gh auth login

# 验证设置
gh repo list
```

### 自定义 GitHub 命令

使用 `/fix-github-issue` 命令进行自动修复：

```
/fix-github-issue 123
```

这将：

1. 获取问题详细信息
2. 分析问题
3. 搜索相关代码
4. 实施修复
5. 运行测试
6. 创建 PR

*注意：GitHub CLI 适用于任何 AI 编码助手 - 只需安装它，AI 就可以使用 `gh` 命令与您的仓库交互。*

---

## ✅ 提示 9：使用开发容器的 SAFE YOLO 模式

允许 Claude Code 执行任何操作，同时通过容器化维护安全性。这使得可以在不对主机系统造成破坏性行为的情况下进行快速开发。

**前提条件**：

- 安装 [Docker](https://www.docker.com/)
- VS Code（或兼容编辑器）

**安全功能**：

- 网络隔离与白名单
- 无主机文件系统访问权限
- 限制出站连接
- 安全实验环境

**设置过程**：

1. **在 VS Code 中打开**并按 `F1`
2. **选择**"Dev Containers: Reopen in Container"
3. **等待**容器构建
4. **打开终端**（`Ctrl+J`）
5. **在容器中认证** Claude Code
6. **以 YOLO 模式运行**：
   ```bash
   claude --dangerously-skip-permissions
   ```

**为什么使用开发容器**？

- 安全测试危险操作
- 实验系统更改
- 快速原型设计
- 一致的开发环境
- 无需担心系统损坏

---

## ✅ 提示 10：使用 GIT WORKTREES 进行并行开发

使用 Git worktrees 使多个 Claude 实例同时处理独立任务，或自动化同一功能的并行实现。

### 手动 Worktree 设置

```bash
# 为不同功能创建 worktrees
git worktree add ../project-auth feature/auth
git worktree add ../project-api feature/api

# 在每个 worktree 中启动 Claude
cd ../project-auth && claude  # Terminal 1
cd ../project-api && claude   # Terminal 2
```

### 自动化并行代理

AI 编码助手是非确定性的。运行多次尝试可提高成功率并提供实现选项。

**设置并行 worktrees**：

```bash
/prep-parallel user-system 3
```

**执行并行实现**：

1. 创建计划文件（`plan.md`）
2. 运行并行执行：

```bash
/execute-parallel user-system plan.md 3
```

**选择最佳实现**：

```bash
# 查看结果
cat trees/user-system-*/RESULTS.md

# 测试每个实现
cd trees/user-system-1 && npm test

# 合并最佳方案
git checkout main
git merge user-system-2
```

### 优势

- **无冲突**：每个实例独立工作
- **多种方法**：比较不同实现
- **质量门**：仅考虑测试通过的实现
- **轻松集成**：合并最佳方案

---

## 🎯 快速命令参考

| Command                                        | Purpose              |
| ---------------------------------------------- | -------------------- |
| `/init`                                      | 生成初始 CLAUDE.md   |
| `/permissions`                               | 管理工具权限         |
| `/clear`                                     | 在任务之间清除上下文 |
| `/agents`                                    | 创建和管理子代理     |
| `/primer`                                    | 分析仓库结构         |
| `ESC`                                        | 中断 Claude          |
| `Shift+Tab`                                  | 进入规划模式         |
| `/generate-prp INITIAL.md`                   | 创建实现蓝图         |
| `/execute-prp PRPs/feature.md`               | 从蓝图实现           |
| `/prep-parallel [feature] [count]`           | 设置并行 worktrees   |
| `/execute-parallel [feature] [plan] [count]` | 运行并行实现         |
| `/fix-github-issue [number]`                 | 自动修复 GitHub 问题 |
| `/prep-parallel [feature] [count]`           | 设置并行 worktrees   |
| `/execute-parallel [feature] [plan] [count]` | 运行并行实现         |

---

## 📚 附加资源

- [Claude Code 文档](https://docs.anthropic.com/en/docs/claude-code)
- [Claude Code 最佳实践](https://www.anthropic.com/engineering/claude-code-best-practices)
- [MCP 服务器库](https://github.com/modelcontextprotocol)

---

## 🚀 下一步

1. **简单开始**：设置 CLAUDE.md 和基本权限
2. **添加斜杠命令**：为您的工作流创建自定义命令
3. **安装 MCP 服务器**：添加 Serena 以增强编码能力
4. **实现子代理**：为您的技术栈添加专家
5. **配置钩子**：自动化重复任务
6. **尝试并行开发**：尝试多种方法

记住：当您提供清晰的上下文、具体的示例和全面的验证时，Claude Code 最为强大。快乐编码！🎉
