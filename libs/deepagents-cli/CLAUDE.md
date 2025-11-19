[根目录](../../CLAUDE.md) > [libs](../) > **deepagents-cli**

# DeepAgents CLI 工具

DeepAgents CLI 是框架的命令行界面，提供交互式Agent体验、命令管理和多种集成功能。

## 模块职责

CLI工具负责：
- **交互式界面** - 提供用户友好的命令行交互体验
- **命令管理** - 处理各种CLI命令和参数解析
- **代理管理** - 支持多代理创建、配置和状态管理
- **集成支持** - 集成外部工具和沙箱环境
- **技能系统** - 管理和执行可扩展的技能插件

## 入口与启动

### 主要入口点

```python
# deepagents_cli/__main__.py - Python模块执行入口
from deepagents_cli.main import cli_main

if __name__ == "__main__":
    cli_main()
```

```python
# pyproject.toml - 可执行脚本配置
[project.scripts]
deepagents = "deepagents_cli:cli_main"
deepagents-cli = "deepagents_cli:cli_main"
```

### 启动流程

**文件**: `deepagents_cli/main.py`

CLI启动包含以下步骤：

1. **依赖检查** - 验证必需的CLI依赖包
2. **参数解析** - 处理命令行参数和子命令
3. **配置加载** - 读取设置和环境配置
4. **沙箱创建** - 初始化执行环境（如需要）
5. **Agent创建** - 使用配置创建DeepAgent
6. **交互循环** - 启动用户输入处理循环

## 对外接口

### 命令行接口

CLI支持以下主要命令：

```bash
# 交互式模式（默认）
deepagents

# 代理管理
deepagents list                    # 列出所有可用代理
deepagents reset --agent <name>    # 重置指定代理状态

# 技能管理
deepagents skills list             # 列出可用技能
deepagents skills install <name>   # 安装技能
deepagents skills create <name>    # 创建新技能

# 帮助信息
deepagents help                    # 显示帮助
```

### 编程接口

```python
# deepagents_cli/agent.py - 代理管理
def create_agent_with_config(agent_name: str, **kwargs)
def list_agents() -> list[str]
def reset_agent(agent_name: str)

# deepagents_cli/commands.py - 命令处理
def handle_command(session_state: SessionState, command: str)
def execute_bash_command(command: str) -> str

# deepagents_cli/execution.py - 任务执行
async def execute_task(agent, user_input: str)
```

## 关键依赖与配置

### 主要依赖

```toml
# pyproject.toml
dependencies = [
  "deepagents==0.2.7",                    # 核心库
  "requests",                             # HTTP请求
  "rich>=13.0.0",                         # 终端美化
  "prompt-toolkit>=3.0.52",               # 交互式输入
  "langchain-openai>=0.1.0",              # OpenAI模型
  "tavily-python",                        # 网络搜索
  "python-dotenv",                        # 环境变量
  "daytona>=0.113.0",                    # Daytona沙箱
  "modal>=0.65.0",                        # Modal云平台
  "markdownify>=0.13.0",                 # Markdown处理
  "langchain>=1.0.7",                     # LangChain
]
```

### 开发依赖

```toml
test = [
    "pytest>=8.3.4",
    "pytest-asyncio>=0.25.3",
    "pytest-cov>=6.0.0",
    "pytest-mock>=3.14.0",
    "pytest-socket>=0.7.0",
    "pytest-timeout>=2.3.1",
    "responses>=0.25.0",
    "ruff>=0.9.7",
]
```

### 配置系统

**文件**: `deepagents_cli/config.py`

配置管理包含：
- **模型配置** - 支持多种LLM提供商
- **沙箱配置** - Daytona、Modal等环境
- **UI配置** - 颜色主题、提示符等
- **会话状态** - 代理状态和记忆管理

## 数据模型

### 会话状态

```python
class SessionState:
    """CLI会话状态管理"""
    agent_name: str                       # 当前代理名称
    agent: CompiledStateGraph             # Agent实例
    memory: dict                          # 会话记忆
    settings: dict                        # 配置设置
    token_tracker: TokenTracker           # Token使用跟踪
```

### 技能配置

```python
# examples/skills/*/SKILL.md 技能描述格式
---
name: skill-name
description: 技能功能描述
---

# 技能文档
包含使用方法、参数说明、依赖等信息
```

## 架构组件

### 核心模块结构

```
deepagents_cli/
├── __init__.py          # 包初始化
├── __main__.py          # Python模块执行入口
├── main.py              # 主入口和CLI循环
├── agent.py             # 代理创建和管理
├── commands.py          # 命令处理逻辑
├── config.py            # 配置管理
├── execution.py         # 任务执行
├── input.py             # 用户输入处理
├── ui.py                # 用户界面组件
├── tools.py             # 内置工具（网络搜索等）
├── project_utils.py     # 项目工具函数
├── token_utils.py       # Token处理工具
├── agent_memory.py      # 代理记忆管理
├── integrations/        # 外部集成
│   ├── daytona.py      # Daytona沙箱
│   ├── modal.py        # Modal云平台
│   ├── runloop.py      # Runloop沙箱
│   └── sandbox_factory.py # 沙箱工厂
└── skills/              # 技能系统
    ├── __init__.py
    ├── commands.py     # 技能命令处理
    ├── load.py         # 技能加载器
    └── middleware.py   # 技能中间件
```

### 沙箱集成

CLI支持多种执行环境：

| 沙箱类型 | 用途 | 特点 |
|----------|------|------|
| **Daytona** | 开发环境 | 完整的IDE体验 |
| **Modal** | 云函数 | 无服务器执行 |
| **Runloop** | 隔离环境 | 安全的代码执行 |

### 技能系统

技能是可扩展的功能插件：

- **技能结构** - 标准化的技能目录和文档
- **技能加载** - 动态加载和管理技能
- **技能执行** - 安全的技能执行环境
- **技能仓库** - 社区技能分享

## 测试与质量

### 测试结构

```
tests/
├── unit_tests/                    # 单元测试
│   ├── test_agent.py             # 代理管理测试
│   ├── test_config.py            # 配置系统测试
│   ├── test_file_ops.py          # 文件操作测试
│   ├── test_imports.py           # 导入测试
│   ├── tools/
│   │   └── test_fetch_url.py     # 工具测试
│   └── skills/
│       └── test_commands.py      # 技能命令测试
├── integration_tests/            # 集成测试
│   ├── test_sandbox_factory.py   # 沙箱工厂测试
│   └── benchmarks/
│       └── test_simple_tasks.py  # 基准测试
└── test_project_memory.py        # 项目记忆测试
```

### 质量配置

```toml
[tool.ruff]
line-length = 100  # CLI使用较短行长度
select = ["ALL"]
ignore = ["COM812", "ISC001", "PERF203", "SLF001"]

[tool.pytest.ini_options]
timeout = 10  # 默认测试超时时间
```

## 常见问题 (FAQ)

### Q: 如何创建自定义代理？
A: 使用CLI的交互式代理创建功能：
```bash
deepagents
# 在交互模式中使用 /agent create 命令
```

### Q: 如何添加新技能？
A: 创建技能目录并添加SKILL.md文档：
```bash
mkdir ~/.deepagents/skills/my-skill
echo "技能文档内容" > ~/.deepagents/skills/my-skill/SKILL.md
echo "技能实现代码" > ~/.deepagents/skills/my-skill/my_skill.py
```

### Q: 如何配置沙箱环境？
A: 设置相应的环境变量：
```bash
# Daytona
export DAYTONA_API_KEY="your-key"

# Modal
export MODAL_TOKEN="your-token"

# 然后在CLI中选择对应沙箱
deepagents --sandbox daytona
```

### Q: 如何管理多个代理？
A: 使用代理管理命令：
```bash
# 列出代理
deepagents list

# 切换代理
deepagents --agent agent-name

# 重置代理状态
deepagents reset --agent agent-name
```

## 相关文件清单

### 核心文件
- `deepagents_cli/main.py` - 主入口和CLI循环（200+行）
- `deepagents_cli/agent.py` - 代理创建和管理
- `deepagents_cli/commands.py` - 命令处理逻辑
- `deepagents_cli/config.py` - 配置管理系统

### 集成模块
- `deepagents_cli/integrations/` - 外部沙箱集成
- `deepagents_cli/skills/` - 技能系统实现
- `deepagents_cli/tools.py` - 内置工具集合

### 测试文件
- `tests/unit_tests/` - 单元测试（8个文件）
- `tests/integration_tests/` - 集成测试（2个文件）
- `tests/test_project_memory.py` - 项目记忆测试

### 配置文件
- `pyproject.toml` - 项目配置和依赖
- `uv.lock` - 锁定依赖版本
- `Makefile` - 构建和开发命令

## 变更记录 (Changelog)

### 2025-11-19 - 初始化文档
- 完成CLI工具架构分析
- 识别4个主要功能模块和3种沙箱集成
- 扫描11个测试文件，覆盖命令处理和配置管理
- 记录技能系统和代理管理功能

---

*本文档由自适应架构师自动生成和更新*