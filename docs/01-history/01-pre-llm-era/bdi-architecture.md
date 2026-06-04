# BDI 架构与理性 Agent (1987-2000s)

> Belief-Desire-Intention 模型是理性 Agent 设计的理论基石，深刻影响了从传统多智能体系统到现代 LLM Agent 的架构演进。

## 理论起源

BDI 模型并非一种具体的软件实现，而是源于分析哲学中关于"人类实践推理"（practical reasoning）的理论框架。哲学家 Michael Bratman 在 1987 年提出了意图（Intention）在理性行为中的核心作用——意图不仅是愿望的子集，更是一种"承诺"，它约束了行动者的后续推理和资源分配。

Anand Rao 和 Michael Georgeff 在 1991-1995 年间将这一哲学框架形式化为计算模型，使其能够在软件系统中实现。他们的工作发表在 ICMAS（International Conference on Multi-Agent Systems）等会议上，奠定了 BDI 在计算机科学领域的理论地位。

## 核心概念

BDI 模型的核心是三个互相关联的心智状态：

**信念（Beliefs）** 代表 Agent 对世界当前状态的认知。信念可能是不完整的、不确定的，甚至是错误的——它们反映的是 Agent 的"主观世界模型"而非客观真理。在形式化语义中，信念通常建模为一组可能世界（possible worlds）中 Agent 认为可达的子集。

**愿望（Desires）** 代表 Agent 希望达成的目标状态集合。愿望可以是相互矛盾的（例如同时希望节省成本和获得最高质量），也可以是暂时不可达的。愿望构成了 Agent 的动机空间，但并非所有愿望都会被付诸行动。

**意图（Intentions）** 是 Agent 经过审议后承诺要去实现的具体目标。意图具有以下关键属性：

- **持久性**：一旦形成意图，Agent 会持续努力直到成功、失败、或发现意图不再可行/可取
- **约束性**：意图会过滤掉与之冲突的其他选项，避免反复无目的地重新审议
- **层次性**：高层意图可以分解为子意图和具体计划

## BDI 推理循环

BDI Agent 的运行遵循一个感知-推理-行动的循环过程：

```
┌─────────────────────────────────────────────────┐
│                BDI 推理循环                        │
│                                                   │
│  感知环境 → 更新信念 → 生成选项 → 审议过滤 →      │
│  形成意图 → 规划执行 → 监控结果 → (循环)          │
└─────────────────────────────────────────────────┘
```

具体步骤：

1. **信念修正（Belief Revision）**：根据新的感知输入更新信念集合
2. **选项生成（Option Generation）**：基于当前信念和愿望，生成可能的行动方案
3. **审议（Deliberation）**：从候选方案中选择最优的作为意图——这是 BDI 区别于纯反应式系统的关键
4. **手段-目的推理（Means-End Reasoning）**：将抽象意图分解为具体的可执行计划
5. **执行与监控**：执行计划步骤，并监控前提条件是否仍然成立

## 经典实现平台

### PRS (Procedural Reasoning System)

由 SRI International 在 1980 年代末开发，是最早的 BDI 实现之一。PRS 被应用于 NASA 的故障诊断系统和澳大利亚航空管制系统中。

### JACK Agent Language

基于 Java 的 BDI Agent 开发平台，由 Agent Oriented Software 公司开发。JACK 提供了声明式的 BDI 编程范式，支持计划（Plans）、事件（Events）和信念集（BeliefSets）的直接建模。

### Jason

一个基于 AgentSpeak(L) 语言的开源 BDI 解释器。AgentSpeak 是 Rao 在 1996 年提出的 BDI 编程语言，Jason 为其提供了完整的运行时环境。Jason 通常与 JADE（Java Agent Development Environment）配合使用，构建分布式多智能体系统。

```
// AgentSpeak(L) 示例：一个简单的 BDI Agent
+!deliver(Package, Destination) :    // 触发计划的事件
    at(Package, Location) &          // 前提条件（信念）
    Location \== Destination         // 条件检查
    <-
    pick_up(Package);                // 行动
    move_to(Destination);            // 行动
    put_down(Package).               // 行动
```

### JADEX

在 JADE 平台之上实现的 BDI 推理引擎，使用 XML 定义 Agent 的信念、目标和计划，Java 实现具体行为逻辑。JADEX 在工业应用中有较多采用。

## 与反应式架构的对比

BDI 属于"慎思式"（deliberative）架构，与 Brooks 的包容式架构（Subsumption Architecture）等反应式（reactive）方法形成对比：

| 维度 | BDI（慎思式） | 反应式 |
|------|--------------|--------|
| 决策方式 | 基于内部模型推理 | 直接感知-行动映射 |
| 响应速度 | 较慢（需要推理） | 极快（固定映射） |
| 环境适应 | 强（可更新信念） | 弱（固定规则） |
| 可解释性 | 高（可追溯推理链） | 低 |
| 适用场景 | 复杂目标导向任务 | 简单实时响应任务 |

实践中，InteRRaP（1993）等混合架构将两者结合：底层使用反应式行为处理紧急事件，上层使用 BDI 推理进行战略规划。

## 在现代 LLM Agent 中的复兴

BDI 模型在 2024-2026 年的 LLM Agent 浪潮中经历了一次理论复兴。虽然现代 Agent 很少直接使用 AgentSpeak 或 JACK，但 BDI 的核心理念被重新诠释并融入了新的架构设计中：

**作为顶层推理框架：** 我们可以将 Agent 的推理过程显式地分解为 BDI 循环——根据信念（当前上下文和知识库）和愿望（用户目标），让 LLM 生成多个候选方案，再通过审议过程（可以是另一个 LLM 调用或规则引擎）选择最终意图，最后调用工具执行。这个过程的每一步都是可记录、可审计的。

**意图持久性解决"遗忘"问题：** LLM 在长对话中容易偏离主题或忘记目标。BDI 的意图持久性原则提供了解决思路——将顶层意图显式存储在状态中，每轮推理前检查意图是否仍然有效，避免 Agent 在执行长期任务时迷失方向。

**多 Agent 协作中的承诺协议：** 在多 Agent 系统中，意图可以作为 Agent 之间的"承诺"机制——当 Agent A 向 Agent B 承诺完成某个子任务时，这个承诺就构成了一个意图，A 会持续努力直到完成或显式撤回。

**融合设计示例：**

```python
class BDIAgent:
    """将 BDI 理论融入 LLM Agent 的概念示例"""
    
    def __init__(self, llm, tools):
        self.beliefs = KnowledgeBase()      # 信念：知识库 + 上下文
        self.desires = GoalStack()          # 愿望：目标栈
        self.intentions = IntentionQueue()  # 意图：已承诺的计划队列
        self.llm = llm
        self.tools = tools
    
    def deliberation_cycle(self, perception):
        # 1. 信念修正
        self.beliefs.update(perception)
        
        # 2. 选项生成（LLM 生成候选方案）
        options = self.llm.generate_options(
            beliefs=self.beliefs.summary(),
            desires=self.desires.active(),
            constraints=self.intentions.commitments()
        )
        
        # 3. 审议与过滤（选择最优方案作为意图）
        selected = self.llm.deliberate(
            options=options,
            criteria="feasibility, utility, commitment_consistency"
        )
        
        # 4. 形成意图并执行
        if selected and not self.intentions.conflicts_with(selected):
            self.intentions.commit(selected)
            return self.execute_intention(selected)
        
        # 5. 意图持久性检查
        self.intentions.review_and_prune(self.beliefs)
```

## 历史影响与评价

BDI 模型的贡献在于提供了一个优雅的中间抽象层——它比纯符号逻辑更贴近实际系统的需求，又比纯反应式系统提供了更强的推理能力和可解释性。其"承诺"和"审议"的概念，至今仍是理解和设计复杂 Agent 行为的重要理论工具。

然而，经典 BDI 也有其局限：形式化语义过于抽象，难以直接指导工程实现；信念修正和选项生成的具体算法留给了实现者；在高度动态和不确定的环境中，何时应该放弃一个意图的判断标准并不清晰。这些问题在 LLM 时代获得了新的解决路径——LLM 的灵活性正好弥补了经典 BDI 在知识表示和选项生成方面的不足。

## 延伸阅读

- Bratman, M.E. (1987). *Intention, Plans, and Practical Reason*. Harvard University Press
- Rao, A.S. & Georgeff, M.P. (1995). "BDI Agents: From Theory to Practice". ICMAS-95
- Bordini, R.H., Hübner, J.F., & Wooldridge, M. (2007). *Programming Multi-Agent Systems in AgentSpeak using Jason*
- Wooldridge, M. (2000). "Reasoning about Rational Agents". MIT Press
