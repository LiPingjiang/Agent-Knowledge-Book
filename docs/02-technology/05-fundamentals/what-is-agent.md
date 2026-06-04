<!-- last updated: 2025-06 -->
# Agent 的定义与本质

> 本章从多个视角定义 Agent，追溯概念源头，并厘清当下 LLM Agent 语境下的核心含义。

## 1. 一个词，多重含义

"Agent" 一词在不同语境下有截然不同的含义。在开始构建 Agent 之前，我们需要先理解这个概念的谱系。

**哲学层面**：Agent（行动者）指具有意向性（intentionality）的实体——它有信念、有欲望、有意图，并据此采取行动。这是最抽象的定义。

**经典 AI 层面**：Russell & Norvig 在《Artificial Intelligence: A Modern Approach》中给出了被引用最广的定义——

> An agent is anything that can be viewed as perceiving its environment through sensors and acting upon that environment through actuators.
> 
> （Agent 是任何能够通过传感器感知环境、并通过执行器对环境采取行动的实体。）
>
> [Russell & Norvig, 2020] Artificial Intelligence: A Modern Approach, 4th Edition.

这个定义极其宽泛：一个恒温器、一个软件爬虫、一个自动驾驶系统，都可以是 Agent。

**LLM Agent 层面**：当下工程语境中的 Agent，特指以大语言模型为核心推理引擎、能够自主规划和执行多步任务的系统。这是本知识大全的核心关注对象。

## 2. 当下主流定义对比

### 2.1 Anthropic 的定义（2024.12）

Anthropic 在《Building Effective Agents》中做了一个关键区分：

> - **Workflows（工作流）**：LLM 和工具通过预定义的代码路径进行编排的系统。
> - **Agents（智能体）**：LLM 动态地指导自身的处理过程和工具使用，保持对任务完成方式的控制权的系统。

两者统称为 **Agentic Systems（智能体系统）**。核心区分标准是：**谁在控制流程**——是预写的代码，还是 LLM 自身的判断。

Anthropic 进一步指出 Agent 的运行特征：
- 从人类指令或对话开始任务
- 独立规划和执行，必要时回到人类获取信息或判断
- 每一步从环境获取"ground truth"（如工具调用结果、代码执行输出）来评估进展
- 可在检查点暂停等待人类反馈
- 通常有终止条件（任务完成或最大迭代次数）

[Anthropic, 2024] Building Effective Agents. https://www.anthropic.com/research/building-effective-agents

### 2.2 OpenAI 的定义（2025.04）

OpenAI 在《A Practical Guide to Building Agents》中的定义：

> Agents are systems that independently accomplish tasks on your behalf.
>
> （Agent 是能够独立代表你完成任务的系统。）

OpenAI 强调 Agent 的三个核心特征：
1. **依赖 LLM 进行流程决策与控制**——能判断任务完成状态，必要时自主修正
2. **能够访问和使用工具**——搜索、代码执行、API 调用等
3. **基于环境反馈的闭环执行**——不是一次性生成，而是多轮迭代

OpenAI 明确指出不属于 Agent 的系统：简单聊天机器人、单轮 LLM 调用、情感分类器——这些集成了 LLM 但不用 LLM 控制工作流执行。

[OpenAI, 2025] A Practical Guide to Building Agents. https://platform.openai.com/docs/guides/agents

### 2.3 Andrew Ng 的视角（2024）

吴恩达在 2024 年 Snowflake 峰会上提出了 **Agentic Workflow（智能体工作流）** 的概念框架：

> Agentic workflows could drive more AI progress than even the next generation of foundation models.

他将 Agentic 设计模式归纳为四种核心能力：
1. **Reflection（反思）**——模型审视和改进自己的输出
2. **Tool Use（工具使用）**——调用外部工具扩展能力
3. **Planning（规划）**——将复杂任务分解为步骤
4. **Multi-agent Collaboration（多智能体协作）**——多个 Agent 分工合作

这个框架强调的不是"Agent 是什么"，而是"什么让系统具有 Agentic 特性"——这是一个**程度**问题，而非二元判断。

[Andrew Ng, 2024] AI Agentic Workflows and Their Potential. Snowflake Summit 2024.

### 2.4 学术界综合定义

Wang et al. 在对 LLM-based Agent 的综述中提出了一个统一框架：

> LLM-based autonomous agent is a system that leverages an LLM as its core computational engine, enabling it to exhibit capabilities of autonomous planning, acting, and learning in dynamic environments.
>
> （基于 LLM 的自主 Agent 是一个利用 LLM 作为核心计算引擎的系统，使其能够在动态环境中展现自主规划、行动和学习的能力。）

[Wang et al., 2024] A Survey on Large Language Model based Autonomous Agents. Frontiers of Computer Science.

## 3. Agent 的本质特征

综合以上定义，我们可以提炼出 LLM Agent 的核心本质特征：

### 3.1 自主性（Autonomy）

Agent 能够在没有人类逐步指导的情况下独立做出决策。这不意味着完全无人监督——而是说 Agent 自行决定"下一步做什么"。

自主性是一个**光谱**：
- 低自主性：按固定步骤执行（Workflow）
- 中自主性：在预设范围内灵活决策（受约束的 Agent）
- 高自主性：完全自主规划和执行（自主 Agent）

### 3.2 环境交互（Environment Interaction）

Agent 不是在真空中运行。它感知环境状态、对环境施加动作、并观察动作结果。这形成了一个 **感知-思考-行动（Perceive-Think-Act）** 的循环。

在 LLM Agent 中，"环境"通常是：
- 工具调用的返回结果
- 代码执行的输出
- 文件系统的状态
- API 响应
- 用户的对话输入

### 3.3 目标导向（Goal-Directedness）

Agent 的行为是为了达成某个目标。它不只是响应输入，而是持续朝目标推进，在遇到障碍时调整策略。

### 3.4 推理与规划（Reasoning & Planning）

LLM Agent 的核心能力来自 LLM 的推理能力。它能够：
- 将复杂任务分解为子任务
- 评估不同行动方案的优劣
- 基于中间结果调整计划
- 从失败中学习和恢复

### 3.5 工具使用（Tool Use）

LLM 本身只能生成文本。通过调用外部工具（搜索引擎、代码解释器、API、文件系统等），Agent 将文本生成能力转化为对现实世界的作用力。

### 3.6 记忆（Memory）

Agent 需要在多步执行过程中维持上下文、记住已完成的步骤、学习到的信息，并在长期任务中积累经验。

## 4. 形式化表达

从形式上，一个 LLM Agent 可以表示为：

```
Agent = LLM + Memory + Tools + Planning + Action Loop
```

更精确地：

```
在每个时间步 t：
  1. 观察 oₜ = Perceive(environment)
  2. 更新记忆 mₜ = Memory(mₜ₋₁, oₜ)
  3. 决策 aₜ = LLM(prompt, mₜ, tools)
  4. 执行 rₜ = Execute(aₜ, environment)
  5. 评估是否达成目标或需要继续
```

这个循环持续进行，直到任务完成、到达最大步数、或被人类中断。

## 5. Agent 不是什么

为了更清晰地界定 Agent 的边界，明确几个"不是"：

| 不是 Agent | 原因 |
|-----------|------|
| 单轮 ChatBot | 无多步执行、无目标导向 |
| RAG 系统 | 检索增强生成是一次性的，无规划和行动循环 |
| 固定 Pipeline | 代码控制流程，LLM 只是其中一个环节 |
| 情感分类器 | LLM 作为分类工具，非流程控制者 |
| Prompt Chain | 如果步骤是预定义的且无动态分支，则是 Workflow 而非 Agent |

需要注意：边界并非绝对。一个 RAG 系统如果加入了"判断是否需要追加检索"的逻辑，就开始具有 Agentic 特性。这也是为什么 Andrew Ng 强调 "Agentic" 是一个程度问题。

## 6. 从工程视角看 Agent

对于软件工程师而言，Agent 本质上是一种**新的程序执行范式**：

| 传统软件 | Agent |
|---------|-------|
| 确定性执行 | 概率性决策 |
| 编译时确定流程 | 运行时动态决定流程 |
| Bug = 代码错误 | 失败 = 决策失误 + 环境误解 |
| 测试：输入→输出 | 测试：场景→行为模式 |
| 性能瓶颈：CPU/IO | 性能瓶颈：LLM 延迟 + Token 成本 |
| 扩展：加机器 | 扩展：优化 Prompt + 工具设计 |

这意味着构建 Agent 需要一套不同于传统软件开发的心智模型和工程实践——这也是本知识大全后续章节要详细展开的内容。

## 参考文献

- [Russell & Norvig, 2020] Artificial Intelligence: A Modern Approach, 4th Edition. Pearson.
- [Anthropic, 2024] Building Effective Agents. https://www.anthropic.com/research/building-effective-agents
- [OpenAI, 2025] A Practical Guide to Building Agents. https://platform.openai.com/docs/guides/agents
- [Andrew Ng, 2024] AI Agentic Workflows and Their Potential. Snowflake Summit 2024.
- [Wang et al., 2024] A Survey on Large Language Model based Autonomous Agents. Frontiers of Computer Science, 18(6).
- [Yao et al., 2023] ReAct: Synergizing Reasoning and Acting in Language Models. ICLR 2023.
- [Wooldridge & Jennings, 1995] Intelligent Agents: Theory and Practice. The Knowledge Engineering Review, 10(2).
