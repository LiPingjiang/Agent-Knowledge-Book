# Agent 知识大全

> 面向软件工程师的 Agent 技术完全指南：从历史源流到工程实战。

---

## 上篇 · Agent 史学

### [第一章 前 LLM 时代 (1950s–2020)](docs/01-history/01-pre-llm-era/_index.md)

从符号 AI 的逻辑推理，到 BDI 的信念-愿望-意图模型，再到认知架构 SOAR 与 ACT-R 的统一理论尝试；从多智能体博弈到强化学习 Agent 的崛起——回溯 Agent 概念在 LLM 诞生之前的七十年演化。

- [符号 AI 与逻辑推理时代](docs/01-history/01-pre-llm-era/symbolic-ai-era.md)
- [BDI 架构：信念、愿望与意图](docs/01-history/01-pre-llm-era/bdi-architecture.md)
- [反应式与混合架构](docs/01-history/01-pre-llm-era/reactive-and-bdi.md)
- [认知架构：SOAR 与 ACT-R](docs/01-history/01-pre-llm-era/cognitive-architectures.md)
- [多智能体系统](docs/01-history/01-pre-llm-era/multi-agent-systems.md)
- [强化学习 Agent](docs/01-history/01-pre-llm-era/rl-agents.md)
- [对话系统的演进](docs/01-history/01-pre-llm-era/dialogue-systems.md)

### [第二章 LLM Agent 的崛起 (2022–2023)](docs/01-history/02-llm-agent-rise/_index.md)

GPT-3.5 的涌现能力点燃了 Agent 革命。从思维链推理到 ReAct 范式，从 Toolformer 的工具学习到 AutoGPT 的自主循环，再到斯坦福小镇的社会模拟——这一年改变了一切。

- [基础模型的涌现能力](docs/01-history/02-llm-agent-rise/foundation-models.md)
- [思维链 (Chain-of-Thought)](docs/01-history/02-llm-agent-rise/chain-of-thought.md)
- [ReAct：推理与行动的统一](docs/01-history/02-llm-agent-rise/react-paradigm.md)
- [Toolformer 与工具使用](docs/01-history/02-llm-agent-rise/toolformer-and-tool-use.md)
- [AutoGPT 与 BabyAGI](docs/01-history/02-llm-agent-rise/autogpt-and-babyagi.md)
- [生成式 Agent 与斯坦福小镇](docs/01-history/02-llm-agent-rise/generative-agents.md)
- [早期框架生态](docs/01-history/02-llm-agent-rise/early-frameworks.md)

### [第三章 Agent 工程化时代 (2024–2025)](docs/01-history/03-agent-engineering/_index.md)

Agent 从实验走向生产。框架爆发、协议标准化 (MCP/A2A)、Coding Agent 登上舞台、企业级落地方案逐步成熟。

- [框架爆发](docs/01-history/03-agent-engineering/framework-explosion.md)
- [Coding Agent 的演进](docs/01-history/03-agent-engineering/coding-agents.md)
- [企业级落地实践](docs/01-history/03-agent-engineering/enterprise-adoption.md)
- [Agent 即产品](docs/01-history/03-agent-engineering/agent-as-product.md)
- [MCP/A2A 协议标准化](docs/01-history/03-agent-engineering/mcp-and-protocols.md)
- [开源生态全景](docs/01-history/03-agent-engineering/open-source-ecosystem.md)

### [第四章 历史经验与教训](docs/01-history/04-lessons-learned/_index.md)

七十年 Agent 研究积累的宝贵教训：自主性的边界在哪里？规划为什么会失败？记忆系统有哪些设计困境？如何在成本与能力之间找到平衡？

- [自主性与可控性的平衡](docs/01-history/04-lessons-learned/autonomy-vs-control.md)
- [规划能力的局限与突破](docs/01-history/04-lessons-learned/planning-failures.md)
- [记忆系统的设计困境](docs/01-history/04-lessons-learned/memory-challenges.md)
- [可靠性工程教训](docs/01-history/04-lessons-learned/reliability-engineering.md)
- [人机交互模式的演进](docs/01-history/04-lessons-learned/human-agent-interaction.md)
- [成本与延迟的工程权衡](docs/01-history/04-lessons-learned/cost-and-latency.md)

---

## 下篇 · Agent 技术体系

### [第五章 Agent 基础理论](docs/02-technology/05-fundamentals/_index.md)

什么是 Agent？如何分类？与传统 Pipeline 有何本质区别？从定义出发，建立完整的概念框架。

- [什么是 Agent](docs/02-technology/05-fundamentals/what-is-agent.md)
- [Agent 分类学](docs/02-technology/05-fundamentals/taxonomy.md)
- [五层能力模型](docs/02-technology/05-fundamentals/capability-model.md)
- [Agent vs Pipeline](docs/02-technology/05-fundamentals/agent-vs-pipeline.md)
- [Agentic 模式总论](docs/02-technology/05-fundamentals/agentic-patterns.md)

### [第六章 架构模式](docs/02-technology/06-architecture/_index.md)

从单 Agent 循环到事件驱动、从状态机到 DAG 工作流——Agent 系统的架构语言。

- [单 Agent 循环](docs/02-technology/06-architecture/single-agent-loop.md)
- [状态机架构](docs/02-technology/06-architecture/state-machine.md)
- [DAG 工作流](docs/02-technology/06-architecture/dag-workflow.md)
- [事件驱动架构](docs/02-technology/06-architecture/event-driven.md)
- [路由架构](docs/02-technology/06-architecture/router-architecture.md)
- [黑板架构](docs/02-technology/06-architecture/blackboard-architecture.md)
- [Human-in-the-Loop](docs/02-technology/06-architecture/human-in-the-loop.md)
- [微服务化 Agent](docs/02-technology/06-architecture/microservice-agents.md)

### [第七章 核心模块详解](docs/02-technology/07-core-modules/_index.md)

Agent 的"大脑"由哪些模块组成？规划、记忆、工具使用、推理、反思——逐一拆解。

- [规划 (Planning)](docs/02-technology/07-core-modules/planning.md)
- [记忆 (Memory)](docs/02-technology/07-core-modules/memory.md)
- [工具使用 (Tool Use)](docs/02-technology/07-core-modules/tool-use.md)
- [推理 (Reasoning)](docs/02-technology/07-core-modules/reasoning.md)
- [反思与自我修正](docs/02-technology/07-core-modules/reflection.md)
- [上下文管理](docs/02-technology/07-core-modules/context-management.md)
- [行动执行](docs/02-technology/07-core-modules/action-execution.md)
- [感知与多模态](docs/02-technology/07-core-modules/perception.md)
- [Prompt 工程](docs/02-technology/07-core-modules/prompt-engineering.md)
- [错误恢复](docs/02-technology/07-core-modules/error-recovery.md)

### [第八章 Multi-Agent 系统](docs/02-technology/08-multi-agent/_index.md)

当多个 Agent 协同工作——编排模式、通信协议、共享记忆、冲突解决。

- [协作模式总览](docs/02-technology/08-multi-agent/collaboration-patterns.md)
- [Orchestrator-Worker 模式](docs/02-technology/08-multi-agent/orchestrator-worker.md)
- [层级式组织](docs/02-technology/08-multi-agent/hierarchical.md)
- [对等协作 (P2P)](docs/02-technology/08-multi-agent/peer-to-peer.md)
- [辩论与讨论](docs/02-technology/08-multi-agent/debate-and-discussion.md)
- [通信协议](docs/02-technology/08-multi-agent/communication-protocols.md)
- [共享记忆](docs/02-technology/08-multi-agent/shared-memory.md)
- [任务分解与分配](docs/02-technology/08-multi-agent/task-decomposition.md)
- [冲突解决](docs/02-technology/08-multi-agent/conflict-resolution.md)

### [第九章 框架与产品全景](docs/02-technology/09-frameworks/_index.md)

主流框架逐一深入：设计哲学、架构特点、适用场景、代码实战。

- [框架分类与选型指南](docs/02-technology/09-frameworks/classification.md)
- [框架对比矩阵](docs/02-technology/09-frameworks/comparison-matrix.md)
- [LangChain / LangGraph](docs/02-technology/09-frameworks/langchain-langgraph.md)
- [Google ADK](docs/02-technology/09-frameworks/google-adk.md)
- [OpenAI Agents SDK](docs/02-technology/09-frameworks/openai-agents-sdk.md)
- [Anthropic Claude Agent](docs/02-technology/09-frameworks/anthropic-claude-agent.md)
- [AutoGen](docs/02-technology/09-frameworks/autogen.md)
- [CrewAI](docs/02-technology/09-frameworks/crewai.md)
- [MetaGPT](docs/02-technology/09-frameworks/metagpt.md)
- [CAMEL](docs/02-technology/09-frameworks/camel.md)
- [Semantic Kernel](docs/02-technology/09-frameworks/semantic-kernel.md)
- [Dify](docs/02-technology/09-frameworks/dify.md)
- [Coze](docs/02-technology/09-frameworks/coze.md)
- [AutoGPT 框架版](docs/02-technology/09-frameworks/autogpt-framework.md)
- [Haystack](docs/02-technology/09-frameworks/haystack.md)
- [Agent 产品全景图](docs/02-technology/09-frameworks/agent-products.md)

### [第十章 评测体系](docs/02-technology/10-evaluation/_index.md)

如何衡量 Agent 的能力？从 SWE-bench 到 WebArena，从学术 Benchmark 到工程实践指标。

- [评测方法论](docs/02-technology/10-evaluation/methodology.md)
- [SWE-bench](docs/02-technology/10-evaluation/swe-bench.md)
- [WebArena](docs/02-technology/10-evaluation/webarena.md)
- [GAIA](docs/02-technology/10-evaluation/gaia.md)
- [AgentBench](docs/02-technology/10-evaluation/agentbench.md)
- [ToolBench](docs/02-technology/10-evaluation/toolbench.md)
- [HumanEval-Agent](docs/02-technology/10-evaluation/humaneval-agent.md)
- [安全性评测](docs/02-technology/10-evaluation/safety-evaluation.md)
- [工程实践指标](docs/02-technology/10-evaluation/real-world-metrics.md)

### [第十一章 安全与对齐](docs/02-technology/11-safety/_index.md)

Agent 越强大，安全越重要。从威胁建模到 Prompt 注入防御，从权限沙箱到输出护栏。

- [威胁模型](docs/02-technology/11-safety/threat-model.md)
- [Prompt 注入防御](docs/02-technology/11-safety/prompt-injection.md)
- [权限控制与沙箱](docs/02-technology/11-safety/permission-control.md)
- [输出验证与护栏](docs/02-technology/11-safety/output-validation.md)
- [审计与日志](docs/02-technology/11-safety/audit-and-logging.md)
- [对齐策略](docs/02-technology/11-safety/alignment-strategies.md)

### [第十二章 工程实践](docs/02-technology/12-engineering/_index.md)

从开发到上线的全流程：测试策略、可观测性、成本优化、部署架构、调试技巧。

- [开发流程](docs/02-technology/12-engineering/development-workflow.md)
- [测试策略](docs/02-technology/12-engineering/testing-strategies.md)
- [可观测性](docs/02-technology/12-engineering/observability.md)
- [成本优化](docs/02-technology/12-engineering/cost-optimization.md)
- [部署架构](docs/02-technology/12-engineering/deployment.md)
- [调试技巧](docs/02-technology/12-engineering/debugging.md)
- [版本管理](docs/02-technology/12-engineering/versioning.md)

---

## 附录

- [术语表](docs/appendix/glossary.md)
- [框架对比总表](docs/appendix/framework-comparison.md)
- [参考文献](docs/appendix/references.md)

---

## 版权声明

**Copyright © 2025 李平江. All Rights Reserved.**

本作品采用最严格的专有许可证发布。未经版权持有人书面授权，禁止以任何形式复制、分发、传播、修改或商业使用本作品的任何部分。详见 [LICENSE](LICENSE)。
