# Agent Knowledge Book · Agent 知识大全

面向「软件研发工程师转型 Agent 研发」的系统性知识库。

## 项目定位

本项目是一部 Agent 领域的知识大全，覆盖从历史沿革到当下前沿技术的完整知识体系。目标读者为有软件工程背景、正在或即将转型 Agent 方向的工程师。

## 语言约定

- 正文以中文撰写
- 英文术语首次出现时标注原文，后续可直接使用
- 代码示例、配置、CLI 命令保留英文

---

## 目录

> 点击链接即可进入对应章节。每个子目录的 `_index.md` 是该章节的导读页。

### [总览](docs/_index.md)

---

### Part I · Agent 史学

#### [第 1 章 · 前 LLM 时代 (1950s–2020)](docs/01-history/01-pre-llm-era/_index.md)

| # | 主题 | 文件 |
|---|------|------|
| 1.1 | 符号 AI 与早期智能体 | [symbolic-ai-agents.md](docs/01-history/01-pre-llm-era/symbolic-ai-agents.md) |
| 1.2 | 符号 AI 时代总论 | [symbolic-ai-era.md](docs/01-history/01-pre-llm-era/symbolic-ai-era.md) |
| 1.3 | BDI 架构与理性 Agent | [bdi-architecture.md](docs/01-history/01-pre-llm-era/bdi-architecture.md) |
| 1.4 | 反应式与 BDI 混合架构 | [reactive-and-bdi.md](docs/01-history/01-pre-llm-era/reactive-and-bdi.md) |
| 1.5 | 认知架构：SOAR 与 ACT-R | [cognitive-architectures.md](docs/01-history/01-pre-llm-era/cognitive-architectures.md) |
| 1.6 | 多智能体系统 | [multi-agent-systems.md](docs/01-history/01-pre-llm-era/multi-agent-systems.md) |
| 1.7 | 软件 Agent 与中间件 | [software-agents.md](docs/01-history/01-pre-llm-era/software-agents.md) |
| 1.8 | 强化学习 Agent | [reinforcement-learning-agents.md](docs/01-history/01-pre-llm-era/reinforcement-learning-agents.md) |
| 1.9 | RL Agent 补充 | [rl-agents.md](docs/01-history/01-pre-llm-era/rl-agents.md) |
| 1.10 | 对话系统演进 | [dialogue-systems.md](docs/01-history/01-pre-llm-era/dialogue-systems.md) |

#### [第 2 章 · LLM Agent 的崛起 (2022–2023)](docs/01-history/02-llm-agent-rise/_index.md)

| # | 主题 | 文件 |
|---|------|------|
| 2.1 | 基础模型的涌现能力 | [foundation-models.md](docs/01-history/02-llm-agent-rise/foundation-models.md) |
| 2.2 | 思维链 (Chain-of-Thought) | [chain-of-thought.md](docs/01-history/02-llm-agent-rise/chain-of-thought.md) |
| 2.3 | ReAct 范式 | [react-paradigm.md](docs/01-history/02-llm-agent-rise/react-paradigm.md) |
| 2.4 | Toolformer 与工具使用 | [toolformer-and-tool-use.md](docs/01-history/02-llm-agent-rise/toolformer-and-tool-use.md) |
| 2.5 | AutoGPT 与 BabyAGI | [autogpt-and-babyagi.md](docs/01-history/02-llm-agent-rise/autogpt-and-babyagi.md) |
| 2.6 | 生成式 Agent (斯坦福小镇) | [generative-agents.md](docs/01-history/02-llm-agent-rise/generative-agents.md) |
| 2.7 | 早期框架生态 | [early-frameworks.md](docs/01-history/02-llm-agent-rise/early-frameworks.md) |

#### [第 3 章 · Agent 工程化时代 (2024–2025)](docs/01-history/03-agent-engineering/_index.md)

| # | 主题 | 文件 |
|---|------|------|
| 3.1 | 框架爆发 | [framework-explosion.md](docs/01-history/03-agent-engineering/framework-explosion.md) |
| 3.2 | Coding Agent 演进 | [coding-agents.md](docs/01-history/03-agent-engineering/coding-agents.md) |
| 3.3 | 企业级落地 | [enterprise-adoption.md](docs/01-history/03-agent-engineering/enterprise-adoption.md) |
| 3.4 | Agent 即产品 | [agent-as-product.md](docs/01-history/03-agent-engineering/agent-as-product.md) |
| 3.5 | MCP/A2A 协议标准化 | [mcp-and-protocols.md](docs/01-history/03-agent-engineering/mcp-and-protocols.md) |
| 3.6 | 开源生态 | [open-source-ecosystem.md](docs/01-history/03-agent-engineering/open-source-ecosystem.md) |

#### [第 4 章 · 历史经验与教训](docs/01-history/04-lessons-learned/_index.md)

| # | 主题 | 文件 |
|---|------|------|
| 4.1 | 自主性 vs 可控性 | [autonomy-vs-control.md](docs/01-history/04-lessons-learned/autonomy-vs-control.md) |
| 4.2 | 规划的局限与失败 | [planning-failures.md](docs/01-history/04-lessons-learned/planning-failures.md) |
| 4.3 | 记忆系统的挑战 | [memory-challenges.md](docs/01-history/04-lessons-learned/memory-challenges.md) |
| 4.4 | 可靠性工程 | [reliability-engineering.md](docs/01-history/04-lessons-learned/reliability-engineering.md) |
| 4.5 | 人机交互演进 | [human-agent-interaction.md](docs/01-history/04-lessons-learned/human-agent-interaction.md) |
| 4.6 | 成本与延迟权衡 | [cost-and-latency.md](docs/01-history/04-lessons-learned/cost-and-latency.md) |

---

### Part II · Agent 技术体系

#### [第 5 章 · Agent 基础理论](docs/02-technology/05-fundamentals/_index.md)

| # | 主题 | 文件 |
|---|------|------|
| 5.1 | 什么是 Agent | [what-is-agent.md](docs/02-technology/05-fundamentals/what-is-agent.md) |
| 5.2 | Agent 分类学 | [taxonomy.md](docs/02-technology/05-fundamentals/taxonomy.md) |
| 5.3 | 五层能力模型 | [capability-model.md](docs/02-technology/05-fundamentals/capability-model.md) |
| 5.4 | Agent vs Pipeline | [agent-vs-pipeline.md](docs/02-technology/05-fundamentals/agent-vs-pipeline.md) |
| 5.5 | Agentic 模式总论 | [agentic-patterns.md](docs/02-technology/05-fundamentals/agentic-patterns.md) |

#### [第 6 章 · Agent 架构模式](docs/02-technology/06-architecture/_index.md)

| # | 主题 | 文件 |
|---|------|------|
| 6.1 | 单 Agent 循环 | [single-agent-loop.md](docs/02-technology/06-architecture/single-agent-loop.md) |
| 6.2 | 状态机架构 | [state-machine.md](docs/02-technology/06-architecture/state-machine.md) |
| 6.3 | DAG 工作流 | [dag-workflow.md](docs/02-technology/06-architecture/dag-workflow.md) |
| 6.4 | 事件驱动架构 | [event-driven.md](docs/02-technology/06-architecture/event-driven.md) |
| 6.5 | 路由架构 | [router-architecture.md](docs/02-technology/06-architecture/router-architecture.md) |
| 6.6 | 黑板架构 | [blackboard-architecture.md](docs/02-technology/06-architecture/blackboard-architecture.md) |
| 6.7 | Human-in-the-Loop | [human-in-the-loop.md](docs/02-technology/06-architecture/human-in-the-loop.md) |
| 6.8 | 微服务化 Agent | [microservice-agents.md](docs/02-technology/06-architecture/microservice-agents.md) |

#### [第 7 章 · 核心模块详解](docs/02-technology/07-core-modules/_index.md)

| # | 主题 | 文件 |
|---|------|------|
| 7.1 | 规划 (Planning) | [planning.md](docs/02-technology/07-core-modules/planning.md) |
| 7.2 | 记忆 (Memory) | [memory.md](docs/02-technology/07-core-modules/memory.md) |
| 7.3 | 工具使用 (Tool Use) | [tool-use.md](docs/02-technology/07-core-modules/tool-use.md) |
| 7.4 | 推理 (Reasoning) | [reasoning.md](docs/02-technology/07-core-modules/reasoning.md) |
| 7.5 | 反思与自我修正 | [reflection.md](docs/02-technology/07-core-modules/reflection.md) |
| 7.6 | 上下文管理 | [context-management.md](docs/02-technology/07-core-modules/context-management.md) |
| 7.7 | 行动执行 | [action-execution.md](docs/02-technology/07-core-modules/action-execution.md) |
| 7.8 | 感知与多模态 | [perception.md](docs/02-technology/07-core-modules/perception.md) |
| 7.9 | Prompt 工程 | [prompt-engineering.md](docs/02-technology/07-core-modules/prompt-engineering.md) |
| 7.10 | 错误恢复 | [error-recovery.md](docs/02-technology/07-core-modules/error-recovery.md) |

#### [第 8 章 · Multi-Agent 系统](docs/02-technology/08-multi-agent/_index.md)

| # | 主题 | 文件 |
|---|------|------|
| 8.1 | 协作模式总览 | [collaboration-patterns.md](docs/02-technology/08-multi-agent/collaboration-patterns.md) |
| 8.2 | Orchestrator-Worker | [orchestrator-worker.md](docs/02-technology/08-multi-agent/orchestrator-worker.md) |
| 8.3 | 层级式组织 | [hierarchical.md](docs/02-technology/08-multi-agent/hierarchical.md) |
| 8.4 | 对等协作 (P2P) | [peer-to-peer.md](docs/02-technology/08-multi-agent/peer-to-peer.md) |
| 8.5 | 辩论与讨论 | [debate-and-discussion.md](docs/02-technology/08-multi-agent/debate-and-discussion.md) |
| 8.6 | 通信协议 | [communication-protocols.md](docs/02-technology/08-multi-agent/communication-protocols.md) |
| 8.7 | 共享记忆 | [shared-memory.md](docs/02-technology/08-multi-agent/shared-memory.md) |
| 8.8 | 任务分解与分配 | [task-decomposition.md](docs/02-technology/08-multi-agent/task-decomposition.md) |
| 8.9 | 冲突解决 | [conflict-resolution.md](docs/02-technology/08-multi-agent/conflict-resolution.md) |

#### [第 9 章 · 框架与产品全景](docs/02-technology/09-frameworks/_index.md)

| # | 主题 | 文件 |
|---|------|------|
| 9.1 | 框架分类与选型 | [classification.md](docs/02-technology/09-frameworks/classification.md) |
| 9.2 | 框架对比矩阵 | [comparison-matrix.md](docs/02-technology/09-frameworks/comparison-matrix.md) |
| 9.3 | LangChain / LangGraph | [langchain-langgraph.md](docs/02-technology/09-frameworks/langchain-langgraph.md) |
| 9.4 | OpenAI Agents SDK | [openai-agents-sdk.md](docs/02-technology/09-frameworks/openai-agents-sdk.md) |
| 9.5 | Anthropic Claude Agent | [anthropic-claude-agent.md](docs/02-technology/09-frameworks/anthropic-claude-agent.md) |
| 9.6 | AutoGen | [autogen.md](docs/02-technology/09-frameworks/autogen.md) |
| 9.7 | CrewAI | [crewai.md](docs/02-technology/09-frameworks/crewai.md) |
| 9.8 | MetaGPT | [metagpt.md](docs/02-technology/09-frameworks/metagpt.md) |
| 9.9 | CAMEL | [camel.md](docs/02-technology/09-frameworks/camel.md) |
| 9.10 | Semantic Kernel | [semantic-kernel.md](docs/02-technology/09-frameworks/semantic-kernel.md) |
| 9.11 | Dify | [dify.md](docs/02-technology/09-frameworks/dify.md) |
| 9.12 | Coze | [coze.md](docs/02-technology/09-frameworks/coze.md) |
| 9.13 | AutoGPT (框架版) | [autogpt-framework.md](docs/02-technology/09-frameworks/autogpt-framework.md) |
| 9.14 | Haystack | [haystack.md](docs/02-technology/09-frameworks/haystack.md) |
| 9.15 | Agent 产品全景 | [agent-products.md](docs/02-technology/09-frameworks/agent-products.md) |

#### [第 10 章 · 评测体系](docs/02-technology/10-evaluation/_index.md)

| # | 主题 | 文件 |
|---|------|------|
| 10.1 | 评测方法论 | [methodology.md](docs/02-technology/10-evaluation/methodology.md) |
| 10.2 | SWE-bench | [swe-bench.md](docs/02-technology/10-evaluation/swe-bench.md) |
| 10.3 | WebArena | [webarena.md](docs/02-technology/10-evaluation/webarena.md) |
| 10.4 | GAIA | [gaia.md](docs/02-technology/10-evaluation/gaia.md) |
| 10.5 | AgentBench | [agentbench.md](docs/02-technology/10-evaluation/agentbench.md) |
| 10.6 | ToolBench | [toolbench.md](docs/02-technology/10-evaluation/toolbench.md) |
| 10.7 | HumanEval-Agent | [humaneval-agent.md](docs/02-technology/10-evaluation/humaneval-agent.md) |
| 10.8 | 安全性评测 | [safety-evaluation.md](docs/02-technology/10-evaluation/safety-evaluation.md) |
| 10.9 | 工程实践指标 | [real-world-metrics.md](docs/02-technology/10-evaluation/real-world-metrics.md) |

#### [第 11 章 · 安全与对齐](docs/02-technology/11-safety/_index.md)

| # | 主题 | 文件 |
|---|------|------|
| 11.1 | 威胁模型 | [threat-model.md](docs/02-technology/11-safety/threat-model.md) |
| 11.2 | Prompt 注入防御 | [prompt-injection.md](docs/02-technology/11-safety/prompt-injection.md) |
| 11.3 | 权限控制与沙箱 | [permission-control.md](docs/02-technology/11-safety/permission-control.md) |
| 11.4 | 输出验证与护栏 | [output-validation.md](docs/02-technology/11-safety/output-validation.md) |
| 11.5 | 审计与日志 | [audit-and-logging.md](docs/02-technology/11-safety/audit-and-logging.md) |
| 11.6 | 对齐策略 | [alignment-strategies.md](docs/02-technology/11-safety/alignment-strategies.md) |

#### [第 12 章 · 工程实践](docs/02-technology/12-engineering/_index.md)

| # | 主题 | 文件 |
|---|------|------|
| 12.1 | 开发流程 | [development-workflow.md](docs/02-technology/12-engineering/development-workflow.md) |
| 12.2 | 测试策略 | [testing-strategies.md](docs/02-technology/12-engineering/testing-strategies.md) |
| 12.3 | 可观测性 | [observability.md](docs/02-technology/12-engineering/observability.md) |
| 12.4 | 成本优化 | [cost-optimization.md](docs/02-technology/12-engineering/cost-optimization.md) |
| 12.5 | 部署架构 | [deployment.md](docs/02-technology/12-engineering/deployment.md) |
| 12.6 | 调试技巧 | [debugging.md](docs/02-technology/12-engineering/debugging.md) |
| 12.7 | 版本管理 | [versioning.md](docs/02-technology/12-engineering/versioning.md) |

---

### 附录

| 主题 | 文件 |
|------|------|
| 术语表 | [glossary.md](docs/appendix/glossary.md) |
| 框架对比总表 | [framework-comparison.md](docs/appendix/framework-comparison.md) |
| 参考文献 | [references.md](docs/appendix/references.md) |
| Deep Research 评审记录 | [deep-research-review.md](docs/appendix/deep-research-review.md) |

---

## 项目元信息

- 创建时间：2025-06
- 维护方式：手动补充 + Deep Research Agent 辅助
- 知识载体：纯 Markdown 文档树
- 组织方式：按主题树状结构，README 为导航入口
- 统计：12 章 + 附录，约 120 个文件，23,000+ 行内容
