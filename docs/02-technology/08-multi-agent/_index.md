# Multi-Agent 系统

本章探讨多 Agent 协作系统的设计与实现，涵盖主流协作模式、通信机制、任务分配策略等核心议题。Multi-Agent 系统通过多个专业化 Agent 的协同工作，能够解决单 Agent 难以胜任的复杂问题。

## 本章内容

- [协作模式总览](./collaboration-patterns.md) — Multi-Agent 协作的主要范式与适用场景
- [Orchestrator-Worker 模式](./orchestrator-worker.md) — 中心编排者分配任务给工作者的模式
- [辩论与讨论模式](./debate-and-discussion.md) — 多 Agent 通过辩论达成共识或提升质量
- [层级式组织](./hierarchical.md) — 树形管理结构的多层 Agent 组织
- [对等协作](./peer-to-peer.md) — 无中心的平等协作模式
- [通信协议与消息传递](./communication-protocols.md) — Agent 间的通信机制设计
- [共享记忆与知识库](./shared-memory.md) — 多 Agent 共享信息的机制
- [任务分解与分配策略](./task-decomposition.md) — 如何将复杂任务拆分并分配给合适的 Agent
- [冲突解决机制](./conflict-resolution.md) — 多 Agent 意见分歧时的解决策略
- [生产部署实证](./production-deployment.md) — MAS 从 PoC 到生产的核心挑战与行业案例
- [A2A 协议 (Agent-to-Agent)](./a2a-protocol.md) — Google A2A 开放协议：Agent Card、发现机制与跨框架互操作
- [MCP 协议深度解析](./mcp-and-protocols.md) — stdio/SSE/Streamable HTTP 传输、四种原语本质、Sampling 反向推理、能力协商、产业落地分析
