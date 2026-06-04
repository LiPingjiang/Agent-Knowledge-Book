# Agent 架构模式

本章深入探讨 Agent 系统的各种架构设计模式，从最基础的单 Agent 循环到复杂的事件驱动和微服务化架构。每种架构模式都有其适用场景、优势与局限性，帮助读者根据实际需求选择合适的架构方案。

## 本章内容

- [单 Agent 循环架构 (Perceive-Think-Act)](./single-agent-loop.md) — 最基础的 Agent 运行循环模型
- [路由架构](./router-architecture.md) — 基于意图识别的请求分发架构
- [DAG 工作流架构](./dag-workflow.md) — 有向无环图驱动的任务编排
- [事件驱动架构](./event-driven.md) — 基于事件发布/订阅的异步 Agent 架构
- [黑板架构](./blackboard-architecture.md) — 共享知识空间的协作式架构
- [微服务化 Agent](./microservice-agents.md) — 将 Agent 能力拆分为独立服务
- [Human-in-the-Loop 架构](./human-in-the-loop.md) — 人机协作的架构设计
- [状态机架构](./state-machine.md) — 基于有限状态机的 Agent 控制流
