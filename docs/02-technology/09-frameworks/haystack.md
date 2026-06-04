---
title: "Haystack：面向搜索与 RAG 的框架"
description: "解析 deepset 的 Haystack 框架如何从搜索引擎演进为支持 Agent 能力的 RAG 平台"
tags: ["haystack", "rag", "search", "pipeline", "deepset"]
date: 2025-06-01
author: "Agent Knowledge Book"
---

# Haystack：面向搜索与 RAG 的框架

Haystack 由德国公司 deepset 开发，最早是一个面向企业搜索的 NLP 框架。随着 LLM 和 RAG（Retrieval-Augmented Generation，检索增强生成）技术的崛起，Haystack 在 2023-2024 年进行了重大重构（从 1.x 到 2.x），转型为一个以 pipeline 为核心抽象的 RAG 和 Agent 框架。它不是"Agent 优先"的框架，而是"搜索/RAG 优先，兼具 Agent 能力"的独特定位。

## 演进历程

Haystack 的发展轨迹在 Agent 框架生态中独树一帜——它不是乘着 Agent 热潮诞生的新框架，而是一个有深厚搜索基因的成熟项目逐步拥抱 Agent 能力。

2019-2022 年：专注于文档搜索和问答系统，基于传统 NLP 模型（BERT、DPR）和搜索引擎（Elasticsearch）。这个阶段的 Haystack 是"NLP 框架"而非"AI Agent 框架"。

2023 年：引入 LLM 支持，拥抱 RAG 范式，开始 2.0 版本的全面重构。核心设计从"搜索管道"泛化为"任意数据处理管道"。

2024 年：Haystack 2.0 正式发布，全新的 pipeline 架构，组件系统完全重写，支持 Agent 组件。

2025 年：Agent 能力持续增强，但核心定位仍是"做好 RAG 的每一个环节"。

## Pipeline：核心抽象

Haystack 的一切都围绕 Pipeline 构建。Pipeline 是一个由组件（Component）组成的有向图，数据在组件间流动和转换。与 LangGraph 的 StateGraph 不同，Haystack Pipeline 更像数据处理管道——强调数据的流转和转换，而非控制流的编排。

```python
from haystack import Pipeline
from haystack.components.retrievers.in_memory import InMemoryBM25Retriever
from haystack.components.generators import OpenAIGenerator
from haystack.components.builders import PromptBuilder
from haystack.document_stores.in_memory import InMemoryDocumentStore
from haystack import Document

# 创建文档存储并索引文档
document_store = InMemoryDocumentStore()
document_store.write_documents([
    Document(content="向量搜索是基于语义相似度的检索技术，通过将文本转换为高维向量来计算相似度。"),
    Document(content="BM25 是传统的基于词频的检索算法，在精确匹配场景中表现优秀。"),
    Document(content="混合检索结合向量搜索和关键词搜索的优势，通常能获得更好的召回率。"),
])

# 定义 RAG 管道
rag_pipeline = Pipeline()
rag_pipeline.add_component("retriever", InMemoryBM25Retriever(document_store=document_store))
rag_pipeline.add_component("prompt_builder", PromptBuilder(
    template="""根据以下文档回答问题。如果文档中没有相关信息，请说明。
    
文档：
{% for doc in documents %}
- {{ doc.content }}
{% endfor %}

问题：{{ question }}
回答："""
))
rag_pipeline.add_component("llm", OpenAIGenerator(model="gpt-4o"))

# 连接组件
rag_pipeline.connect("retriever.documents", "prompt_builder.documents")
rag_pipeline.connect("prompt_builder", "llm")

# 运行管道
result = rag_pipeline.run({
    "retriever": {"query": "什么是向量搜索？"},
    "prompt_builder": {"question": "什么是向量搜索？"}
})
print(result["llm"]["replies"][0])
```

### 组件系统（Component）

Haystack 2.0 的组件是强类型的自包含处理单元。每个组件通过装饰器声明输入输出类型，这使得管道在构建时就能进行类型检查：

```python
from haystack import component
from haystack import Document
from typing import List

@component
class QueryExpander:
    """将用户查询扩展为多个变体以提升召回率"""
    
    @component.output_types(queries=List[str])
    def run(self, query: str, num_expansions: int = 3):
        # 实际实现中可以用 LLM 生成查询变体
        expanded = [
            query,
            f"{query} 的定义和原理",
            f"{query} 的应用场景",
        ]
        return {"queries": expanded[:num_expansions]}

@component
class DocumentFilter:
    """按相关性分数过滤文档"""
    
    @component.output_types(documents=List[Document])
    def run(self, documents: List[Document], min_score: float = 0.5):
        filtered = [doc for doc in documents if (doc.score or 0) >= min_score]
        return {"documents": filtered}
```

### 预置组件生态

Haystack 内置了覆盖 RAG 全流程的丰富组件：

文档处理层包括文件转换器（PDF、Word、HTML、Markdown）、文本分块器（多种策略）、文档清洗器。

嵌入层包括多种嵌入模型适配器（OpenAI、Cohere、Sentence-Transformers、Hugging Face）。

检索层包括向量检索器、BM25 检索器、混合检索器，支持 Elasticsearch、OpenSearch、Pinecone、Weaviate 等主流向量数据库。

生成层包括多模型适配器、提示模板构建器、输出解析器。

## Agent 能力

Haystack 2.x 通过 `ChatAgent` 和 Tool 组件提供了 Agent 能力。其设计理念是"Agent 是管道的一种运行模式"：

```python
from haystack.components.generators.chat import OpenAIChatGenerator
from haystack.dataclasses import ChatMessage
from haystack.tools import Tool

# 将 RAG 管道封装为工具
def search_knowledge_base(query: str) -> str:
    """在知识库中搜索相关信息"""
    result = rag_pipeline.run({
        "retriever": {"query": query},
        "prompt_builder": {"question": query}
    })
    return result["llm"]["replies"][0]

search_tool = Tool(
    name="search_knowledge_base",
    description="搜索内部知识库获取相关信息",
    function=search_knowledge_base,
    parameters={
        "type": "object",
        "properties": {"query": {"type": "string", "description": "搜索查询"}},
        "required": ["query"]
    }
)

# 创建带工具的聊天模型
chat_generator = OpenAIChatGenerator(
    model="gpt-4o",
    tools=[search_tool]
)
```

## 与 Agent 优先框架的差异

Haystack 与 LangGraph/CrewAI 的本质区别在于设计重心的不同。

LangGraph 的核心问题是"如何编排 Agent 的行为逻辑"——状态管理、条件路由、循环、人工干预都是一等公民。Haystack 的核心问题是"如何构建可靠的数据处理管道"——文档处理、检索优化、生成质量是设计重点，Agent 只是管道的一种运行模式。

这意味着在 RAG 密集型应用中，Haystack 的组件库（文档转换器、分块器、嵌入器、检索器、重排器等）比任何 Agent 框架都更丰富和成熟。但在需要复杂 Agent 编排逻辑的场景中，它的表达力确实不如 LangGraph。

## 优势

**RAG 能力无出其右**：从文档解析到检索优化，Haystack 覆盖了 RAG 管道的每一个环节，且每个组件都经过生产验证。支持十余种向量数据库和搜索引擎。

**模块化和可测试性**：强类型的组件系统使得单元测试和集成测试都很方便。每个组件可以独立测试，管道的行为完全由组件组合决定。

**企业搜索基因**：对 Elasticsearch、OpenSearch 等企业级搜索引擎有一流支持。deepset 本身的商业模式就是企业搜索解决方案。

**稳定性和向后兼容**：deepset 作为商业公司提供长期维护承诺，API 变化相对保守，不像某些框架频繁做破坏性变更。

**deepset Cloud**：商业版本提供托管服务、可视化管道编辑、评估工具等企业级功能。

## 局限

**Agent 能力不是强项**：Agent 相关功能是在 2.0 之后逐步加入的，成熟度和灵活性不如 LangGraph 等 Agent 优先框架。

**学习曲线**：Pipeline 的组件连接逻辑和类型系统需要适应期，不如 CrewAI 那样"5 分钟上手"。

**社区规模**：相比 LangChain 生态的庞大社区，Haystack 的社区和第三方集成数量较少。

**Python only**：不支持其他语言，对于 Java/.NET 企业生态的覆盖需要通过 REST API。

## 适用场景

Haystack 最适合以下场景：企业知识库问答系统（RAG 是核心需求）、文档搜索和语义检索应用、需要精细控制 RAG 管道各环节（分块策略、检索策略、重排策略）的项目、已有 Elasticsearch/OpenSearch 基础设施的企业、需要生产级 RAG 可靠性的商业应用。

一个简单的判断标准：如果你的应用是"90% RAG + 10% Agent"，Haystack 是最佳选择。如果是"90% Agent + 10% RAG"，建议选择 LangGraph 并集成 RAG 组件。如果是"50/50"，两者都可以，取决于团队熟悉度。

## 本章小结

Haystack 代表了从传统搜索/NLP 领域演进而来的 Agent 框架路线。它的价值不在于通用 Agent 编排能力的前沿性，而在于"做好 RAG 这一件事"的深度和可靠性。在企业搜索和知识管理场景中，Haystack 多年积累的组件库和生产经验是其不可替代的核心竞争力。对于需要构建可靠 RAG 系统的团队，Haystack 应该是首选评估对象。

## 延伸阅读

- [Haystack 官方文档](https://docs.haystack.deepset.ai/)
- [Haystack GitHub](https://github.com/deepset-ai/haystack)
- [deepset 博客](https://www.deepset.ai/blog)
- [Haystack 2.0 迁移指南](https://docs.haystack.deepset.ai/docs/migration-guide)
- [deepset Cloud](https://www.deepset.ai/deepset-cloud)
- [RAG 技术深度解析](../05-rag/) — 本书 RAG 章节
- [框架对比](./comparison-matrix.md) — Haystack 在框架生态中的位置
