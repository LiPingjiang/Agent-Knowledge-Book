<!-- last updated: 2025-06 -->

# Agent 与传统 Pipeline 的本质区别

> 传统 Pipeline 将控制流硬编码在代码中，每一步确定且可预测；Agent 则将控制权交给模型，由 LLM 在运行时动态决策下一步行动。理解这一根本差异，是从软件工程师转型为 Agent 工程师的认知起点。

---

## 1. 传统软件 Pipeline 回顾

作为软件工程师，我们对 Pipeline 模式再熟悉不过了。无论是 ETL Pipeline（Extract-Transform-Load）、CI/CD Pipeline（Continuous Integration / Continuous Delivery）、还是数据处理 Pipeline（如 Apache Beam、Spark），它们共享一组核心特征：

**确定性执行**：给定相同输入，Pipeline 永远产出相同结果。一条 Jenkins Pipeline 定义了从代码检出、编译、测试到部署的每一步，开发者能精确预测执行路径。

**预定义步骤与拓扑结构**：Pipeline 的步骤在编写时就已确定。执行拓扑要么是线性（step1 → step2 → step3），要么是 DAG（有向无环图，允许并行和汇聚），但无论哪种形式，拓扑在运行前就已固定。

**中间件模式**：每个节点只关心自己的输入和输出，通过标准化接口串联。这是 Unix 哲学"做好一件事"在系统架构中的体现——管道和过滤器（Pipes and Filters）模式。

**Pipeline 的工程优势不可忽视**：可预测性意味着可测试，每个节点可以独立做单元测试；可监控意味着可观测，每一步的延迟、错误率、吞吐量都能被精确度量；成本可控意味着资源消耗在部署前就能估算；可回溯意味着出了问题可以精确定位到哪个节点、哪一行代码。

这些优势让 Pipeline 成为了工程化生产系统的基石。理解这些优势的价值，才能理解 Agent 引入了哪些新的工程挑战。

---

## 2. LLM Pipeline（非 Agent）

随着 LLM 的普及，很多系统在 Pipeline 中引入了 LLM 节点，但它们本质上仍然是 Pipeline——因为**流程控制权仍在代码中，而非模型中**。这个区分至关重要。

### 2.1 Prompt Chain（提示链）

将一个复杂任务分解为多个固定步骤，每步调用一次 LLM，前一步的输出作为下一步的输入：

```
文档 → [LLM: 提取摘要] → [LLM: 翻译成英文] → [LLM: 生成标题] → 结果
```

步骤是固定的，顺序是确定的，LLM 只是每个节点的"处理器"，它不能决定"我是否需要下一步"或"我应该跳回上一步"。Anthropic 将其称为 Prompt Chaining Workflow [Anthropic, 2024]。

### 2.2 RAG Pipeline（检索增强生成）

典型的 RAG 系统遵循固定的三步流程：

```
用户查询 → [Embedding + 向量检索] → [拼接上下文] → [LLM 生成回答] → 返回
```

每一步做什么是预先写死的。检索多少文档（top-k）、如何排序、如何拼接 Prompt，都是代码决定的。LLM 不能说"这些检索结果不够好，让我换个查询再搜一次"——除非你显式地在代码中编写了这个重试逻辑。

### 2.3 分类 + 处理 Pipeline（路由模式）

用 LLM 做意图分类，然后根据分类结果走不同的代码分支：

```
用户输入 → [LLM: 分类意图] → switch(intent) {
    "退款" → 退款流程(固定步骤)
    "咨询" → FAQ检索流程(固定步骤)  
    "投诉" → 人工转接流程(固定步骤)
}
```

虽然 LLM 做了"决策"（分类），但它只做了一次决策，而且决策的选项是代码预定义的枚举值。后续的流程完全由代码控制。Anthropic 将此称为 Routing Workflow [Anthropic, 2024]。

### 2.4 为什么这些不是 Agent

这些系统使用了 LLM，但**不是 Agent**。判断标准很简单：**谁在控制流程的走向？**

如果答案是"代码中的 if-else、for-loop、Pipeline 定义"——那就是 LLM Pipeline。如果答案是"模型在运行时自己决定下一步做什么"——那才是 Agent。

这个区分来自 Anthropic 的架构定义：Workflow 是"LLMs and tools are orchestrated through predefined code paths"，而 Agent 是"LLMs dynamically direct their own processes and tool usage, maintaining control over how they accomplish tasks" [Anthropic, 2024]。

---

## 3. Agent 的本质差异

从 Pipeline 到 Agent，不是增量的变化，而是范式的转移。这个转移的核心在于以下几个维度：

### 3.1 控制权转移：从代码到模型

这是最根本的区别。在 Pipeline 中，开发者编写代码来决定"下一步做什么"。在 Agent 中，模型在运行时自己决定"下一步做什么"。

```python
# Pipeline: 控制权在代码
def pipeline(input):
    step1_result = llm.call(prompt_1, input)
    step2_result = llm.call(prompt_2, step1_result)
    step3_result = llm.call(prompt_3, step2_result)
    return step3_result

# Agent: 控制权在模型
def agent_loop(goal):
    while not is_goal_achieved():
        next_action = llm.decide(goal, history, available_tools)
        result = execute(next_action)
        history.append(result)
    return history
```

这个看似简单的差异，引发了一系列深层的工程后果。

### 3.2 非确定性执行路径

Pipeline 对同一输入始终走相同路径。Agent 则不然——同一个任务描述，模型可能选择不同的工具、不同的执行顺序、不同的中间步骤。这不是 bug，而是 Agent 的本质特性。

就像两个程序员接到同一个需求，可能写出结构完全不同但都正确的代码一样。Agent 的"正确性"不是路径正确，而是结果正确。

### 3.3 开放式行动空间

Pipeline 的每一步只有一个可能的操作（调用下一个节点）。Agent 在每个时刻面对的是一个**开放的行动空间**：它可以调用任何可用工具、可以决定不调用工具而是直接回复、可以要求更多信息、可以回退并重试。

OpenAI 在其实践指南中指出，Agent 的核心能力是"利用大型语言模型来管理工作流程的执行和决策。它能够识别工作流程何时完成，并在需要时主动纠正其行为" [OpenAI, 2025]。

### 3.4 自适应循环：基于环境反馈调整行为

Agent 的核心运行机制是一个循环（loop），而非一条链（chain）：

```
观察(Observe) → 思考(Think) → 行动(Act) → 观察新状态 → ...
```

关键在于"观察新状态"——Agent 会根据上一步行动的实际结果来调整下一步行动。如果一个工具调用失败了，Agent 可以尝试不同的方法；如果检索结果不够好，Agent 可以重新组织查询。这种**闭环反馈**能力是 Pipeline 不具备的。

Andrew Ng 将此总结为 Agentic AI 的四大设计模式之一——Reflection（反思），即让 LLM 审视自己的输出并进行改进 [Ng, 2024]。

### 3.5 终止条件的模糊性

Pipeline 的终止条件是确定的：所有步骤执行完毕，即终止。Agent 的终止条件本质上是模糊的：不是"步骤执行完了"，而是"目标是否达成了"。这需要模型自己判断。

这种模糊性意味着 Agent 可能过早终止（认为完成了但其实没有），也可能过度执行（陷入无限循环）。OpenAI 建议通过最大轮次限制（maximum number of iterations）作为安全网 [OpenAI, 2025]，Anthropic 同样推荐设置 stopping conditions 来维持控制 [Anthropic, 2024]。

---

## 4. 从工程视角做对比

以下从十个工程维度系统对比 Pipeline 与 Agent 的差异：

| 对比维度 | 传统 Pipeline | Agent |
|---------|--------------|-------|
| **控制流** | 硬编码在代码中（if/else、DAG 定义） | 模型在运行时动态决策 |
| **可预测性** | 确定性：同一输入 → 同一路径 → 同一输出 | 概率性：同一输入可能走不同路径、产出不同结果 |
| **错误处理** | try-catch + 重试策略 + 死信队列 | 自主恢复：模型识别错误并尝试替代方案 |
| **测试方法** | 单元测试 + 集成测试（assert 精确值） | 行为评估（eval）：评估结果质量而非精确匹配 |
| **调试方式** | Stack trace + 日志 + 断点 | Trace（完整对话/工具调用记录）+ Replay |
| **成本模型** | 固定且可预测（与输入量线性相关） | 变动且不可预测（与任务复杂度、循环次数相关） |
| **延迟特征** | 可预测（各步骤延迟之和） | 不可预测（取决于循环次数和工具调用链） |
| **状态管理** | 外部存储（DB、消息队列、文件系统） | 内部上下文（对话历史、记忆）+ 外部工具 |
| **扩展方式** | 加节点、加实例、水平扩容 | 加工具、优化 Prompt、增强模型能力 |
| **适用场景** | 结构化、高吞吐、要求确定性的任务 | 开放式、需要推理判断、路径不可预定义的任务 |

### 逐维度深入解读

**控制流**：Pipeline 的控制流在部署前就确定了——CI/CD Pipeline 不会在运行时突然决定"跳过测试直接部署"。Agent 的控制流在运行时动态生成——coding agent 可能决定先读文件、再搜索、再修改、再运行测试，这个序列不是预先编排的。

**可预测性**：Pipeline 的确定性是我们能做 SLA 承诺的基础。Agent 的概率性意味着我们只能承诺"大概率完成"而不是"确定完成"。这需要全新的质量保证体系。

**错误处理**：Pipeline 遇到错误只能走预定义的错误处理路径。Agent 遇到错误可以"想想为什么失败了"并尝试完全不同的方法——比如一个 API 调用失败了，Agent 可能会尝试从另一个数据源获取相同信息。

**测试方法**：这可能是最大的工程文化冲击。Pipeline 可以写 `assert result == expected_value`，Agent 只能写 `assert quality_score(result) > threshold`。从精确匹配到质量评估，测试的理念和方法论都需要重建。

**成本模型**：Pipeline 处理一条消息的成本是固定的（比如 3 次 LLM 调用）。Agent 处理一个任务可能需要 3 次循环，也可能需要 30 次循环。这使得成本预算变得极具挑战性。Anthropic 明确指出"The autonomous nature of agents means higher costs, and the potential for compounding errors" [Anthropic, 2024]。

---

## 5. 什么时候用 Pipeline，什么时候用 Agent

### 5.1 Pipeline 更优的场景

**高吞吐量、低延迟的在线服务**：电商平台的商品推荐 Pipeline 每秒处理数万请求，需要毫秒级响应。用 Agent 逐个"思考"每个推荐请求既不经济也不必要。

**流程固定且合规性要求高的场景**：金融交易结算、医疗数据处理等场景，监管要求每一步可审计、可回溯、可重现。Pipeline 的确定性恰好满足这些要求。

**输入输出格式高度结构化**：ETL 任务将 CSV 转为 Parquet、数据清洗去重、格式标准化——这些任务的逻辑是确定的，用 Agent 只会增加成本和不确定性。

**已经有明确最优解的任务**：如果你已经知道最佳处理流程，用 Pipeline 把它编码下来就行。Agent 的价值在于"探索未知路径"，对已知最优路径没有附加价值。

### 5.2 Agent 更优的场景

**开放式问题求解**：软件工程中的 bug 修复——你不知道 bug 在哪个文件、需要改几行代码、是否需要改测试。SWE-bench 上的 coding agent 正是处理这类任务 [Anthropic, 2024]。

**需要多轮推理和判断的任务**：复杂的客户服务场景——需要理解用户意图、查询多个系统、根据政策做判断、在多种选项中选择最优方案。

**路径不可预定义的任务**：研究型任务——"帮我调研竞品 X 的技术架构"，你不知道需要搜索几次、需要读哪些文档、需要从哪些角度分析。

**需要适应性和自纠错的场景**：数据分析——"这个数据异常的原因是什么？"可能需要尝试多种假设、查看不同维度的数据、排除干扰因素，直到找到合理解释。

### 5.3 混合模式

实践中最常见的不是非此即彼，而是混合使用：

**Pipeline 中嵌入 Agent 节点**：一个数据处理 Pipeline 的大部分步骤是确定性的（读取、清洗、转换），但其中一步"数据质量异常处理"用 Agent 来做，因为异常的种类不可穷举。

**Agent 内部调用 Pipeline**：一个 coding agent 在决定"需要重构这个函数"后，调用一个固定的 Pipeline 来执行（格式化代码 → 运行测试 → 检查覆盖率），因为这部分流程是确定性的。

### 5.4 Anthropic 的核心建议

Anthropic 在其指南中反复强调："When building applications with LLMs, we recommend finding the simplest solution possible, and only increasing complexity when needed" [Anthropic, 2024]。

翻译成工程决策就是：从 Pipeline 开始，只有当 Pipeline 无法胜任时，才引入 Agent 的复杂性。这不是保守，而是工程智慧——每一层抽象都有成本，Agent 的成本尤其高。

---

## 6. 从 Pipeline 到 Agent 的渐进式演进路径

从工程实践角度，系统从纯 Pipeline 到完全自主 Agent 的演进通常经历以下四个阶段：

### 阶段一：固定 Pipeline + LLM 节点

在现有 Pipeline 中用 LLM 替代某些规则节点。流程固定，LLM 只是"更智能的函数"。

```python
# 阶段一：LLM 作为 Pipeline 中的处理节点
def process_ticket(ticket):
    # Step 1: LLM 做分类（替代规则引擎）
    category = llm.classify(ticket.text, categories=["bug", "feature", "question"])
    # Step 2: 固定逻辑
    if category == "bug":
        priority = llm.assess_priority(ticket.text)
        assign_to_team(ticket, "engineering", priority)
    elif category == "feature":
        add_to_backlog(ticket)
    else:
        auto_reply(ticket, search_faq(ticket.text))
```

**特征**：控制流完全在代码中，LLM 调用次数固定，延迟可预测。

### 阶段二：引入路由 / 条件分支（LLM 做决策点）

LLM 不仅处理数据，还决定流程的分支走向。但分支选项仍是预定义的有限集合。

```python
# 阶段二：LLM 做路由决策
def handle_request(request):
    # LLM 决定走哪条路径（但路径本身是预定义的）
    route = llm.decide_route(
        request, 
        options=["simple_answer", "deep_research", "escalate_to_human"]
    )
    if route == "simple_answer":
        return llm.generate_answer(request, context=basic_context)
    elif route == "deep_research":
        docs = search_engine.query(request)
        analysis = llm.analyze(docs)
        return llm.generate_answer(request, context=analysis)
    elif route == "escalate_to_human":
        return create_human_ticket(request)
```

**特征**：LLM 影响控制流，但可选路径是有限的。这是 Anthropic 所说的 Routing Workflow。

### 阶段三：引入循环和自纠错

引入循环结构，允许 LLM 对自己的输出进行评估并迭代改进。这是向 Agent 迈出的关键一步。

```python
# 阶段三：引入循环 + 自评估
def research_with_reflection(question, max_iterations=3):
    answer = ""
    for i in range(max_iterations):
        # 生成/改进答案
        answer = llm.generate(question, previous_answer=answer, sources=search(question))
        # 自评估
        evaluation = llm.evaluate(
            question=question, 
            answer=answer,
            criteria=["completeness", "accuracy", "relevance"]
        )
        if evaluation.score > 0.85:
            break  # 质量足够，终止循环
        # 根据评估反馈调整搜索策略
        question = llm.refine_query(question, evaluation.feedback)
    return answer
```

**特征**：有循环，LLM 能自我纠正，但循环次数有上限，退出条件由代码定义。这对应 Anthropic 的 Evaluator-Optimizer Workflow 和 Andrew Ng 的 Reflection 模式。

### 阶段四：开放式 Agent Loop

完全开放的 Agent 循环：模型自主选择工具、自主决定下一步、自主判断何时终止。

```python
# 阶段四：完整 Agent Loop
def agent(goal, tools, max_steps=20):
    history = []
    for step in range(max_steps):
        # 模型自主决策：做什么、用什么工具、还是结束
        decision = llm.plan_next_action(
            goal=goal,
            history=history,
            available_tools=tools
        )
        if decision.action == "finish":
            return decision.final_answer
        # 执行模型选择的行动
        result = execute_tool(decision.tool, decision.arguments)
        history.append({"action": decision, "result": result})
    return "达到最大步数限制，任务未完成"
```

**特征**：模型拥有完全的控制权——选择哪个工具、用什么参数、是否继续、何时结束。这就是 Anthropic 定义的 Agent。

### 演进路径总结

每个阶段都不是替代前一阶段，而是在前一阶段的基础上增加灵活性。系统中可以同时存在处于不同阶段的组件——这才是工程实践中的常态。

---

## 7. Agent 引入的新工程挑战

Agent 的灵活性不是免费的。它带来了一系列传统软件工程从未面对过的挑战：

### 7.1 不可重现性

同一输入不保证相同输出。这直接冲击了软件工程最基本的假设之一——可重现性。传统的回归测试（"这个输入应该产出这个结果"）在 Agent 系统中失效了。

**对策**：转向基于评估（Evaluation）的质量保证体系。不检查"输出是否与预期精确匹配"，而检查"输出是否满足质量标准"。建立 eval benchmark，用统计方法评估系统表现。（详见后续「Agent 评测与可观测性」章节）

### 7.2 成本不可控

Agent 的执行成本取决于任务复杂度和模型的决策路径，两者都不可预测。一个简单的任务可能 2 次循环就完成，一个复杂任务可能用掉 50 次循环。每次循环都消耗 token，成本可能差一个数量级。

**对策**：设置最大循环次数（max iterations）、token 预算上限、以及基于启发式的早停策略。对成本敏感的场景，考虑用 Pipeline 覆盖常见路径（80%），只对长尾情况启用 Agent。

### 7.3 安全边界

Pipeline 只能执行代码中预定义的操作。Agent 拥有工具调用能力，理论上可以执行任何已授权的操作——包括删除数据、发送消息、修改配置。如果模型的判断出错，后果可能比 Pipeline 严重得多。

**对策**：实施最小权限原则（least privilege）、关键操作需要人工确认（human-in-the-loop）、在沙箱环境中运行 Agent、对工具调用设置速率限制和白名单。OpenAI 将此称为"防护措施"（Guardrails）[OpenAI, 2025]。

### 7.4 评测困难

传统软件的正确性可以用 assert 语句验证。Agent 的"正确性"是多维度的：结果正确吗？过程合理吗？效率可接受吗？有没有副作用？这些维度往往需要人类判断，难以完全自动化。

**对策**：构建多层评测体系——单工具调用级别的精确测试 + 端到端任务级别的质量评估 + 人工抽样审查。使用 LLM-as-Judge 做自动化评估，但保留人工验证作为最终标准。（详见后续「Agent 评测与可观测性」章节）

### 7.5 调试与可观测性

Pipeline 出了问题，查 Stack trace 和日志即可定位。Agent 出了问题，你需要阅读完整的对话历史和工具调用序列，理解模型"为什么做了那个决定"。这更像是在做行为分析而非代码调试。

**对策**：建立完善的 Trace 系统，记录每一步的输入、输出、模型推理过程。支持 Replay（回放）能力，能够在相同上下文下重新运行某一步。Anthropic 强调"Prioritize transparency by explicitly showing the agent's planning steps" [Anthropic, 2024]。

---

## 参考文献

- [Anthropic, 2024] Building Effective Agents. https://www.anthropic.com/research/building-effective-agents
- [OpenAI, 2025] A Practical Guide to Building Agents. https://cdn.openai.com/business-guides-and-resources/a-practical-guide-to-building-agents.pdf
- [Andrew Ng, 2024] Agentic Design Patterns. Sequoia AI Ascent Talk, March 2024. https://www.deeplearning.ai/the-batch/how-agents-can-improve-llm-performance/
- [Anthropic, 2024] Building Agents with the Claude Agent SDK. https://platform.claude.com/docs/en/agent-sdk/overview
- [OpenAI, 2025] Agent Orchestration - OpenAI Agents SDK. https://openai.github.io/openai-agents-python/multi_agent/
