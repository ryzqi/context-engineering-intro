# CLAUDE.md

本文档为 Claude Code 在此仓库中处理 Python 代码提供全面指导。

## 核心开发哲学

### KISS (Keep It Simple, Stupid)

简洁应该是设计中的关键目标。在可能的情况下，选择简单直接的解决方案而非复杂的方案。简单方案更易于理解、维护和调试。

### YAGNI (You Aren't Gonna Need It)

避免基于推测构建功能。仅在需要时才实现功能，而不是在预见到将来可能有用时。

### 设计原则

- **依赖倒置**：高层模块不应依赖于低层模块。两者都应该依赖于抽象。
- **开闭原则**：软件实体应该对扩展开放，对修改关闭。
- **单一职责**：每个函数、类和模块应该有一个明确的目的。
- **快速失败**：尽早检查潜在错误，并在出现问题时立即引发异常。

## 🧱 代码结构与模块化

### 文件和函数限制

- **永远不要创建超过500行代码的文件**。如果接近此限制，请通过拆分为模块进行重构。
- **函数应少于50行**，具有单一明确的职责。
- **类应少于100行**，表示单一概念或实体。
- **将代码组织成明确分离的模块**，按功能或职责分组。
- **行长度应限制在100个字符**，pyproject.toml中的ruff规则
- **执行Python命令时始终使用venv_linux**（虚拟环境），包括单元测试。

### 项目架构

遵循严格的垂直切片架构，测试与测试代码相邻：

```
src/project/
    __init__.py
    main.py
    tests/
        test_main.py
    conftest.py

    # 核心模块
    database/
        __init__.py
        connection.py
        models.py
        tests/
            test_connection.py
            test_models.py

    auth/
        __init__.py
        authentication.py
        authorization.py
        tests/
            test_authentication.py
            test_authorization.py

    # 功能切片
    features/
        user_management/
            __init__.py
            handlers.py
            validators.py
            tests/
                test_handlers.py
                test_validators.py

        payment_processing/
            __init__.py
            processor.py
            gateway.py
            tests/
                test_processor.py
                test_gateway.py
```

## 🛠️ 开发环境

### UV 包管理

本项目使用 UV 进行快速 Python 包和环境管理。

```bash
# 安装 UV (如果尚未安装)
curl -LsSf https://astral.sh/uv/install.sh   | sh

# 创建虚拟环境
uv venv

# 同步依赖
uv sync

# 添加包 ***永远不要直接在 pyproject.toml 中更新依赖***
# 始终使用 UV ADD
uv add requests

# 添加开发依赖
uv add --dev pytest ruff mypy

# 移除包
uv remove requests

# 在环境中运行命令
uv run python script.py
uv run pytest
uv run ruff check .

# 安装特定 Python 版本
uv python install 3.12
```

### 开发命令

```bash
# 运行所有测试
uv run pytest

# 使用详细输出运行特定测试
uv run pytest tests/test_module.py -v

# 使用覆盖率运行测试
uv run pytest --cov=src --cov-report=html

# 格式化代码
uv run ruff format .

# 检查代码风格
uv run ruff check .

# 自动修复代码风格问题
uv run ruff check --fix .

# 类型检查
uv run mypy src/

# 运行 pre-commit 钩子
uv run pre-commit run --all-files
```

## 📋 风格与约定

### Python 风格指南

- **遵循 PEP8** 并采用以下特定选择：
  - 行长度：100个字符（由 pyproject.toml 中的 Ruff 设置）
  - 字符串使用双引号
  - 多行结构中使用尾随逗号
- **始终为函数签名和类属性使用类型提示**
- **使用 `ruff format` 格式化**（比 Black 更快的替代方案）
- **使用 `pydantic` v2** 进行数据验证和设置管理

### 文档字符串标准

为所有公共函数、类和模块使用 Google 风格文档字符串：

```python
def calculate_discount(
    price: Decimal,
    discount_percent: float,
    min_amount: Decimal = Decimal("0.01")
) -> Decimal:
    """
    计算产品折扣价格。

    Args:
        price: 产品原始价格
        discount_percent: 折扣百分比 (0-100)
        min_amount: 最终价格的最低允许值

    Returns:
        应用折扣后的最终价格

    Raises:
        ValueError: 如果 discount_percent 不在 0 和 100 之间
        ValueError: 如果最终价格低于 min_amount

    Example:
        >>> calculate_discount(Decimal("100"), 20)
        Decimal('80.00')
    """
```

### 命名约定

- **变量和函数**：`snake_case`
- **类**：`PascalCase`
- **常量**：`UPPER_SNAKE_CASE`
- **私有属性/方法**：`_leading_underscore`
- **类型别名**：`PascalCase`
- **枚举值**：`UPPER_SNAKE_CASE`

## 🧪 测试策略

### 测试驱动开发 (TDD)

1. **先编写测试** - 在实现之前定义预期行为
2. **确保测试失败** - 确保测试确实测试了某些内容
3. **编写最少代码** - 仅编写使测试通过的代码
4. **重构** - 在保持测试通过的同时改进代码
5. **重复** - 一次一个测试

### 测试最佳实践

```python
# 始终对设置使用 pytest fixtures
import pytest
from datetime import datetime

@pytest.fixture
def sample_user():
    """为测试提供示例用户。"""
    return User(
        id=123,
        name="Test User",
        email="test@example.com",
        created_at=datetime.now()
    )

# 使用描述性测试名称
def test_user_can_update_email_when_valid(sample_user):
    """测试用户可以使用有效输入更新其电子邮件。"""
    new_email = "newemail@example.com"
    sample_user.update_email(new_email)
    assert sample_user.email == new_email

# 测试边界情况和错误条件
def test_user_update_email_fails_with_invalid_format(sample_user):
    """测试拒绝无效的电子邮件格式。"""
    with pytest.raises(ValidationError) as exc_info:
        sample_user.update_email("not-an-email")
    assert "Invalid email format" in str(exc_info.value)
```

### 测试组织

- 单元测试：隔离测试单个函数/方法
- 集成测试：测试组件交互
- 端到端测试：测试完整用户工作流程
- 将测试文件放在测试代码旁边
- 使用 `conftest.py` 用于共享 fixtures
- 目标是 80%+ 代码覆盖率，但关注关键路径

## 🚨 错误处理

### 异常最佳实践

```python
# 为您的领域创建自定义异常
class PaymentError(Exception):
    """与支付相关的错误的基础异常。"""
    pass

class InsufficientFundsError(PaymentError):
    """当账户资金不足时引发。"""
    def __init__(self, required: Decimal, available: Decimal):
        self.required = required
        self.available = available
        super().__init__(
            f"Insufficient funds: required {required}, available {available}"
        )

# 使用特定的异常处理
try:
    process_payment(amount)
except InsufficientFundsError as e:
    logger.warning(f"Payment failed: {e}")
    return PaymentResult(success=False, reason="insufficient_funds")
except PaymentError as e:
    logger.error(f"Payment error: {e}")
    return PaymentResult(success=False, reason="payment_error")

# 使用上下文管理器管理资源
from contextlib import contextmanager

@contextmanager
def database_transaction():
    """为数据库操作提供事务范围。"""
    conn = get_connection()
    trans = conn.begin_transaction()
    try:
        yield conn
        trans.commit()
    except Exception:
        trans.rollback()
        raise
    finally:
        conn.close()
```

### 日志策略

```python
import logging
from functools import wraps

# 配置结构化日志
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

logger = logging.getLogger(__name__)

# 记录函数进入/退出以进行调试
def log_execution(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        logger.debug(f"Entering {func.__name__}")
        try:
            result = func(*args, **kwargs)
            logger.debug(f"Exiting {func.__name__} successfully")
            return result
        except Exception as e:
            logger.exception(f"Error in {func.__name__}: {e}")
            raise
    return wrapper
```

## 🔧 配置管理

### 环境变量和设置

```python
from pydantic_settings import BaseSettings
from functools import lru_cache

class Settings(BaseSettings):
    """具有验证的应用程序设置。"""
    app_name: str = "MyApp"
    debug: bool = False
    database_url: str
    redis_url: str = "redis://localhost:6379"
    api_key: str
    max_connections: int = 100

    class Config:
        env_file = ".env"
        env_file_encoding = "utf-8"
        case_sensitive = False

@lru_cache()
def get_settings() -> Settings:
    """获取缓存的设置实例。"""
    return Settings()

# 使用
settings = get_settings()
```

## 🏗️ 数据模型和验证

### Pydantic 模型示例 (使用 pydantic v2 严格模式)

```python
from pydantic import BaseModel, Field, validator, EmailStr
from datetime import datetime
from typing import Optional, List
from decimal import Decimal

class ProductBase(BaseModel):
    """具有公共字段的基本产品模型。"""
    name: str = Field(..., min_length=1, max_length=255)
    description: Optional[str] = None
    price: Decimal = Field(..., gt=0, decimal_places=2)
    category: str
    tags: List[str] = []

    @validator('price')
    def validate_price(cls, v):
        if v > Decimal('1000000'):
            raise ValueError('Price cannot exceed 1,000,000')
        return v

    class Config:
        json_encoders = {
            Decimal: str,
            datetime: lambda v: v.isoformat()
        }

class ProductCreate(ProductBase):
    """用于创建新产品的模型。"""
    pass

class ProductUpdate(BaseModel):
    """用于更新产品的模型 - 所有字段可选。"""
    name: Optional[str] = Field(None, min_length=1, max_length=255)
    description: Optional[str] = None
    price: Optional[Decimal] = Field(None, gt=0, decimal_places=2)
    category: Optional[str] = None
    tags: Optional[List[str]] = None

class Product(ProductBase):
    """包含数据库字段的完整产品模型。"""
    id: int
    created_at: datetime
    updated_at: datetime
    is_active: bool = True

    class Config:
        from_attributes = True  # 启用 ORM 模式
```

## 🔄 Git 工作流程

### 分支策略

- `main` - 可用于生产的代码
- `develop` - 功能集成分支
- `feature/*` - 新功能
- `fix/*` - Bug 修复
- `docs/*` - 文档更新
- `refactor/*` - 代码重构
- `test/*` - 测试添加或修复

### 提交信息格式

提交信息中永远不要包含 claude code 或由 claude code 编写

```
<type>(<scope>): <subject>

<body>

<footer>
```

类型：feat, fix, docs, style, refactor, test, chore

示例：

```

feat(auth): 添加双因素认证

- 实现 TOTP 生成和验证
- 为身份验证应用程序添加 QR 码生成
- 使用 2FA 字段更新用户模型

Closes #123

```

## 🗄️ 数据库命名标准

### 实体特定主键

所有数据库表使用实体特定主键以确保清晰和一致性：

```sql
-- ✅ 标准化：实体特定主键
sessions.session_id UUID PRIMARY KEY
leads.lead_id UUID PRIMARY KEY
messages.message_id UUID PRIMARY KEY
daily_metrics.daily_metric_id UUID PRIMARY KEY
agencies.agency_id UUID PRIMARY KEY
```

### 字段命名约定

```sql
-- 主键：{entity}_id
session_id, lead_id, message_id

-- 外键：{referenced_entity}_id
session_id REFERENCES sessions(session_id)
agency_id REFERENCES agencies(agency_id)

-- 时间戳：{action}_at
created_at, updated_at, started_at, expires_at

-- 布尔值：is_{state}
is_connected, is_active, is_qualified

-- 计数：{entity}_count
message_count, lead_count, notification_count

-- 持续时间：{property}_{unit}
duration_seconds, timeout_minutes
```

### 仓库模式自动推导

增强的 BaseRepository 自动推导表名和主键：

```python
# ✅ 标准化：基于约定的仓库
class LeadRepository(BaseRepository[Lead]):
    def __init__(self):
        super().__init__()  # 自动推导 "leads" 和 "lead_id"

class SessionRepository(BaseRepository[AvatarSession]):
    def __init__(self):
        super().__init__()  # 自动推导 "sessions" 和 "session_id"
```

**优点**:

- ✅ 自我文档化的模式
- ✅ 清晰的外键关系
- ✅ 消除仓库方法重写
- ✅ 与实体命名模式一致

### 模型-数据库对齐

模型与数据库字段完全匹配，以消除字段映射复杂性：

```python
# ✅ 标准化：模型与数据库完全匹配
class Lead(BaseModel):
    lead_id: UUID = Field(default_factory=uuid4)  # 匹配数据库字段
    session_id: UUID                               # 匹配数据库字段
    agency_id: str                                 # 匹配数据库字段
    created_at: datetime = Field(default_factory=lambda: datetime.now(UTC))

    model_config = ConfigDict(
        use_enum_values=True,
        populate_by_name=True,
        alias_generator=None  # 使用精确字段名
    )
```

### API 路由标准

```python
# ✅ 标准化：RESTful 且参数命名一致
router = APIRouter(prefix="/api/v1/leads", tags=["leads"])

@router.get("/{lead_id}")           # GET /api/v1/leads/{lead_id}
@router.put("/{lead_id}")           # PUT /api/v1/leads/{lead_id}
@router.delete("/{lead_id}")        # DELETE /api/v1/leads/{lead_id}

# 子资源
@router.get("/{lead_id}/messages")  # GET /api/v1/leads/{lead_id}/messages
@router.get("/agency/{agency_id}")  # GET /api/v1/leads/agency/{agency_id}
```

有关完整命名标准，请参阅 [NAMING_CONVENTIONS.md](./NAMING_CONVENTIONS.md)。

## 📝 文档标准

### 代码文档

- 每个模块应有解释其用途的文档字符串
- 公共函数必须有完整的文档字符串
- 复杂逻辑应有带 `# Reason:` 前缀的内联注释
- 保持 README.md 更新包含设置说明和示例
- 维护 CHANGELOG.md 用于版本历史

### API 文档

```python
from fastapi import APIRouter, HTTPException, status
from typing import List

router = APIRouter(prefix="/products", tags=["products"])

@router.get(
    "/",
    response_model=List[Product],
    summary="列出所有产品",
    description="检索所有活动产品的分页列表"
)
async def list_products(
    skip: int = 0,
    limit: int = 100,
    category: Optional[str] = None
) -> List[Product]:
    """
    检索带有可选过滤的产品。

    - **skip**: 要跳过的项目数（用于分页）
    - **limit**: 要返回的最大项目数
    - **category**: 按产品类别过滤
    """
    # 实现代码
```

## 🚀 性能考虑

### 优化指南

- 优化前先进行性能分析 - 使用 `cProfile` 或 `py-spy`
- 使用 `lru_cache` 处理昂贵的计算
- 对大型数据集优先使用生成器
- 对 I/O 密集型操作使用 `asyncio`
- 对 CPU 密集型任务考虑 `multiprocessing`
- 适当地缓存数据库查询

### 优化示例

```python
from functools import lru_cache
import asyncio
from typing import AsyncIterator

@lru_cache(maxsize=1000)
def expensive_calculation(n: int) -> int:
    """缓存昂贵计算的结果。"""
    # 复杂计算
    return result

async def process_large_dataset() -> AsyncIterator[dict]:
    """处理大型数据集而不将所有内容加载到内存中。"""
    async with aiofiles.open('large_file.json', mode='r') as f:
        async for line in f:
            data = json.loads(line)
            # 处理并逐个返回项目
            yield process_item(data)
```

## 🛡️ 安全最佳实践

### 安全指南

- 永远不要提交机密 - 使用环境变量
- 使用 Pydantic 验证所有用户输入
- 对数据库操作使用参数化查询
- 为 API 实现速率限制
- 使用 `uv` 保持依赖项更新
- 对所有外部通信使用 HTTPS
- 实现适当的认证和授权

### 安全实现示例

```python
from passlib.context import CryptContext
import secrets

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def hash_password(password: str) -> str:
    """使用 bcrypt 哈希密码。"""
    return pwd_context.hash(password)

def verify_password(plain_password: str, hashed_password: str) -> bool:
    """验证密码与其哈希值。"""
    return pwd_context.verify(plain_password, hashed_password)

def generate_secure_token(length: int = 32) -> str:
    """生成加密安全的随机令牌。"""
    return secrets.token_urlsafe(length)
```

## 🔍 调试工具

### 调试命令

```bash
# 使用 ipdb 进行交互式调试
uv add --dev ipdb
# 添加断点：import ipdb; ipdb.set_trace()

# 内存分析
uv add --dev memory-profiler
uv run python -m memory_profiler script.py

# 行分析
uv add --dev line-profiler
# 为函数添加 @profile 装饰器

# 使用 rich traceback 调试
uv add --dev rich
# 在代码中：from rich.traceback import install; install()
```

## 📊 监控和可观测性

### 结构化日志

```python
import structlog

logger = structlog.get_logger()

# 使用上下文记录日志
logger.info(
    "payment_processed",
    user_id=user.id,
    amount=amount,
    currency="USD",
    processing_time=processing_time
)
```

## 📚 有用资源

### 基本工具

- UV 文档：https://github.com/astral-sh/uv
- Ruff：https://github.com/astral-sh/ruff
- Pytest：https://docs.pytest.org/
- Pydantic：https://docs.pydantic.dev/
- FastAPI：https://fastapi.tiangolo.com/

### Python 最佳实践

- PEP 8：https://pep8.org/
- PEP 484 (类型提示)：https://www.python.org/dev/peps/pep-0484/
- The Hitchhiker's Guide to Python：https://docs.python-guide.org/

## ⚠️ 重要注意事项

- **永远不要假设或猜测** - 有疑问时，请寻求澄清
- **在使用前始终验证文件路径和模块名称**
- **在添加新模式或依赖项时更新 CLAUDE.md**
- **测试您的代码** - 没有测试的功能是不完整的
- **记录您的决策** - 未来的开发者（包括您自己）会感谢您

## 🔍 搜索命令要求

**关键**：始终使用 `rg` (ripgrep) 代替传统的 `grep` 和 `find` 命令：

```bash
# ❌ 不要使用 grep
grep -r "pattern" .

# ✅ 使用 rg 代替
rg "pattern"

# ❌ 不要使用带有名称的 find
find . -name "*.py"

# ✅ 使用带有文件过滤的 rg
rg --files | rg "\.py$"
# 或
rg --files -g "*.py"
```

**执行规则**：

```
(
    r"^grep\b(?!.*\|)",
    "使用 'rg' (ripgrep) 代替 'grep' 以获得更好的性能和功能",
),
(
    r"^find\s+\S+\s+-name\b",
    "使用 'rg --files | rg pattern' 或 'rg --files -g pattern' 代替 'find -name' 以获得更好的性能",
),
```

## 🚀 GitHub Flow 工作流程摘要

main (protected) ←── PR ←── feature/your-feature
↓ ↑
deploy development

### 日常工作流程：

1. git checkout main && git pull origin main
2. git checkout -b feature/new-feature
3. 进行更改 + 测试
4. git push origin feature/new-feature
5. 创建 PR → 审查 → 合并到 main

---

_本文档是一份动态指南。随着项目发展和新模式出现，请更新它。_
