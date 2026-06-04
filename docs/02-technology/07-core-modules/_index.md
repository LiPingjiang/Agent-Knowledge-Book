# 核心模块详解

本章详细解析构成 Agent 系统的各个核心模块，包括规划、记忆、工具使用、推理、反思等关键组件。理解这些模块的工作原理和设计要点，是构建高质量 Agent 系统的基础。

## 本章内容

- [规划模块 (Planning)](./planning.md) — Agent 如何将复杂目标分解为可执行的步骤序列
- [记忆模块 (Memory)](./memory.md) — 短期记忆、长期记忆与工作记忆的设计
- [工具使用 (Tool Use)](./tool-use.md) — Agent 如何发现、选择和调用外部工具
- [推理模块 (Reasoning)](./reasoning.md) — Chain-of-Thought、Tree-of-Thought 等推理策略
- [反思与自我修正 (Reflection)](./reflection.md) — Agent 的自我评估与迭代改进机制
- [上下文管理 (Context Window)](./context-management.md) — 有限上下文窗口下的信息管理策略
- [行动执行 (Action Execution)](./action-execution.md) — 从决策到实际执行的桥接层设计
- [感知模块 (Perception/Grounding)](./perception.md) — 多模态输入的理解与接地
- [Agent Prompt 工程](./prompt-engineering.md) — 面向 Agent 场景的 Prompt 设计方法
- [错误恢复与容错](./error-recovery.md) — Agent 执行失败时的恢复策略
