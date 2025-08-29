name: "Base PRP Template v2 - 上下文丰富的验证循环"
description: |

## 目的

为 AI 代理提供模板，通过充分上下文和自验证能力实现功能，通过迭代优化达成可工作代码。

## 核心原则

1. **上下文至上**：包含所有必要文档、示例和注意事项
2. **验证循环**：提供 AI 可执行的测试/检查以修复问题
3. **信息密集**：使用代码库中的关键词和模式
4. **渐进式成功**：从简单开始，验证后再增强
5. **全局规则**：严格遵守 CLAUDE.md 中的所有规则

---

## 目标

[需构建的具体内容 - 明确最终状态和需求]

## 原因

- [业务价值和用户影响]
- [与现有功能的集成]
- [解决的问题及受益对象]

## 内容

[用户可见行为和技术要求]

### 成功标准

- [ ] [具体可衡量的结果]

## 所需全部上下文

### 文档与参考 (列出实现功能所需的所有上下文)

```yaml
# 必读 - 将这些包含在上下文窗口中
- url: [官方 API 文档 URL]
  why: [需要使用的具体章节/方法]
  
- file: [path/to/example.py]
  why: [需遵循的模式，需避免的陷阱]
  
- doc: [库文档 URL] 
  section: [关于常见陷阱的具体章节]
  critical: [防止常见错误的关键见解]

- docfile: [PRPs/ai_docs/file.md]
  why: [用户已粘贴到项目中的文档]
```

### 当前代码库树 (在项目根目录运行 `tree`)

```bash

```

### 期望的代码库树 (含需添加的文件及文件职责)

```bash

```

### 代码库已知陷阱与库特性

```python
# CRITICAL: [库名称] 需要 [特定设置]
# 示例: FastAPI 要求端点使用异步函数
# 示例: 该 ORM 不支持超过 1000 条记录的批量插入
# 示例: 我们使用 pydantic v2
```

## 实现蓝图

### 数据模型与结构

创建核心数据模型，确保类型安全和一致性。

```python
示例: 
 - orm 模型
 - pydantic 模型
 - pydantic schemas
 - pydantic 验证器
```

### 需完成的任务列表 (按实现顺序)

```yaml
任务 1:
MODIFY src/existing_module.py:
  - FIND 模式: "class OldImplementation"
  - INJECT 在包含 "def __init__" 的行之后
  - PRESERVE 现有方法签名

CREATE src/new_feature.py:
  - MIRROR 模式来自: src/similar_feature.py
  - MODIFY 类名和核心逻辑
  - KEEP 错误处理模式完全一致

...(...)

任务 N:
...
```

### 每个任务的伪代码 (按需添加)

```python

# 任务 1
# 伪代码含 CRITICAL 细节 (不写完整代码)
async def new_feature(param: str) -> Result:
    # PATTERN: 始终先验证输入 (参见 src/validators.py)
    validated = validate_input(param)  # 抛出 ValidationError
  
    # GOTCHA: 该库需要连接池
    async with get_connection() as conn:  # 参见 src/db/pool.py
        # PATTERN: 使用现有重试装饰器
        @retry(attempts=3, backoff=exponential)
        async def _inner():
            # CRITICAL: API 在 >10 req/sec 时返回 429
            await rate_limiter.acquire()
            return await external_api.call(validated)
    
        result = await _inner()
  
    # PATTERN: 标准化响应格式
    return format_response(result)  # 参见 src/utils/responses.py
```

### 集成点

```yaml
DATABASE:
  - migration: "在 users 表添加 'feature_enabled' 列"
  - index: "CREATE INDEX idx_feature_lookup ON users(feature_id)"
  
CONFIG:
  - add to: config/settings.py
  - pattern: "FEATURE_TIMEOUT = int(os.getenv('FEATURE_TIMEOUT', '30'))"
  
ROUTES:
  - add to: src/api/routes.py  
  - pattern: "router.include_router(feature_router, prefix='/feature')"
```

## 验证循环

### 级别 1: 语法与风格

```bash
# 先运行这些 - 修复所有错误后再继续
ruff check src/new_feature.py --fix  # 自动修复可修复项
mypy src/new_feature.py              # 类型检查

# 期望: 无错误。若有错误，阅读错误并修复。
```

### 级别 2: 单元测试 (每个新功能/文件/函数使用现有测试模式)

```python
# 创建 test_new_feature.py 包含以下测试用例:
def test_happy_path():
    """基础功能正常工作"""
    result = new_feature("valid_input")
    assert result.status == "success"

def test_validation_error():
    """无效输入抛出 ValidationError"""
    with pytest.raises(ValidationError):
        new_feature("")

def test_external_api_timeout():
    """优雅处理超时"""
    with mock.patch('external_api.call', side_effect=TimeoutError):
        result = new_feature("valid")
        assert result.status == "error"
        assert "timeout" in result.message
```

```bash
# 运行并迭代直至通过:
uv run pytest test_new_feature.py -v
# 若失败: 阅读错误，理解根本原因，修复代码，重新运行 (切勿为通过测试而伪造)
```

### 级别 3: 集成测试

```bash
# 启动服务
uv run python -m src.main --dev

# 测试端点
curl -X POST http://localhost:8000/feature \
  -H "Content-Type: application/json" \
  -d '{"param": "test_value"}'

# 期望: {"status": "success", "data": {...}}
# 若出错: 检查 logs/app.log 获取堆栈跟踪
```

## 最终验证清单

- [ ] 所有测试通过: `uv run pytest tests/ -v`
- [ ] 无 linting 错误: `uv run ruff check src/`
- [ ] 无类型错误: `uv run mypy src/`
- [ ] 手动测试成功: [具体 curl/命令]
- [ ] 优雅处理错误情况
- [ ] 日志信息充分但不过度冗余
- [ ] 按需更新文档

---

## 需避免的反模式

- ❌ 当现有模式有效时不要创建新模式
- ❌ 不要因"应该能工作"而跳过验证
- ❌ 不要忽略失败测试 - 修复它们
- ❌ 不要在异步上下文中使用同步函数
- ❌ 不要将应配置的值硬编码
- ❌ 不要捕获所有异常 - 需具体化
