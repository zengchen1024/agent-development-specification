# 编码规范 · Python 专项

> 本文件是 `coding/base.md` 的 Python 语言扩展，仅记录 Python 特有约定。

---

## 1. 代码风格

### 1.1 格式化与 Lint

- 使用 `ruff` 作为 formatter 和 linter（统一替代 black + flake8 + isort）
- 所有代码必须通过 `ruff format` 和 `ruff check`（CI 强制）
- 行长度上限：

> [团队填写] 例如：120 字符

### 1.2 命名约定

| 类型 | 约定 | 示例 |
|------|------|------|
| 变量 / 函数 | snake_case | `user_name`, `get_user()` |
| 类 | PascalCase | `UserService` |
| 常量 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| 私有 | 单下划线前缀 | `_internal_helper()` |
| 模块 | snake_case | `user_service.py` |

- 避免使用双下划线（dunder）前缀，除非明确实现 Python 魔法方法

---

## 2. 类型系统

### 2.1 类型注解

- 所有公开函数、方法的参数和返回值必须有类型注解
- 使用 `from __future__ import annotations` 启用延迟求值（Python 3.10+ 项目可不加）
- 使用 `mypy` 或 `pyright` 进行静态类型检查（CI 强制）

```python
# 推荐
def get_user(user_id: int) -> User | None:
    ...

# 不推荐（无类型注解）
def get_user(user_id):
    ...
```

### 2.2 类型工具

- 使用 `dataclass` 或 `pydantic.BaseModel` 定义数据结构
- 优先使用 `pydantic` 处理外部输入验证（HTTP 请求体、配置文件）
- 避免使用裸 `dict` 传递结构化数据

---

## 3. 错误处理

### 3.1 异常使用原则

- 只捕获预期的异常类型，禁止裸 `except:` 或 `except Exception:`（除最顶层 handler）
- 自定义异常继承自业务基类，而非直接继承 `Exception`

```python
class AppError(Exception):
    """业务异常基类"""
    pass

class UserNotFoundError(AppError):
    def __init__(self, user_id: int):
        self.user_id = user_id
        super().__init__(f"user {user_id} not found")
```

### 3.2 上下文管理器

- 资源（文件、数据库连接、HTTP Session）必须使用 `with` 语句管理
- 自定义资源类实现 `__enter__` / `__exit__`（或使用 `contextlib.contextmanager`）

---

## 4. 异步编程

### 4.1 async/await 使用原则

- 同一代码库中不混用同步和异步（选定一种范式后统一）
- 异步函数命名不加 `async_` 前缀（函数签名已有 `async` 关键字）
- 禁止在异步函数中调用同步阻塞 IO（使用 `asyncio.to_thread()` 包装）

```python
# 推荐：在异步上下文中运行同步阻塞函数
result = await asyncio.to_thread(blocking_io_function, arg)
```

### 4.2 并发控制

- 使用 `asyncio.gather()` 并发执行多个协程
- 使用 `asyncio.Semaphore` 控制并发数量
- 避免无限制地创建 Task（可能导致内存和连接耗尽）

---

## 5. 项目结构

> [团队填写] 定义标准目录结构，例如：

```
service/
├── src/
│   └── <package>/
│       ├── __init__.py
│       ├── domain/         # 实体和领域逻辑
│       ├── service/        # 业务逻辑
│       ├── repository/     # 数据访问
│       ├── api/            # HTTP/gRPC 路由和 handler
│       └── config.py       # 配置定义
├── tests/
│   ├── unit/
│   └── integration/
├── migrations/
├── pyproject.toml          # 项目配置、依赖、工具配置统一入口
└── Dockerfile
```

---

## 6. 依赖管理

- 使用 `uv` 管理依赖和虚拟环境（统一替代 pip + venv + poetry）
- 依赖声明在 `pyproject.toml`；`uv.lock` 提交到仓库
- 区分运行时依赖和开发依赖：

```toml
[project]
dependencies = ["fastapi", "pydantic", "sqlalchemy"]

[project.optional-dependencies]
dev = ["pytest", "ruff", "mypy"]
```

> [团队填写] Python 版本要求：
> ```
> requires-python = ">=3.11"
> ```

---

## 7. 测试

- 使用 `pytest` 作为测试框架
- 测试文件命名：`test_<module>.py`，测试函数命名：`test_<scenario>()`
- 使用 `pytest-asyncio` 测试异步代码
- 使用 `unittest.mock` 或 `pytest-mock` 进行 mock
- 使用 `pytest-cov` 生成覆盖率报告

> 详细测试策略参见 `docs/standards/testing.md`
