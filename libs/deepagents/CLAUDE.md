[根目录](../../CLAUDE.md) > [libs](../) > **deepagents**

# DeepAgents 核心库

DeepAgents 核心库是整个框架的基础，提供Agent创建、中间件系统和后端抽象等核心功能。

## 模块职责

核心库负责：
- **Agent创建** - 提供便捷的Agent创建接口和默认配置
- **中间件系统** - 实现可插拔的功能扩展机制
- **后端抽象** - 定义文件存储和执行环境的标准接口
- **工具集成** - 内置文件操作、任务管理等核心工具

## 入口与启动

### 主要入口点

```python
# deepagents/__init__.py - 主要导出
from deepagents.graph import create_deep_agent
from deepagents.middleware.filesystem import FilesystemMiddleware
from deepagents.middleware.subagents import CompiledSubAgent, SubAgent, SubAgentMiddleware

__all__ = [
    "CompiledSubAgent",
    "FilesystemMiddleware",
    "SubAgent",
    "SubAgentMiddleware",
    "create_deep_agent"
]
```

### Agent创建接口

**文件**: `deepagents/graph.py`

`create_deep_agent()` 函数是创建Agent的主要接口：

```python
def create_deep_agent(
    model: str | BaseChatModel | None = None,
    tools: Sequence[BaseTool | Callable | dict[str, Any]] | None = None,
    *,
    system_prompt: str | None = None,
    middleware: Sequence[AgentMiddleware] = (),
    subagents: list[SubAgent | CompiledSubAgent] | None = None,
    backend: BackendProtocol | BackendFactory | None = None,
    interrupt_on: dict[str, bool | InterruptOnConfig] | None = None,
    # ... 其他参数
) -> CompiledStateGraph
```

**默认配置**：
- 模型：Claude Sonnet 4 (claude-sonnet-4-5-20250929)
- 中间件：TODO列表、文件系统、子代理、摘要化等
- 工具：文件操作、任务管理、子代理委派

## 对外接口

### 核心API

| 接口 | 位置 | 功能 |
|------|------|------|
| `create_deep_agent()` | `graph.py` | 创建配置好的Agent |
| `FilesystemMiddleware` | `middleware/filesystem.py` | 文件系统工具中间件 |
| `SubAgentMiddleware` | `middleware/subagents.py` | 子代理管理中间件 |

### 内置工具

所有DeepAgent默认包含以下工具：

| 工具名 | 描述 | 提供者 |
|--------|------|--------|
| `write_todos` | 创建和管理结构化任务列表 | TodoListMiddleware |
| `read_todos` | 读取当前TODO列表状态 | TodoListMiddleware |
| `ls` | 列出目录文件 | FilesystemMiddleware |
| `read_file` | 读取文件内容（支持分页） | FilesystemMiddleware |
| `write_file` | 创建或覆盖文件 | FilesystemMiddleware |
| `edit_file` | 字符串替换编辑文件 | FilesystemMiddleware |
| `glob` | 模式匹配查找文件 | FilesystemMiddleware |
| `grep` | 文件内容搜索 | FilesystemMiddleware |
| `execute`* | 沙箱环境执行命令 | FilesystemMiddleware |
| `task` | 委派任务给子代理 | SubAgentMiddleware |

*execute工具仅在backend实现SandboxBackendProtocol时可用

## 关键依赖与配置

### 主要依赖

```toml
# pyproject.toml
dependencies = [
    "langchain-anthropic>=1.0.0,<2.0.0",
    "langchain>=1.0.2,<2.0.0",
    "langchain-core>=1.0.0,<2.0.0",
    "wcmatch",
    "daytona>=0.113.0",  # 沙箱环境
    "runloop-api-client>=0.66.1",  # 沙箱环境
    "tavily>=1.1.0",  # 网络搜索
]
```

### 开发依赖

```toml
[dependency-groups]
test = [
  "pytest",
  "pytest-cov",
  "pytest-xdist",
  "ruff>=0.12.2,<0.13.0",
  "mypy>=1.18.1,<1.19.0",
]

dev = [
  "langchain-openai",  # OpenAI模型支持
  "twine",            # 包发布
  "build",            # 构建工具
]
```

## 数据模型

### 后端协议

**文件**: `deepagents/backends/protocol.py`

定义了统一的文件操作接口：

```python
@runtime_checkable
class BackendProtocol(Protocol):
    """可插拔存储后端协议"""

    def ls_info(self, path: str) -> list["FileInfo"]
    def read(self, file_path: str, offset: int = 0, limit: int = 2000) -> str
    def grep_raw(self, pattern: str, path: str | None = None, glob: str | None = None) -> list["GrepMatch"] | str
    def glob_info(self, pattern: str, path: str = "/") -> list["FileInfo"]
    def write(self, file_path: str, content: str) -> WriteResult
    def edit(self, file_path: str, old_string: str, new_string: str, replace_all: bool = False) -> EditResult

@runtime_checkable
class SandboxBackendProtocol(BackendProtocol, Protocol):
    """支持命令执行的沙箱后端"""

    def execute(self, command: str) -> ExecuteResponse
    @property
    def id(self) -> str
```

### 数据结构

- **FileInfo** - 文件元信息（路径、大小、修改时间等）
- **GrepMatch** - 搜索匹配结果
- **WriteResult/EditResult** - 写/编辑操作结果
- **ExecuteResponse** - 命令执行结果

### 子代理配置

```python
class SubAgent(TypedDict):
    name: str                              # 代理名称
    description: str                       # 功能描述
    system_prompt: str                     # 系统提示
    tools: Sequence[BaseTool | Callable]   # 工具集
    model: NotRequired[str | BaseChatModel] # 模型（可选）
    middleware: NotRequired[list[AgentMiddleware]] # 中间件（可选）
    interrupt_on: NotRequired[dict]        # 中断配置（可选）
```

## 架构组件

### 中间件系统

**默认中间件栈**：
1. **TodoListMiddleware** - 任务规划和进度跟踪
2. **FilesystemMiddleware** - 文件操作和上下文卸载
3. **SubAgentMiddleware** - 子代理委派和管理
4. **SummarizationMiddleware** - 自动摘要（170k tokens触发）
5. **AnthropicPromptCachingMiddleware** - 提示缓存（成本优化）
6. **PatchToolCallsMiddleware** - 修复中断后的工具调用
7. **HumanInTheLoopMiddleware** - 人工干预（需配置）

### 后端实现

| 后端类型 | 用途 | 特点 |
|----------|------|------|
| **StateBackend** | 默认 | 临时存储在Agent状态中 |
| **FilesystemBackend** | 磁盘访问 | 真实文件系统操作 |
| **StoreBackend** | 持久化 | 使用LangGraph Store |
| **CompositeBackend** | 混合路由 | 不同路径使用不同后端 |

## 测试与质量

### 测试结构

```
tests/
├── unit_tests/           # 单元测试
│   ├── backends/        # 后端系统测试
│   └── test_middleware.py # 中间件测试
├── integration_tests/    # 集成测试
│   ├── test_deepagents.py    # 完整工作流
│   ├── test_filesystem_middleware.py
│   ├── test_subagent_middleware.py
│   └── test_hitl.py      # 人工干预测试
└── utils.py             # 测试工具
```

### 质量工具配置

```toml
[tool.ruff]
line-length = 150
select = ["ALL"]
ignore = ["COM812", "ISC001", "PERF203", "SLF001"]

[tool.mypy]
strict = true
ignore_missing_imports = true
enable_error_code = ["deprecated"]
disallow_any_generics = false
warn_return_any = false
```

## 常见问题 (FAQ)

### Q: 如何自定义Agent行为？
A: 通过中间件系统添加自定义功能：
```python
from langchain.agents.middleware import AgentMiddleware

class CustomMiddleware(AgentMiddleware):
    def on_model_start(self, request: ModelRequest) -> ModelRequest:
        # 自定义处理逻辑
        return request

agent = create_deep_agent(middleware=[CustomMiddleware()])
```

### Q: 如何实现持久化存储？
A: 使用CompositeBackend配置持久化路径：
```python
from deepagents.backends import CompositeBackend, StateBackend, StoreBackend

agent = create_deep_agent(
    backend=CompositeBackend(
        default=StateBackend(),
        routes={"/memories/": StoreBackend(store=store)}
    )
)
```

### Q: 如何添加自定义工具？
A: 在创建Agent时传入tools参数：
```python
from langchain_core.tools import tool

@tool
def custom_tool(input: str) -> str:
    """自定义工具描述"""
    return f"处理: {input}"

agent = create_deep_agent(tools=[custom_tool])
```

## 相关文件清单

### 核心文件
- `deepagents/__init__.py` - 主要导出接口
- `deepagents/graph.py` - Agent创建和配置
- `deepagents/backends/` - 后端系统实现
- `deepagents/middleware/` - 中间件实现

### 配置文件
- `pyproject.toml` - 项目配置和依赖
- `Makefile` - 构建和开发命令
- `README.md` - 项目说明文档

### 测试文件
- `tests/unit_tests/` - 单元测试（17个文件）
- `tests/integration_tests/` - 集成测试（4个文件）
- `tests/utils.py` - 测试工具函数

## 变更记录 (Changelog)

### 2025-11-19 - 初始化文档
- 完成核心库架构分析
- 详细记录中间件系统和后端协议
- 识别7个后端实现和4个中间件组件
- 扫描21个测试文件，覆盖核心功能

---

*本文档由自适应架构师自动生成和更新*