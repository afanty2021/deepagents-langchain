[根目录](../CLAUDE.md) > **examples**

# DeepAgents 示例技能库

示例技能库提供了预构建的技能模板和实际使用案例，展示了如何扩展DeepAgents的功能。

## 模块职责

示例技能库负责：
- **技能模板** - 提供标准化技能开发模板
- **使用示例** - 展示各种实际应用场景
- **最佳实践** - 演示技能开发规范
- **功能演示** - 展示DeepAgents的扩展能力

## 技能结构

### 标准技能格式

每个技能遵循统一的目录结构：

```
skill-name/
├── SKILL.md          # 技能文档（必需）
├── skill_name.py     # 技能实现代码
├── requirements.txt  # 依赖列表（可选）
└── examples/         # 使用示例（可选）
```

### 技能文档规范

**SKILL.md** 包含以下YAML前置元数据：

```yaml
---
name: skill-name
description: 技能功能的简短描述
---
```

文档内容包含：
- **使用场景** - 何时使用此技能
- **使用方法** - 详细的调用说明
- **参数说明** - 输入参数和选项
- **输出格式** - 返回结果格式
- **依赖要求** - 必需的Python包
- **注意事项** - 使用限制和建议

## 可用技能

### 1. arXiv 搜索技能

**路径**: `examples/skills/arxiv-search/`

**功能**: 搜索arXiv预印本库中的学术论文

**适用场景**:
- 查找期刊发表前的预印本论文
- 搜索计算生物学、生物信息学论文
- 获取机器学习在生物领域的应用研究
- 查找最新的研究进展

**使用方法**:
```bash
# 基础搜索
.venv/bin/python [SKILLS_DIR]/arxiv-search/arxiv_search.py "your search query"

# 限制结果数量
python arxiv_search.py "protein folding" --max-papers 5
```

**依赖**: `arxiv` Python包

**特点**:
- 无需API密钥
- 按相关性排序结果
- 支持多领域搜索（计算机科学、生物学、数学等）
- 简洁的文本输出格式

### 2. 网络研究技能

**路径**: `examples/skills/web-research/`

**功能**: 基于Tavily的综合网络搜索和研究

**适用场景**:
- 深度网络调研
- 多源信息整合
- 实时信息获取
- 竞品分析

**使用方法**:
```bash
python web_research.py "研究主题" --max-sources 10
```

**依赖**: `tavily-python` 和Tavily API密钥

### 3. LangGraph 文档查询

**路径**: `examples/skills/langgraph-docs/`

**功能**: 搜索LangGraph官方文档

**适用场景**:
- LangGraph开发指南查询
- API文档搜索
- 最佳实践查找
- 故障排除指南

**使用方法**:
```bash
python langgraph_docs.py "搜索关键词"
```

## 技能开发指南

### 创建新技能

1. **创建技能目录**:
```bash
mkdir examples/skills/my-skill
cd examples/skills/my-skill
```

2. **编写技能文档** (SKILL.md):
```markdown
---
name: my-skill
description: 我的自定义技能
---

# My Skill

这是我的自定义技能描述...

## 使用方法
```bash
python my_skill.py "参数"
```

## 依赖
- package-name
```

3. **实现技能代码** (my_skill.py):
```python
#!/usr/bin/env python3
"""技能实现代码"""

import argparse
import sys

def main():
    parser = argparse.ArgumentParser(description="我的技能")
    parser.add_argument("input", help="输入参数")
    args = parser.parse_args()

    # 技能逻辑实现
    result = process_input(args.input)
    print(result)

if __name__ == "__main__":
    main()
```

4. **测试技能**:
```bash
python my_skill.py "测试参数"
```

### 技能开发最佳实践

1. **错误处理** - 优雅处理缺失依赖和错误输入
2. **参数验证** - 验证用户输入的有效性
3. **输出格式** - 提供清晰、结构化的输出
4. **文档完整** - 详细的使用说明和示例
5. **依赖明确** - 明确列出所有必需依赖

### 技能集成

在DeepAgents中使用技能：

```python
from deepagents import create_deep_agent
from langchain_core.tools import tool

@tool
def use_my_skill(query: str) -> str:
    """使用我的自定义技能"""
    import subprocess
    result = subprocess.run([
        "python", "/path/to/my_skill.py", query
    ], capture_output=True, text=True)
    return result.stdout

agent = create_deep_agent(tools=[use_my_skill])
```

## 技能生态系统

### 社区技能

DeepAgents支持社区贡献的技能：

- **GitHub仓库** - 分享技能到社区仓库
- **技能市场** - 集中的技能发现平台
- **评价系统** - 用户反馈和评分

### 技能管理

CLI工具提供技能管理功能：

```bash
# 列出可用技能
deepagents skills list

# 安装新技能
deepagents skills install skill-name

# 创建新技能
deepagents skills create new-skill

# 更新技能
deepagents skills update skill-name
```

## 扩展示例

### 学术研究场景

```python
# 组合多个技能进行学术研究
from deepagents import create_deep_agent

# 同时使用arXiv搜索和网络研究
agent = create_deep_agent(
    system_prompt="""你是一名学术研究员，擅长跨学科研究。
    使用arXiv查找最新论文，使用网络搜索补充背景信息。""",
    tools=[arxiv_search, web_research]
)

result = agent.invoke({
    "messages": [{
        "role": "user",
        "content": "研究深度学习在蛋白质结构预测中的最新进展"
    }]
})
```

### 技术文档查询

```python
# 专门用于技术文档查询的代理
doc_agent = create_deep_agent(
    system_prompt="你是技术文档专家，专门帮助查询和理解技术文档。",
    tools=[langgraph_docs_search, web_search]
)
```

## 相关文件清单

### 技能实现
- `examples/skills/arxiv-search/arxiv_search.py` - arXiv搜索实现
- `examples/skills/arxiv-search/SKILL.md` - arXiv技能文档
- `examples/skills/web-research/SKILL.md` - 网络研究技能文档
- `examples/skills/langgraph-docs/SKILL.md` - LangGraph文档查询

### 技能资源
- 技能模板文件
- 依赖管理文件
- 使用示例代码

## 变更记录 (Changelog)

### 2025-11-19 - 初始化文档
- 完成示例技能库分析
- 识别3个核心技能：arXiv搜索、网络研究、文档查询
- 建立技能开发标准和最佳实践
- 提供技能集成和管理指南

---

*本文档由自适应架构师自动生成和更新*