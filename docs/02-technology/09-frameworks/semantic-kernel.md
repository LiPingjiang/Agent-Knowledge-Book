---
title: "Semantic Kernel：微软的企业级方案"
description: "解析微软 Semantic Kernel 的企业级 AI 编排能力及其在 Azure 生态中的定位"
tags: ["semantic-kernel", "microsoft", "enterprise", "azure", "framework"]
date: 2025-06-01
author: "Agent Knowledge Book"
---

# Semantic Kernel：微软的企业级方案

Semantic Kernel（SK）是微软于 2023 年 3 月发布的 AI 编排框架，定位为"企业级 AI 应用的操作系统层"。与 LangChain 等面向初创公司和独立开发者的框架不同，SK 从设计之初就聚焦企业客户的核心诉求：安全合规、可观测性、与现有企业系统的无缝集成。它是微软 Copilot 产品线（Microsoft 365 Copilot、GitHub Copilot 等）背后的核心编排引擎。

## 定位与哲学

Semantic Kernel 的设计哲学可以用一句话概括："AI 是企业软件的一个能力层，而非独立的系统"。这意味着 SK 不试图重新定义开发者的工作方式，而是让 AI 能力自然地融入现有的企业应用架构。

这种定位带来的直接后果是：SK 支持 C#、Python、Java 三种主要企业语言（C# 最为完善）；它深度集成 Azure OpenAI、Azure AI Search、Microsoft Graph 等微软生态组件；它的 API 设计遵循企业软件的最佳实践——依赖注入、接口抽象、配置分离。

## 核心概念

### Kernel（内核）

Kernel 是 SK 的中央协调器，负责管理 AI 服务、插件和执行管道。所有操作都通过 Kernel 进行：

```python
from semantic_kernel import Kernel
from semantic_kernel.connectors.ai.open_ai import AzureChatCompletion

# 创建内核
kernel = Kernel()

# 注册 AI 服务
kernel.add_service(AzureChatCompletion(
    deployment_name="gpt-4o",
    endpoint="https://your-resource.openai.azure.com/",
    api_key="your-key"
))
```

### Plugins（插件）

Plugin 是 SK 的功能扩展单元，将相关功能组织在一起。插件中的函数分为两类：Semantic Function（用自然语言提示定义的 AI 函数）和 Native Function（用代码实现的普通函数）。

```python
from semantic_kernel.functions import kernel_function

class TimePlugin:
    """时间相关的功能插件"""
    
    @kernel_function(description="获取当前时间")
    def get_current_time(self) -> str:
        from datetime import datetime
        return datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    
    @kernel_function(description="计算两个日期之间的天数差")
    def days_between(self, start_date: str, end_date: str) -> str:
        from datetime import datetime
        start = datetime.strptime(start_date, "%Y-%m-%d")
        end = datetime.strptime(end_date, "%Y-%m-%d")
        return str((end - start).days)

# 注册插件
kernel.add_plugin(TimePlugin(), plugin_name="time")
```

### Planner（规划器）

Planner 是 SK 的自动编排组件，能够根据用户意图自动组合多个插件函数来完成目标。它本质上是用 LLM 进行任务规划：

```python
from semantic_kernel.planners import FunctionCallingStepwisePlanner

planner = FunctionCallingStepwisePlanner(
    service_id="gpt-4o",
    max_iterations=10
)

# 自动规划和执行
result = await planner.invoke(
    kernel,
    "帮我计算从今天到 2025 年底还有多少天，并生成一份年度总结提纲"
)
```

### Memory（记忆）

SK 的 Memory 组件提供语义记忆能力，基于向量搜索实现知识的存储和检索：

```python
from semantic_kernel.memory import SemanticTextMemory
from semantic_kernel.connectors.memory.azure_ai_search import AzureAISearchMemoryStore

# 使用 Azure AI Search 作为记忆存储
memory_store = AzureAISearchMemoryStore(
    search_endpoint="https://your-search.search.windows.net",
    admin_key="your-key"
)

memory = SemanticTextMemory(storage=memory_store, embeddings_generator=embedding_service)

# 存储知识
await memory.save_information("company_docs", id="doc1", 
    text="公司的年度营收目标是 10 亿美元")

# 检索相关知识
results = await memory.search("company_docs", "今年的营收目标是多少")
```

## Process Framework：工作流编排

2024 年，SK 推出了 Process Framework，提供了类似 LangGraph 的工作流编排能力，但更符合企业级应用的设计模式：

```python
from semantic_kernel.processes import ProcessBuilder, ProcessStepBuilder

# 定义流程步骤
class GatherInfoStep:
    @kernel_function
    async def gather(self, context: str) -> str:
        # 收集信息
        return "收集到的信息..."

class AnalyzeStep:
    @kernel_function
    async def analyze(self, info: str) -> str:
        # 分析信息
        return "分析结论..."

class ReportStep:
    @kernel_function
    async def generate_report(self, analysis: str) -> str:
        # 生成报告
        return "最终报告..."

# 构建流程
process = ProcessBuilder(name="调研流程")
gather = process.add_step(GatherInfoStep)
analyze = process.add_step(AnalyzeStep)
report = process.add_step(ReportStep)

# 定义边
process.on_input_event().send_event_to(gather)
gather.on_function_result().send_event_to(analyze)
analyze.on_function_result().send_event_to(report)
```

## 企业级特性

### 安全与合规

SK 内置了多层安全机制：内容过滤（通过 Azure AI Content Safety）、数据隔离（每个用户的上下文独立）、审计日志（所有 AI 调用可追溯）、token 限制和成本控制。

### 可观测性

与 Azure Monitor 和 OpenTelemetry 深度集成，提供 AI 调用的完整遥测数据：延迟、token 消耗、错误率、模型性能指标。

### 多模型支持

虽然与 Azure OpenAI 集成最深，但 SK 通过 connector 模式支持多种 AI 服务：OpenAI、Anthropic、Google AI、Hugging Face 等。

## 与 LangChain 的定位差异

| 维度 | Semantic Kernel | LangChain |
|------|-----------------|-----------|
| 主要语言 | C# (最完善) | Python |
| 目标用户 | 企业开发者 | 初创/独立开发者 |
| 设计风格 | 企业架构模式 | 快速原型 |
| 生态集成 | Azure/Microsoft | 广泛但浅 |
| 稳定性 | 高（企业级承诺） | 中（迭代快） |
| 灵活性 | 中 | 高 |
| 学习曲线 | 中（需熟悉企业模式） | 中（抽象多） |

## 适用场景

Semantic Kernel 最适合以下场景：团队使用 C# 或 .NET 技术栈；已深度使用 Azure 生态；需要企业级安全合规保障；项目需要长期维护且追求 API 稳定性；正在为 Microsoft 365 等平台开发 AI 扩展。

如果团队是 Python 优先、追求快速迭代、或者不在 Azure 生态内，LangChain/LangGraph 或 OpenAI SDK 通常是更合适的选择。

## 本章小结

Semantic Kernel 代表了企业级 AI 框架应有的品质——稳健、安全、可观测、与成熟生态深度集成。它不是最炫酷或最灵活的框架，但对于需要将 AI 能力融入企业级应用的团队而言，SK 提供了最可靠的路径。随着企业 AI 应用从 PoC 走向生产，SK 的价值将越发凸显。

## 延伸阅读

- [Semantic Kernel 官方文档](https://learn.microsoft.com/semantic-kernel/)
- [Semantic Kernel GitHub](https://github.com/microsoft/semantic-kernel)
- [SK Python 快速入门](https://learn.microsoft.com/semantic-kernel/get-started/quick-start-guide)
- [Microsoft Copilot 架构](https://learn.microsoft.com/microsoft-365-copilot/extensibility/)
- [企业级 Agent 部署](./comparison-matrix.md) — 框架对比矩阵
