<!-- last updated: 2025-06 -->
# Agent 基础理论

本章是整个知识大全的概念基石。在深入架构、模块、框架之前，先建立对 Agent 的系统性认知——它是什么、有哪些类型、具备什么能力、与传统方案有何不同、有哪些可复用的设计模式。

## 本章内容

- [Agent 的定义与本质](./what-is-agent.md) — 从哲学、经典 AI、LLM 三个层面定义 Agent，对比 Anthropic/OpenAI/学术界的定义，提炼核心本质特征
- [Agent 分类学](./taxonomy.md) — 按自主程度、组织结构、应用领域、决策机制、时间跨度、知识来源六个维度建立分类体系
- [核心能力模型](./capability-model.md) — 五层能力架构：基础能力（LLM）→ 扩展能力（工具）→ 认知能力（规划）→ 协作能力（社会）→ 学习能力（进化）
- [Agent 与传统 Pipeline 的区别](./agent-vs-pipeline.md) — 控制权转移、确定性差异、工程挑战对比，以及从 Pipeline 到 Agent 的渐进演进路径
- [Agentic 模式总论](./agentic-patterns.md) — 从 Prompt Chaining 到 Multi-Agent Collaboration，主流设计模式的全景图与选型决策框架

## 阅读建议

建议按顺序阅读：先从「定义与本质」建立概念，再通过「分类学」形成全景认知，然后理解「能力模型」掌握 Agent 的能力边界，接着通过「与 Pipeline 的区别」建立工程直觉，最后在「Agentic 模式」中获得可复用的设计方案。

## 前置知识

- 基本的软件工程概念（API、设计模式、分布式系统）
- 对 LLM 的基本了解（Prompt、Token、上下文窗口）
- 不需要 Agent 领域的先验知识（本章会从零建立）
