# 上下文工程模板

一个用于开始学习上下文工程的综合模板——上下文工程是一门为 AI 编码助手设计上下文的学科，以便它们拥有端到端完成工作所需的信息。

> **上下文工程比提示工程好 10 倍，比凭感觉编码好 100 倍。**

## 🚀 快速入门

```bash
# 1. 克隆此模板
git clone https://github.com/coleam00/Context-Engineering-Intro.git
cd Context-Engineering-Intro

# 2. 设置你的项目规则（可选 - 已提供模板）
# 编辑 CLAUDE.md 以添加你的项目特定指南

# 3. 添加示例（强烈推荐）
# 将相关代码示例放入 examples/ 文件夹

# 4. 创建你的初始功能请求
# 编辑 INITIAL.md 并填写你的功能需求

# 5. 生成一个全面的 PRP (产品需求提示)
# 在 Claude Code 中运行：
/generate-prp INITIAL.md

# 6. 执行 PRP 以实现你的功能
# 在 Claude Code 中运行：
/execute-prp PRPs/your-feature-name.md
```

## 📚 目录

- [什么是上下文工程？](#什么是上下文工程)
- [模板结构](#模板结构)
- [分步指南](#分步指南)
- [编写有效的 INITIAL.md 文件](#编写有效的-initialmd-文件)
- [PRP 工作流](#the-prp-工作流)
- [有效使用示例](#有效使用示例)
- [最佳实践](#最佳实践)

## 什么是上下文工程？

上下文工程代表了从传统提示工程的范式转变：

### 提示工程 vs 上下文工程

**提示工程：**

- 专注于巧妙的措辞和特定的表述
- 局限于你如何描述一个任务
- 就像给某人一张便利贴

**上下文工程：**

- 一个提供全面上下文的完整系统
- 包括文档、示例、规则、模式和验证
- 就像编写一个包含所有细节的完整剧本

### 为什么上下文工程很重要

1. **减少 AI 失败**：大多数 agent 的失败不是模型失败——而是上下文失败
2. **确保一致性**：AI 遵循你的项目模式和惯例
3. **实现复杂功能**：在有适当上下文的情况下，AI 可以处理多步骤的实现
4. **自我修正**：验证循环允许 AI 修复自己的错误

## 模板结构

```
context-engineering-intro/
├── .claude/
│   ├── commands/
│   │   ├── generate-prp.md    # 生成全面的 PRP
│   │   └── execute-prp.md     # 执行 PRP 以实现功能
│   └── settings.local.json    # Claude Code 权限
├── PRPs/
│   ├── templates/
│   │   └── prp_base.md       # PRP 的基础模板
│   └── EXAMPLE_multi_agent_prp.md  # 一个完整 PRP 的示例
├── examples/                  # 你的代码示例（至关重要！）
├── CLAUDE.md                 # AI 助手的全局规则
├── INITIAL.md               # 功能请求的模板
├── INITIAL_EXAMPLE.md       # 功能请求示例
└── README.md                # 本文件
```

这个模板没有侧重于 RAG 和带有上下文工程的工具，因为我很快会有更多相关内容。;)

## 分步指南

### 1. 设置全局规则 (CLAUDE.md)

`CLAUDE.md` 文件包含 AI 助手在每次对话中都会遵循的项目范围规则。模板包括：

- **项目感知**：阅读规划文档，检查任务
- **代码结构**：文件大小限制，模块组织
- **测试要求**：单元测试模式，覆盖率期望
- **风格约定**：语言偏好，格式规则
- **文档标准**：文档字符串格式，注释实践

**你可以直接使用提供的模板，也可以为你的项目进行定制。**

### 2. 创建你的初始功能请求

编辑 `INITIAL.md` 来描述你想构建什么：

```markdown
## 功能特性：
[描述你想构建什么 - 具体说明功能和需求]

## 示例：
[列出 examples/ 文件夹中的任何示例文件，并解释应如何使用它们]

## 文档：
[包含相关文档、API 或 MCP 服务器资源的链接]

## 其他注意事项：
[提及任何陷阱、特定要求或 AI 助手通常会忽略的事情]
```

**查看 `INITIAL_EXAMPLE.md` 获取完整示例。**

### 3. 生成 PRP

PRP (Product Requirements Prompts) 是全面的实施蓝图，包括：

- 完整的上下文和文档
- 带有验证的实施步骤
- 错误处理模式
- 测试要求

它们类似于 PRD (Product Requirements Documents)，但其制作更具体，用于指导 AI 编码助手。

在 Claude Code 中运行：

```bash
/generate-prp INITIAL.md
```

**注意：** 斜杠命令是在 `.claude/commands/` 中定义的自定义命令。你可以查看它们的实现：

- `.claude/commands/generate-prp.md` - 查看它如何研究和创建 PRP
- `.claude/commands/execute-prp.md` - 查看它如何从 PRP 实现功能

这些命令中的 `$ARGUMENTS` 变量接收你在命令名称后传递的任何内容（例如 `INITIAL.md` 或 `PRPs/your-feature.md`）。

该命令将：

1. 读取你的功能请求
2. 研究代码库以寻找模式
3. 搜索相关文档
4. 在 `PRPs/your-feature-name.md` 中创建一个全面的 PRP

### 4. 执行 PRP

生成后，执行 PRP 以实现你的功能：

```bash
/execute-prp PRPs/your-feature-name.md
```

AI 编码助手将：

1. 从 PRP 中读取所有上下文
2. 创建一个详细的实施计划
3. 执行每个步骤并进行验证
4. 运行测试并修复任何问题
5. 确保所有成功标准都已满足

## 编写有效的 INITIAL.md 文件

### 关键部分解释

**功能特性**：要具体和全面

- ❌ "构建一个网络爬虫"
- ✅ "使用 BeautifulSoup 构建一个异步网络爬虫，用于从电子商务网站提取产品数据，处理速率限制，并将结果存储在 PostgreSQL 中"

**示例**：利用 examples/ 文件夹

- 将相关代码模式放在 `examples/` 中
- 引用要遵循的特定文件和模式
- 解释哪些方面应该被模仿

**文档**：包含所有相关资源

- API 文档 URL
- 库指南
- MCP 服务器文档
- 数据库模式

**其他注意事项**：捕捉重要细节

- 身份验证要求
- 速率限制或配额
- 常见陷阱
- 性能要求

## PRP 工作流

### /generate-prp 是如何工作的

该命令遵循以下过程：

1. **研究阶段**
   - 分析你的代码库以寻找模式
   - 搜索类似的实现
   - 识别要遵循的惯例
2. **文档收集**
   - 获取相关的 API 文档
   - 包括库文档
   - 添加陷阱和怪癖
3. **蓝图创建**
   - 创建分步实施计划
   - 包括验证关卡
   - 添加测试要求
4. **质量检查**
   - 评估置信度（1-10）
   - 确保包含所有上下文

### /execute-prp 是如何工作的

1. **加载上下文**：读取整个 PRP
2. **计划**：使用 TodoWrite 创建详细的任务列表
3. **执行**：实现每个组件
4. **验证**：运行测试和代码检查
5. **迭代**：修复发现的任何问题
6. **完成**：确保所有要求都已满足

查看 `PRPs/EXAMPLE_multi_agent_prp.md` 获取一个生成的完整示例。

## 有效使用示例

`examples/` 文件夹对于成功**至关重要**。当 AI 编码助手可以看到要遵循的模式时，它们的表现会好得多。

### 示例中应包含什么

1. **代码结构模式**
   - 你如何组织模块
   - 导入惯例
   - 类/函数模式
2. **测试模式**
   - 测试文件结构
   - Mock 方法
   - 断言风格
3. **集成模式**
   - API 客户端实现
   - 数据库连接
   - 身份验证流程
4. **CLI 模式**
   - 参数解析
   - 输出格式
   - 错误处理

### 示例结构

```
examples/
├── README.md           # 解释每个示例演示了什么
├── cli.py             # CLI 实现模式
├── agent/             # Agent 架构模式
│   ├── agent.py      # Agent 创建模式
│   ├── tools.py      # 工具实现模式
│   └── providers.py  # 多 provider 模式
└── tests/            # 测试模式
    ├── test_agent.py # 单元测试模式
    └── conftest.py   # Pytest 配置
```

## 最佳实践

### 1. 在 INITIAL.md 中要明确

- 不要假设 AI 知道你的偏好
- 包括具体的要求和约束
- 大量引用示例

### 2. 提供全面的示例

- 更多示例 = 更好的实现
- 展示该做什么和不该做什么
- 包括错误处理模式

### 3. 使用验证关卡

- PRP 包含必须通过的测试命令
- AI 会迭代直到所有验证都成功
- 这确保了代码在第一次尝试时就能工作

### 4. 利用文档

- 包括官方 API 文档
- 添加 MCP 服务器资源
- 引用特定的文档部分

### 5. 自定义 CLAUDE.md

- 添加你的惯例
- 包括项目特定的规则
- 定义编码标准

## 资源

- [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code)
- [Context Engineering Best Practices](https://www.philschmid.de/context-engineering)
