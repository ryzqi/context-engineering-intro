name: "多智能体系统：带有邮件草稿子智能体的研究智能体"
description: |

## 目的

构建一个 Pydantic AI 多智能体系统，其中主要的研究智能体使用 Brave Search API，并拥有一个邮件草稿智能体（使用 Gmail API）作为其工具。这展示了带有外部 API 集成的“智能体即工具”模式。

## 核心原则

1. **上下文为王**：包含所有必要的文档、示例和注意事项
2. **验证循环**：提供 AI 可以运行和修复的可执行测试/代码检查
3. **信息密集**：使用代码库中的关键词和模式
4. **渐进成功**：从简单开始，验证，然后增强

---

## 目标

创建一个生产就绪的多智能体系统，用户可以通过 CLI 研究主题，研究智能体可以将邮件起草任务委托给邮件草稿智能体。该系统应支持多个 LLM 提供商，并安全地处理 API 身份验证。

## 原因

- **商业价值**：自动化研究和邮件起草工作流
- **集成**：展示先进的 Pydantic AI 多智能体模式
- **解决的问题**：减少基于研究的电子邮件通信的人工工作

## 功能

一个基于 CLI 的应用程序，其中：

- 用户输入研究查询
- 研究智能体使用 Brave API 进行搜索
- 研究智能体可以调用邮件草稿智能体来创建 Gmail 草稿
- 结果实时流式传输回用户

### 成功标准

- [ ] 研究智能体通过 Brave API 成功搜索
- [ ] 邮件智能体通过正确的身份验证创建 Gmail 草稿
- [ ] 研究智能体可以作为工具调用邮件智能体
- [ ] CLI 提供带有工具可见性的流式响应
- [ ] 所有测试通过且代码符合质量标准

## 所有需要的上下文

### 文档与参考

```yaml
# 必读 - 将这些包含在你的上下文窗口中
- url: https://ai.pydantic.dev/agents/
  why: 核心智能体创建模式
  
- url: https://ai.pydantic.dev/multi-agent-applications/
  why: 多智能体系统模式，特别是“智能体即工具”
  
- url: https://developers.google.com/gmail/api/guides/sending
  why: Gmail API 身份验证和草稿创建
  
- url: https://api-dashboard.search.brave.com/app/documentation
  why: Brave Search API REST 端点
  
- file: examples/agent/agent.py
  why: 智能体创建、工具注册、依赖项的模式
  
- file: examples/agent/providers.py
  why: 多提供商 LLM 配置模式
  
- file: examples/cli.py
  why: 带有流式响应和工具可见性的 CLI 结构

- url: https://github.com/googleworkspace/python-samples/blob/main/gmail/snippet/send%20mail/create_draft.py
  why: 官方 Gmail 草稿创建示例
```

### 当前代码库树

```bash
.
├── examples/
│   ├── agent/
│   │   ├── agent.py
│   │   ├── providers.py
│   │   └── ...
│   └── cli.py
├── PRPs/
│   └── templates/
│       └── prp_base.md
├── INITIAL.md
├── CLAUDE.md
└── requirements.txt
```

### 期望的代码库树（包含待添加文件）

```bash
.
├── agents/
│   ├── __init__.py               # 包初始化
│   ├── research_agent.py         # 使用 Brave Search 的主智能体
│   ├── email_agent.py           # 具有 Gmail 功能的子智能体
│   ├── providers.py             # LLM 提供商配置
│   └── models.py                # 用于数据验证的 Pydantic 模型
├── tools/
│   ├── __init__.py              # 包初始化
│   ├── brave_search.py          # Brave Search API 集成
│   └── gmail_tool.py            # Gmail API 集成
├── config/
│   ├── __init__.py              # 包初始化
│   └── settings.py              # 环境与配置管理
├── tests/
│   ├── __init__.py              # 包初始化
│   ├── test_research_agent.py   # 研究智能体测试
│   ├── test_email_agent.py      # 邮件智能体测试
│   ├── test_brave_search.py     # Brave 搜索工具测试
│   ├── test_gmail_tool.py       # Gmail 工具测试
│   └── test_cli.py              # CLI 测试
├── cli.py                       # CLI 接口
├── .env.example                 # 环境变量模板
├── requirements.txt             # 更新的依赖项
├── README.md                    # 综合文档
└── credentials/.gitkeep         # 用于存放 Gmail 凭证的目录
```

### 已知陷阱与库的特性

```python
# 关键：Pydantic AI 全程需要异步 - 不要在异步上下文中使用同步函数
# 关键：Gmail API 首次运行时需要 OAuth2 流程 - 需要 credentials.json
# 关键：Brave API 有速率限制 - 免费套餐每月 2000 次请求
# 关键：“智能体即工具”模式需要传递 ctx.usage 以进行 token 跟踪
# 关键：Gmail 草稿需要使用正确的 MIME 格式进行 base64 编码
# 关键：始终使用绝对导入以获得更清晰的代码
# 关键：将敏感凭证存储在 .env 中，绝不提交它们
```

## 实现蓝图

### 数据模型和结构

```python
# models.py - 核心数据结构
from pydantic import BaseModel, Field
from typing import List, Optional
from datetime import datetime

class ResearchQuery(BaseModel):
    query: str = Field(..., description="要研究的课题")
    max_results: int = Field(10, ge=1, le=50)
    include_summary: bool = Field(True)

class BraveSearchResult(BaseModel):
    title: str
    url: str
    description: str
    score: float = Field(0.0, ge=0.0, le=1.0)

class EmailDraft(BaseModel):
    to: List[str] = Field(..., min_items=1)
    subject: str = Field(..., min_length=1)
    body: str = Field(..., min_length=1)
    cc: Optional[List[str]] = None
    bcc: Optional[List[str]] = None

class ResearchEmailRequest(BaseModel):
    research_query: str
    email_context: str = Field(..., description="用于生成邮件的上下文")
    recipient_email: str
```

### 待完成任务列表

```yaml
任务 1：设置配置和环境
创建 config/settings.py:
  - 模式：像示例中使用 os.getenv 一样使用 pydantic-settings
  - 加载带有默认值的环境变量
  - 验证所需的 API 密钥是否存在

创建 .env.example:
  - 包含所有必需的环境变量及其描述
  - 遵循 examples/README.md 中的模式

任务 2：实现 Brave 搜索工具
创建 tools/brave_search.py:
  - 模式：像 examples/agent/tools.py 一样的异步函数
  - 使用 httpx 的简单 REST 客户端（已在 requirements 中）
  - 优雅地处理速率限制和错误
  - 返回结构化的 BraveSearchResult 模型

任务 3：实现 Gmail 工具
创建 tools/gmail_tool.py:
  - 模式：遵循 Gmail 快速入门中的 OAuth2 流程
  - 将 token.json 存储在 credentials/ 目录中
  - 使用正确的 MIME 编码创建草稿
  - 自动处理身份验证刷新

任务 4：创建邮件草稿智能体
创建 agents/email_agent.py:
  - 模式：遵循 examples/agent/agent.py 结构
  - 使用带有 deps_type 模式的 Agent
  - 将 gmail_tool 注册为 @agent.tool
  - 返回 EmailDraft 模型

任务 5：创建研究智能体
创建 agents/research_agent.py:
  - 模式：遵循 Pydantic AI 文档中的多智能体模式
  - 将 brave_search 注册为工具
  - 将 email_agent.run() 注册为工具
  - 使用 RunContext 进行依赖注入

任务 6：实现 CLI 接口
创建 cli.py:
  - 模式：遵循 examples/cli.py 的流式处理模式
  - 带有工具可见性的颜色编码输出
  - 使用 asyncio.run() 正确处理异步
  - 用于会话上下文的会话管理

任务 7：添加综合测试
创建 tests/:
  - 模式：镜像示例的测试结构
  - 模拟外部 API 调用
  - 测试正常路径、边界情况、错误
  - 确保 80%+ 的覆盖率

任务 8：创建文档
创建 README.md:
  - 模式：遵循 examples/README.md 的结构
  - 包括设置、安装、使用说明
  - API 密钥配置步骤
  - 架构图
```

### 每个任务的伪代码

```python
# 任务 2：Brave 搜索工具
async def search_brave(query: str, api_key: str, count: int = 10) -> List[BraveSearchResult]:
    # 模式：像示例使用 aiohttp 一样使用 httpx
    async with httpx.AsyncClient() as client:
        headers = {"X-Subscription-Token": api_key}
        params = {"q": query, "count": count}
    
        # 注意：如果 API 密钥无效，Brave API 会返回 401
        response = await client.get(
            "https://api.search.brave.com/res/v1/web/search",
            headers=headers,
            params=params,
            timeout=30.0  # 关键：设置超时以避免挂起
        )
    
        # 模式：结构化错误处理
        if response.status_code != 200:
            raise BraveAPIError(f"API returned {response.status_code}")
    
        # 使用 Pydantic 解析和验证
        data = response.json()
        return [BraveSearchResult(**result) for result in data.get("web", {}).get("results", [])]

# 任务 5：将邮件智能体作为工具的研究智能体
@research_agent.tool
async def create_email_draft(
    ctx: RunContext[AgentDependencies],
    recipient: str,
    subject: str,
    context: str
) -> str:
    """根据研究上下文创建邮件草稿。"""
    # 关键：传递 usage 以进行 token 跟踪
    result = await email_agent.run(
        f"Create an email to {recipient} about: {context}",
        deps=EmailAgentDeps(subject=subject),
        usage=ctx.usage  # 来自多智能体文档的模式
    )
  
    return f"草稿已创建，ID: {result.data}"
```

### 集成点

```yaml
ENVIRONMENT:
  - add to: .env
  - vars: |
      # LLM 配置
      LLM_PROVIDER=openai
      LLM_API_KEY=sk-...
      LLM_MODEL=gpt-4
  
      # Brave 搜索
      BRAVE_API_KEY=BSA...
  
      # Gmail (credentials.json 的路径)
      GMAIL_CREDENTIALS_PATH=./credentials/credentials.json
  
CONFIG:
  - Gmail OAuth：首次运行会打开浏览器进行授权
  - Token 存储：./credentials/token.json (自动创建)
  
DEPENDENCIES:
  - 更新 requirements.txt，添加：
    - google-api-python-client
    - google-auth-httplib2
    - google-auth-oauthlib
```

## 验证循环

### 级别 1：语法与风格

```bash
# 首先运行这些 - 在继续之前修复任何错误
ruff check . --fix              # 自动修复风格问题
mypy .                          # 类型检查

# 预期：没有错误。如果出现错误，请阅读并修复。
```

### 级别 2：单元测试

```python
# test_research_agent.py
async def test_research_with_brave():
    """测试研究智能体是否正确搜索"""
    agent = create_research_agent()
    result = await agent.run("AI safety research")
    assert result.data
    assert len(result.data) > 0

async def test_research_creates_email():
    """测试研究智能体是否可以调用邮件智能体"""
    agent = create_research_agent()
    result = await agent.run(
        "Research AI safety and draft email to john@example.com"
    )
    assert "draft_id" in result.data

# test_email_agent.py  
def test_gmail_authentication(monkeypatch):
    """测试 Gmail OAuth 流程处理"""
    monkeypatch.setenv("GMAIL_CREDENTIALS_PATH", "test_creds.json")
    tool = GmailTool()
    assert tool.service is not None

async def test_create_draft():
    """测试草稿创建及正确编码"""
    agent = create_email_agent()
    result = await agent.run(
        "Create email to test@example.com about AI research"
    )
    assert result.data.get("draft_id")
```

```bash
# 迭代运行测试直到通过：
pytest tests/ -v --cov=agents --cov=tools --cov-report=term-missing

# 如果失败：调试特定测试，修复代码，重新运行
```

### 级别 3：集成测试

```bash
# 测试 CLI 交互
python cli.py

# 预期交互：
# 你：研究最新的 AI 安全进展
# 🤖 助手：[流式传输研究结果]
# 🛠 使用的工具：
#   1. brave_search (query='AI safety developments', limit=10)
#
# 你：就此为 john@example.com 创建一封邮件草稿
# 🤖 助手：[创建草稿]
# 🛠 使用的工具：
#   1. create_email_draft (recipient='john@example.com', ...)

# 检查 Gmail 草稿箱中已创建的草稿
```

## 最终验证清单

- [ ] 所有测试通过：`pytest tests/ -v`
- [ ] 没有代码检查错误：`ruff check .`
- [ ] 没有类型错误：`mypy .`
- [ ] Gmail OAuth 流程正常工作（浏览器打开，token 已保存）
- [ ] Brave Search 返回结果
- [ ] 研究智能体成功调用邮件智能体
- [ ] CLI 以工具可见性流式传输响应
- [ ] 错误情况得到优雅处理
- [ ] README 包含清晰的设置说明
- [ ] .env.example 包含所有必需的变量

---

## 需要避免的反模式

- ❌ 不要硬编码 API 密钥 - 使用环境变量
- ❌ 不要在异步智能体上下文中使用同步函数
- ❌ 不要跳过 Gmail 的 OAuth 流程设置
- ❌ 不要忽略 API 的速率限制
- ❌ 在多智能体调用中不要忘记传递 ctx.usage
- ❌ 不要提交 credentials.json 或 token.json 文件

## 置信度分数：9/10

高置信度原因：

- 代码库中有清晰的示例可供遵循
- 外部 API 文档齐全
- 已建立的多智能体系统模式
- 全面的验证关卡

在 Gmail OAuth 首次设置的用户体验上存在轻微不确定性，但文档提供了明确的指导。
