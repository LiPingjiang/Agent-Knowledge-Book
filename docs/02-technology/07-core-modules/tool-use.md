---
title: "工具使用：连接 Agent 与外部世界"
description: "系统讲解 Agent 工具使用的设计模式、协议标准、安全机制与工程实践"
tags: ["tool-use", "function-calling", "MCP", "agent-tools", "API-design"]
date: 2024-01-15
author: "Agent Knowledge Book"
---

## 概述

工具使用（Tool Use）是 Agent 从"纯文本生成器"跨越到"行动执行者"的关键桥梁。如果说推理是 Agent 的大脑，那么工具就是 Agent 的双手——它们使 Agent 能够读写文件、查询数据库、调用 API、浏览网页，从而真正地与外部世界产生交互。

工具使用能力的核心挑战不在于"能不能调用工具"，而在于"何时调用、调用哪个、用什么参数、如何处理结果"。本章将系统性地讨论工具使用的设计模式、协议标准和工程实践。

## 工具使用的基本范式

### Function Calling 机制

Function Calling 是当前主流 LLM 提供的原生工具调用能力。其核心流程是：

```mermaid
sequenceDiagram
    participant User as 用户
    participant LLM as LLM
    participant Tool as 工具层
    participant Env as 外部环境
    
    User->>LLM: 用户消息
    LLM->>LLM: 判断是否需要工具
    LLM-->>Tool: 生成工具调用 (name + args)
    Tool->>Env: 执行实际操作
    Env-->>Tool: 返回结果
    Tool-->>LLM: 工具结果注入上下文
    LLM->>User: 基于结果生成回复
```

### OpenAI 风格的 Function Calling

OpenAI 的方案通过 JSON Schema 定义工具接口：

```python
tools = [
    {
        "type": "function",
        "function": {
            "name": "search_database",
            "description": "搜索产品数据库，根据关键词返回匹配的产品列表",
            "parameters": {
                "type": "object",
                "properties": {
                    "query": {
                        "type": "string",
                        "description": "搜索关键词"
                    },
                    "category": {
                        "type": "string",
                        "enum": ["electronics", "clothing", "food"],
                        "description": "产品分类过滤"
                    },
                    "max_results": {
                        "type": "integer",
                        "description": "最大返回数量，默认10"
                    }
                },
                "required": ["query"]
            }
        }
    }
]
```

### Anthropic 风格的 Tool Use

Anthropic 的方案在概念上类似，但强调工具描述的详细程度和使用场景说明，并通过 `tool_use` 和 `tool_result` 消息类型实现多轮工具交互。

## 工具描述设计最佳实践

工具描述（Tool Description）的质量直接影响 LLM 选择和调用工具的准确性。好的工具描述应包含：

**清晰的功能边界**：明确说明工具能做什么和不能做什么。模糊的描述会导致 LLM 在不恰当的时机调用工具。

**参数语义说明**：每个参数都需要清晰的自然语言描述，包括取值范围、默认值和典型用例。

**返回值格式**：说明工具返回数据的结构，帮助 LLM 预期和解析结果。

**使用场景示例**：提供 1-2 个典型调用场景，作为 few-shot 指引。

```python
# 反例：描述过于简略
bad_tool = {
    "name": "query",
    "description": "查询数据",
    "parameters": {"q": {"type": "string"}}
}

# 正例：描述详尽明确
good_tool = {
    "name": "query_order_status",
    "description": (
        "查询用户订单的当前状态。适用于用户询问订单物流、"
        "配送进度、退款状态等场景。不适用于修改订单或取消订单。"
        "返回订单状态枚举值和最后更新时间。"
    ),
    "parameters": {
        "order_id": {
            "type": "string",
            "description": "订单编号，格式为 ORD-XXXXXXXX"
        }
    }
}
```

## MCP：标准化工具接口协议

Model Context Protocol (MCP) 是 Anthropic 在 2024 年提出的开放标准，旨在为 Agent 与工具之间建立统一的通信协议——类似于 USB 对硬件设备所做的事情。

### MCP 的核心设计

MCP 定义了三种核心原语：

- **Tools**：可执行的操作（如搜索、写文件、调用 API）
- **Resources**：可读取的数据源（如文件内容、数据库表）
- **Prompts**：可复用的提示模板

MCP 采用 Client-Server 架构，Agent 作为 Client 连接多个 MCP Server，每个 Server 暴露一组工具和资源。这种设计使得工具的开发和部署可以独立于 Agent 本身。

### MCP 的优势

**标准化**：不同 Agent 框架可以共享同一套工具实现，避免重复开发。

**动态发现**：Agent 可以在运行时发现和加载新工具，无需预先硬编码。

**权限隔离**：每个 MCP Server 运行在独立进程中，天然实现了安全沙箱。

## 工具选择：LLM 如何决定调用哪个工具

当 Agent 配备了几十甚至上百个工具时，准确选择正确的工具成为核心挑战。

### 影响工具选择的因素

- **工具描述质量**：描述越精确，LLM 选择越准确
- **工具数量**：工具过多会导致选择困难（实践中建议单次不超过 20-30 个工具）
- **上下文相关性**：当前对话上下文对工具选择有强引导作用
- **工具间语义距离**：功能相似的工具容易混淆

### 优化策略

**分层工具路由**：先用分类器确定工具类别，再在小范围内精选。

**动态工具加载**：根据当前任务上下文只加载相关工具，减少选择空间。

**工具使用示例**：在 System Prompt 中提供典型场景与工具的对应关系。

## 错误处理

工具调用是 Agent 系统中最容易出错的环节。健壮的错误处理机制至关重要：

```python
class ToolExecutor:
    """带错误处理的工具执行器"""
    
    def execute(self, tool_name: str, args: dict, max_retries: int = 3):
        """执行工具调用，包含重试和降级逻辑"""
        for attempt in range(max_retries):
            try:
                # 参数校验
                validated_args = self.validate_args(tool_name, args)
                
                # 执行工具
                result = self.tools[tool_name].run(validated_args)
                
                # 结果校验
                if self.is_valid_result(result):
                    return ToolResult(success=True, data=result)
                else:
                    raise InvalidResultError(f"工具返回了无效结果: {result}")
                    
            except RateLimitError:
                # 限流：指数退避重试
                wait_time = 2 ** attempt
                time.sleep(wait_time)
                
            except ToolNotFoundError:
                # 工具不存在：尝试降级到替代工具
                fallback = self.find_fallback(tool_name)
                if fallback:
                    return self.execute(fallback, args, max_retries=1)
                return ToolResult(success=False, error="工具不可用且无替代方案")
                
            except ValidationError as e:
                # 参数错误：让 LLM 修正参数
                return ToolResult(
                    success=False, 
                    error=f"参数校验失败: {e}",
                    suggestion="请检查参数格式并重试"
                )
        
        return ToolResult(success=False, error="重试次数耗尽")
```

## 工具组合与链式调用

复杂任务往往需要多个工具协作完成。工具组合（Tool Composition）有两种主要模式：

**顺序链式**：工具 A 的输出作为工具 B 的输入。例如：搜索文档 -> 提取关键信息 -> 写入报告。

**并行扇出**：同时调用多个工具，汇聚结果。例如：同时查询天气、航班和酒店信息。

**条件分支**：根据某个工具的结果决定后续调用路径。

LLM 天然具备编排这些模式的能力——它可以根据中间结果动态决定下一步调用什么工具。这也是 Agent 相比传统工作流引擎的核心优势。

## 安全机制

工具使用引入了显著的安全风险——一个被恶意 Prompt 注入的 Agent 可能通过工具执行危险操作。

### 沙箱隔离

工具执行应在受限环境中运行，限制文件系统访问范围、网络访问白名单和系统调用权限。

### 权限模型

为每个工具定义明确的权限级别：只读操作（低风险）、写入操作（中风险）和不可逆操作如删除、发送（高风险，需人工确认）。

### 速率限制

对工具调用频率设置上限，防止 Agent 进入无限循环或被恶意利用进行资源耗尽攻击。

### 审计日志

记录所有工具调用的完整信息（调用者、参数、结果、时间戳），便于事后追溯和异常检测。

## 工具注册与动态分发

```python
class ToolRegistry:
    """工具注册表：管理工具的注册、发现和分发"""
    
    def __init__(self):
        self._tools = {}
        self._categories = {}
    
    def register(self, tool, category: str = "general"):
        """注册工具"""
        self._tools[tool.name] = tool
        self._categories.setdefault(category, []).append(tool.name)
    
    def get_tools_for_context(self, context: str, max_tools: int = 15):
        """根据上下文动态选择相关工具"""
        # 计算每个工具与当前上下文的相关性
        scored_tools = []
        for name, tool in self._tools.items():
            relevance = compute_relevance(tool.description, context)
            scored_tools.append((name, relevance))
        
        # 返回最相关的工具
        scored_tools.sort(key=lambda x: x[1], reverse=True)
        selected = [self._tools[name] for name, _ in scored_tools[:max_tools]]
        return selected
    
    def to_schema(self, tools=None):
        """将工具列表转为 LLM 可理解的 Schema 格式"""
        tools = tools or self._tools.values()
        return [tool.to_function_schema() for tool in tools]
```

## 从工具到技能：能力的高层封装

到目前为止，本章讨论的都是**原子层面**的工具使用——单次函数调用、单个 API 交互。但实际的 Agent 任务往往需要"一组工具 + 领域知识 + 执行策略"的协同。这催生了一个更高层的抽象概念——在不同平台上被称为 Skill、Plugin、Toolkit、Action 或 Extension。

三层递进关系如下：

| 层级 | 封装内容 | 典型粒度 | 例子 |
|------|---------|---------|------|
| **Tool** | 单个可调用函数 | 一次 API 调用 | `search_web(query)` |
| **Toolkit** | 同一领域的工具集合 + 共享配置 | 一个服务的多个端点 | Gmail Toolkit（发送、搜索、读取） |
| **Skill** | 工具 + 知识 + 指令 + 触发条件 | 完成一类任务的完整能力 | "代码评审 Skill"（含评审标准、Git API 工具、评审流程指令） |

一个完整的 Skill 级能力封装通常包含三个组成部分：**指令层**（告诉 Agent 何时使用、按什么步骤执行）、**工具层**（提供实际执行能力，通常通过 MCP 或 Function Calling 接入）、**知识层**（提供领域判断所需的上下文信息）。缺少任何一层都会导致能力不完整。

当前行业尚未形成统一的 Skill 标准——OpenAI 用 GPTs+Actions、Microsoft 用 Copilot Agents、LangChain 用 Toolkit、CatPaw 用 Skill。但各平台的设计正在向"指令 + 工具 + 知识"三层模型收敛，MCP 正在成为工具层的通用连接标准。

关于各平台能力封装机制的详细对比分析，参见 [从工具到技能：Agent 能力扩展生态](./skill-ecosystem.md)。

## 学术参考

Toolformer [Schick et al., 2023] 是最早展示 LLM 可以自主学会何时及如何使用工具的研究，通过自监督方式让模型在生成过程中插入 API 调用。ToolLLM [Qin et al., 2023] 进一步扩展到大规模工具集（16000+ API），并提出了深度优先搜索的工具选择策略。这些研究奠定了当前 Agent 工具使用的理论基础。

## 本章小结

工具使用能力使 Agent 从"知道"跃迁到"能做"。Function Calling 提供了底层机制，MCP 正在建立行业标准，而良好的工具描述设计、错误处理和安全机制则是生产级 Agent 的必备要素。更高层的 Skill/Plugin 封装正在让工具生态从"原子操作的集合"演化为"可复用、可分发的完整能力包"——Agent 不再只是调用工具，而是加载整套包含知识、策略和工具组合的技能。

## 延伸阅读

- [Schick et al., 2023] "Toolformer: Language Models Can Teach Themselves to Use Tools"
- [Qin et al., 2023] "ToolLLM: Facilitating Large Language Models to Master 16000+ Real-world APIs"
- [Anthropic, 2024] Model Context Protocol Specification: https://modelcontextprotocol.io
- [Patil et al., 2023] "Gorilla: Large Language Model Connected with Massive APIs"
- OpenAI Function Calling Guide: https://platform.openai.com/docs/guides/function-calling
