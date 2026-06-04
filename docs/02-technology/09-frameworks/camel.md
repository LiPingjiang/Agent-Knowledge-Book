---
title: "CAMEL：通信式 Agent 探索框架"
description: "介绍 CAMEL 框架的角色扮演范式及其在多 Agent 通信研究中的学术贡献"
tags: ["camel", "multi-agent", "role-playing", "research", "communication"]
date: 2025-06-01
author: "Agent Knowledge Book"
---

# CAMEL：通信式 Agent 探索框架

CAMEL（Communicative Agents for "Mind" Exploration of Large Language Model Society）是 2023 年 3 月由 Li et al. 发表的研究工作，它探索了一个根本性问题：如何让 AI Agent 之间进行有效的自主通信来完成复杂任务？通过"角色扮演"（role-playing）范式，CAMEL 为多 Agent 通信研究开辟了新方向，其影响力远超框架本身——后来的 AutoGen、CrewAI 等框架都能追溯到 CAMEL 提出的核心思想。

## 核心思想：Inception Prompting

CAMEL 的核心创新是 Inception Prompting（启示提示）——通过精心设计的提示让两个 Agent 分别扮演"指导者"（AI Instructor）和"执行者"（AI Assistant），在几乎无人干预的情况下自主完成任务。

这种设计解决了一个关键困境：如果 Agent 完全自由对话，往往会偏离任务或陷入无意义的循环；如果给予太多约束，又失去了自主协作的价值。Inception Prompting 通过角色设定提供了恰当程度的"引导力"——既不过度约束，又不放任自流。

角色设定的核心要素包括：明确的专业身份（如"计算机视觉专家"）、清晰的任务目标、具体的行为约束（如"每次只给一个指令"）、以及终止条件的定义。

```python
# CAMEL 角色扮演的基本结构（概念示例）
from camel.agents import ChatAgent
from camel.messages import BaseMessage
from camel.types import RoleType

# 定义任务和角色
task_prompt = "开发一个基于 TF-IDF 的文本分类器"

# 指导者 Agent：负责分解任务、给出清晰指令
instructor_sys_msg = f"""你是一位自然语言处理专家。
你正在指导一位 Python 程序员完成以下任务：{task_prompt}。
规则：
1. 每次只给出一个明确的步骤指令
2. 指令必须具体、可执行
3. 在助手完成当前步骤前，不要给出下一步
4. 当任务完全完成时，回复"<CAMEL_TASK_DONE>"
"""

# 执行者 Agent：负责按指令实施
assistant_sys_msg = f"""你是一位经验丰富的 Python 程序员。
你正在接受 NLP 专家的指导完成任务：{task_prompt}。
规则：
1. 严格按照指令执行，不要自行扩展范围
2. 完成每步后报告结果
3. 遇到问题时向指导者请教
"""

instructor = ChatAgent(
    system_message=instructor_sys_msg,
    role_type=RoleType.USER
)

assistant = ChatAgent(
    system_message=assistant_sys_msg,
    role_type=RoleType.ASSISTANT
)

# 自主对话循环
def run_role_playing(max_turns: int = 15):
    """执行角色扮演对话"""
    input_msg = "让我们开始。请告诉我第一步该做什么。"
    
    conversation_log = []
    for turn in range(max_turns):
        # 指导者给出指令
        instructor_response = instructor.step(input_msg)
        conversation_log.append(("指导者", instructor_response.content))
        
        # 检查是否完成
        if "<CAMEL_TASK_DONE>" in instructor_response.content:
            break
        
        # 执行者响应
        assistant_response = assistant.step(instructor_response.content)
        conversation_log.append(("执行者", assistant_response.content))
        
        # 执行者的响应作为下一轮指导者的输入
        input_msg = assistant_response.content
    
    return conversation_log
```

## 研究贡献

### 多 Agent 通信模式的系统性研究

CAMEL 的主要贡献不在于工具本身的实用性，而在于它为研究社区提供了一个系统性框架来研究 Agent 间的通信行为。通过大规模实验（覆盖数十个领域、数百个任务），CAMEL 揭示了 LLM Agent 通信中的若干重要模式：

角色一致性（Role Consistency）：Agent 在对话中维持角色身份的能力会随轮次增加而退化。长对话中"角色漂移"是一个普遍现象。

任务分解策略：不同角色设定下，Agent 展现出截然不同的任务分解风格——有的倾向于先整体规划后逐步执行，有的则采用增量式探索。

指令遵循的边界：当指令与 Agent 内在"偏好"冲突时，指令遵循度会下降。这对后续 Agent 安全研究有重要启示。

### 大规模对话数据生成

CAMEL 框架可以自动生成大量 Agent 对话数据，这些数据本身就是有价值的研究资源。应用包括：训练更好的指令遵循模型、研究多轮交互中的失败模式、评估不同模型在协作任务中的表现差异、构建 Agent 行为的基准数据集。

### 多种社会结构模拟

CAMEL 后续扩展支持了多种 Agent 社会结构的模拟：

**层级结构**（Hierarchical）：一个管理者指挥多个执行者。

**平等协作**（Flat）：多个 Agent 平等讨论达成共识。

**辩论模式**（Debate）：Agent 持不同立场进行辩论，最终综合观点。

**竞争模式**（Competition）：Agent 竞争产出最优方案。

这些模式为研究"什么组织结构下多 Agent 协作最高效"提供了实验框架。

## OmniAgent 扩展

在 CAMEL 基础上，团队后续发展了更通用的任务求解能力——OmniAgent。它集成了代码执行、网络搜索、数学推理等多种工具，使 CAMEL 从纯对话研究工具扩展为可解决实际问题的多 Agent 系统。

OmniAgent 保留了角色扮演的核心机制，但增加了与外部环境交互的能力——Agent 不再仅仅"讨论"任务，还能实际"执行"任务。

## 框架能力概览

CAMEL 作为开源框架（不仅是论文），当前提供以下核心能力：

角色扮演对话生成——给定任何领域和任务描述，自动生成两个或多个 Agent 的完整协作对话过程。

多种 Agent 交互模式——支持一对一、一对多、多对多等不同拓扑的交互结构。

可扩展的工具集成——支持搜索引擎、代码解释器、知识库、数学计算等工具的集成。

模型无关设计——支持 OpenAI、Anthropic、开源模型等多种后端。

数据集生成能力——可批量生成指定领域的对话数据用于研究和训练。

## 学术工具 vs 生产框架的光谱

CAMEL 在"学术工具"到"生产框架"的光谱上明确偏向学术端。与 LangGraph、CrewAI 等面向生产的框架相比，其定位差异体现在几个方面：

CAMEL 优化的是"可研究性"——方便研究者设计实验、收集数据、分析 Agent 行为模式、发表论文。生产框架优化的是"可部署性"——错误处理、性能优化、可观测性、扩展性、运维友好。

这并不意味着 CAMEL 没有实用价值。它在以下场景仍有独特优势：需要快速生成大量 Agent 对话数据（用于模型训练或评估）；研究特定领域的 Agent 协作模式；探索新的多 Agent 交互机制的可行性验证；教学场景中演示多 Agent 系统的基本原理。

## 对行业的深远影响

CAMEL 的 inception prompting 思想深刻影响了后续的多 Agent 框架设计：

AutoGen 的 ConversableAgent 直接继承了"Agent 通过对话协作"的理念，只是在工程层面做了大量增强。

CrewAI 的角色定义（role, goal, backstory）可以看作是 CAMEL 角色设定思想的产品化简化版本。

MetaGPT 的角色分工（ProductManager, Architect, Engineer）受到了 CAMEL "用角色引导 Agent 行为"的启发。

甚至 OpenAI Agents SDK 的 instructions 设计，也体现了"通过清晰的角色定义来引导 Agent 行为"这一 CAMEL 首先系统性验证的思路。

## 本章小结

CAMEL 作为多 Agent 通信研究的先驱，其价值不在于工程层面的生产就绪性，而在于思想层面的开创性。它证明了 LLM Agent 可以通过角色扮演实现有意义的自主协作，并为这一研究方向提供了系统性的实验框架和大量数据。对于希望深入理解多 Agent 系统设计原理的开发者，CAMEL 的论文和代码都是值得研读的第一手材料。它提醒我们：优秀的框架设计往往始于对"Agent 应该如何交互"这一根本问题的深入思考。

## 延伸阅读

- [CAMEL 论文](https://arxiv.org/abs/2303.17760)
- [CAMEL GitHub](https://github.com/camel-ai/camel)
- [CAMEL 官方文档](https://docs.camel-ai.org/)
- [CAMEL Cookbook](https://github.com/camel-ai/camel/tree/master/examples)
- [AutoGen 的对话模式](./autogen.md) — CAMEL 思想的工程化延伸
- [多 Agent 系统综述](./classification.md) — 框架分类全景
