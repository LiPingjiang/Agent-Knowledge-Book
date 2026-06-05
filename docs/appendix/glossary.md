<!-- last updated: 2025-06 -->
# 术语表（Glossary）

Agent 领域核心术语的中英对照与释义。本附录作为全书的术语索引，文中以超链接形式引用此处词条。

---

## 一、计算复杂度与算法

### PSPACE-complete {#pspace-complete}

**PSPACE-complete**（多项式空间完全）是计算复杂度理论中的一个重要类别。PSPACE 是指所有可以用多项式空间（内存）求解的决策问题集合——不限制计算时间，但使用的存储空间必须是输入规模的多项式函数。一个问题是 PSPACE-complete 的，意味着它"至少和 PSPACE 中最难的问题一样难"：(1) 它本身属于 PSPACE；(2) PSPACE 中的任何问题都可以在多项式时间内归约为它。直观理解：PSPACE-complete 问题比 NP-complete 更难（或至少不比它简单），因为 NP ⊆ PSPACE。STRIPS 规划被证明是 PSPACE-complete 意味着：在最坏情况下，求解一般性规划问题所需的计算资源可能超乎想象地大，没有已知的高效通用算法。

### NP-hard {#np-hard}

**NP-hard**（非确定性多项式时间困难）指一类问题，至少和 NP（Non-deterministic Polynomial time）中最难的问题一样难。NP 是"给定一个候选解，可以在多项式时间内验证其正确性"的问题集合。NP-hard 问题不要求自身属于 NP（即不要求解可以快速验证），只要求 NP 中所有问题都能归约到它。典型例子：旅行商问题（TSP）的最优解、最优规划（找最短动作序列）。实际含义：目前没有已知的多项式时间算法可以解决 NP-hard 问题；当问题规模增长时，求解时间可能呈指数级增长。

### NP-complete {#np-complete}

**NP-complete**（NP 完全）是 NP 与 NP-hard 的交集——既属于 NP（解可以快速验证），又是 NP-hard（至少和 NP 中最难的一样难）。NP-complete 是 NP 中"最难"的那些问题。经典例子包括布尔可满足性问题（SAT）、图着色问题、子集和问题。P ≠ NP 猜想（计算机科学最大未解问题之一）等价于"NP-complete 问题不存在多项式时间算法"。

### 2-EXPTIME {#2-exptime}

**2-EXPTIME**（双指数时间）指需要 2^(2^n) 量级时间才能求解的问题类。比 EXPTIME（单指数 2^n）还要难得多。条件规划（Contingent Planning）在不确定性环境下被归入此类，意味着当环境具有不确定性时，规划的计算复杂度会极其恐怖——即使中等规模的问题也可能在宇宙寿命内无法求解。

### O(N) 与时间复杂度 {#big-o}

**大 O 表示法**（Big-O Notation）用于描述算法在最坏情况下的资源消耗增长率。O(N) 表示线性增长（数据量翻倍则时间翻倍），O(log N) 表示对数增长（极其高效，数据量翻倍仅增加常数时间），O(N²) 表示二次增长（数据量翻倍则时间增长 4 倍）。它忽略常数因子，关注增长趋势。

### BFS / DFS {#bfs-dfs}

**BFS**（Breadth-First Search，广度优先搜索）按"层"逐步向外扩展搜索，先探索所有距离为 1 的节点，再探索距离为 2 的，以此类推。保证找到最短路径但内存消耗大。**DFS**（Depth-First Search，深度优先搜索）沿一条路径尽可能深入，遇到死胡同再回溯。内存消耗小但不保证最短路径。两者是图搜索/树搜索的基本策略。

### A* 搜索 {#a-star}

**A\*** 是一种启发式搜索算法，结合了 BFS 的最优性和启发式函数的引导效率。评估函数 f(n) = g(n) + h(n)，其中 g(n) 是从起点到当前节点的实际代价，h(n) 是对当前节点到目标距离的启发式估计。如果 h(n) 满足"可采纳性"（永不高估），则 A* 保证找到最优解。

### MCTS {#mcts}

**MCTS**（Monte Carlo Tree Search，蒙特卡洛树搜索）是一种通过随机模拟来指导搜索决策的算法。四个阶段循环：选择（Selection）→ 扩展（Expansion）→ 模拟（Simulation/Rollout）→ 反向传播（Backpropagation）。AlphaGo 使用 MCTS + 深度神经网络，在围棋中击败人类冠军。在 Agent 规划中，MCTS 被用于在多个推理路径间进行结构化探索。

### 启发式函数 {#heuristic}

**启发式函数**（Heuristic Function）是对"从当前状态到目标还有多远"的近似估计。它不保证精确，但能大幅减少搜索空间。好的启发式函数既要"信息量大"（提供有效指导）又要"计算便宜"（快速求值）。例如在路径规划中，直线距离是一个经典的启发式。

### K-Means {#k-means}

**K-Means** 是最常用的无监督聚类算法。将 N 个数据点划分为 K 个簇，每个点归属于离它最近的簇中心（centroid）。算法反复迭代"分配点到最近中心 → 重新计算中心位置"直到收敛。在向量数据库中用于构建索引：先对向量空间做 K-Means 聚类，查询时只搜索最近的几个簇，从而加速检索。

### Voronoi cells {#voronoi}

**Voronoi 图**（Voronoi Diagram）将空间根据一组"种子点"划分为若干区域，每个区域包含距离某个特定种子点最近的所有点。在向量索引中，K-Means 的每个聚类中心定义一个 Voronoi cell，查询时先确定点落入哪个 cell，再在该 cell 内做精确搜索。

### ANN {#ann}

**ANN**（Approximate Nearest Neighbor，近似最近邻）是指在高维空间中快速找到与查询点"足够近"（但不一定最近）的点的算法族。精确最近邻在高维空间中计算成本极高（维数灾难），ANN 通过牺牲少量精度换取数量级的速度提升。常见实现包括 HNSW、IVF、LSH。

### HNSW {#hnsw}

**HNSW**（Hierarchical Navigable Small World，分层可导航小世界图）是当前最流行的 ANN 算法之一。核心思想来自"小世界网络"理论——在图中每个节点既有"近距离邻居"也有"远距离跳跃连接"。HNSW 构建多层图结构：顶层稀疏（长距离跳跃），底层密集（精细搜索）。查询从顶层开始快速定位大致区域，逐层下降做精细化搜索。

### IVF {#ivf}

**IVF**（Inverted File Index，倒排文件索引）是向量检索中的分区策略。先用 K-Means 将向量空间划分为 N 个区域（聚类），为每个区域建立一个"倒排列表"存储该区域内的所有向量。查询时先确定最可能的 M 个区域（M << N），只在这 M 个区域内搜索，大幅减少需要比较的向量数量。

### 拓扑排序 {#topological-sort}

**拓扑排序**（Topological Sort）是对有向无环图（DAG）的节点进行线性排序，使得对于每条有向边 u→v，u 在排序中出现在 v 之前。在任务调度中用于确定依赖任务的执行顺序——只有所有前置任务完成后，后续任务才能开始。

### 组合爆炸 {#combinatorial-explosion}

**组合爆炸**（Combinatorial Explosion）指当问题维度增加时，候选解的数量以指数级甚至更快的速度增长。例如 N 个布尔变量有 2^N 种组合，N 个城市的旅行路线有 N! 种排列。这是规划、搜索类问题的核心困难来源。

### MapReduce {#mapreduce}

**MapReduce** 是 Google 提出的分布式计算编程模型。分两阶段：Map 阶段将输入数据拆分为独立的小块并行处理；Reduce 阶段将 Map 的结果汇总聚合。在 Agent 模式中被借鉴为"并行化-再汇总"的任务分解模式。

### Fan-out {#fan-out}

**Fan-out**（扇出）是指一个操作触发多个并行子操作的模式。在分布式系统中，一个请求扇出为多个并行调用后再收集结果。在 Agent 模式中指将一个任务同时分派给多个子 Agent 或多个工具并行执行。

### Quorum 读 {#quorum}

**Quorum**（法定人数）是分布式系统中的一致性机制。Quorum 读要求从 N 个副本中获得至少 W 个一致响应才认为读取成功（通常 W > N/2）。在 Agent 投票/共识模式中借鉴为"多数意见"决策机制。

### Saga {#saga}

**Saga** 是分布式系统中管理长事务的模式。将一个大事务拆分为一系列局部事务（步骤），每步都有对应的补偿操作（回滚）。如果某步失败，按反序执行已完成步骤的补偿。在 Agent 编排中用于管理多步骤任务的错误恢复。

### ETL {#etl}

**ETL**（Extract-Transform-Load，提取-转换-加载）是数据工程的经典流程：从数据源抽取原始数据（Extract），进行清洗/转换/聚合（Transform），最后加载到目标存储（Load）。在 Agent 系统中，DAG 工作流常被类比为 ETL 管线。

### LRU {#lru}

**LRU**（Least Recently Used，最近最少使用）是一种缓存淘汰策略。当缓存满时，淘汰最长时间未被访问的条目。在 Agent 记忆系统中，用于管理有限的短期记忆容量——最久未使用的记忆片段被优先遗忘。

### Dead-letter queue {#dead-letter-queue}

**Dead-letter queue**（死信队列）是消息系统中用于存放无法被正常消费的消息的特殊队列。当一条消息反复处理失败（超过重试次数）或无法路由时，被转移到死信队列供人工排查。在事件驱动的 Agent 架构中用于处理无法执行的任务。

---

## 二、数学与统计

### Pareto optimal {#pareto-optimal}

**Pareto 最优**（帕累托最优）是多目标优化中的概念：一个解是 Pareto 最优的，当且仅当不存在另一个解能在某个目标上更好而不在其他目标上变差。在多 Agent 博弈中，Pareto 最优均衡指没有任何参与者能在不损害其他人的情况下获益。

### 贝叶斯网络 {#bayesian-network}

**贝叶斯网络**（Bayesian Network）是一种用有向无环图表示变量间条件依赖关系的概率图模型。节点代表随机变量，边代表因果或依赖关系，每个节点附带条件概率表。用于不确定性推理——给定部分观测，可以推断其他变量的后验概率。

### 指数衰减 {#exponential-decay}

**指数衰减**（Exponential Decay）描述一个量按固定比例持续减少的过程，数学表达为 f(t) = A·e^(-λt)。在 Agent 记忆中用于建模"时间遗忘"——越久远的记忆其权重越低。λ 是衰减率常数，控制遗忘速度。

### 余弦相似度 {#cosine-similarity}

**余弦相似度**（Cosine Similarity）衡量两个向量方向的相似程度，值域 [-1, 1]。计算公式为两向量点积除以各自模长的乘积：cos(θ) = (A·B)/(|A|·|B|)。在向量检索中广泛使用——两段文本的语义相似度通过其 Embedding 向量的余弦相似度来衡量。

### KL 散度 {#kl-divergence}

**KL 散度**（Kullback-Leibler Divergence）衡量两个概率分布之间的"距离"（严格说是非对称的信息损失度量）。D_KL(P||Q) 表示用分布 Q 来近似真实分布 P 时损失的信息量。在 RLHF 训练中用于约束策略模型不偏离原始预训练模型太远。

### Temperature {#temperature}

**Temperature**（采样温度）是 LLM 文本生成中的超参数，控制输出的随机性。数学上它缩放 logits 向量：softmax(logits/T)。T→0 时退化为贪心选择（总选概率最高的 token），T→∞ 时趋向均匀随机。通常 T=0.7 平衡创造性和连贯性。

### Logits {#logits}

**Logits** 是神经网络最后一层的原始输出值（未经 softmax 归一化）。在语言模型中，logits 是词汇表中每个 token 的"未归一化得分"。经过 softmax 转换后变成概率分布，用于采样下一个 token。

### Softmax {#softmax}

**Softmax** 函数将一组实数值转换为概率分布：softmax(xᵢ) = e^(xᵢ) / Σ e^(xⱼ)。所有输出值在 (0,1) 之间且和为 1。在语言模型中用于将 logits 转换为 token 概率。

### Scaling Laws {#scaling-laws}

**Scaling Laws**（缩放定律）是 Kaplan et al. (2020) 发现的经验规律：LLM 的性能（以 loss 衡量）与模型参数量、训练数据量、计算量之间存在幂律关系。这意味着持续增加资源可以可预测地提升模型能力，是 GPT-4 等大模型投资的理论基础。

### i.i.d. {#iid}

**i.i.d.**（independent and identically distributed，独立同分布）是统计学基本假设：数据集中每个样本来自同一分布且彼此独立。强化学习中经验回放（Experience Replay）的动机之一是打破训练数据的时序相关性，使其近似 i.i.d.，从而稳定梯度下降。

### BM25 {#bm25}

**BM25**（Best Matching 25）是信息检索中经典的文本相关性评分函数，基于词频（TF）和逆文档频率（IDF）。它比简单的 TF-IDF 更精细，加入了文档长度归一化和词频饱和度。在混合检索中与向量搜索互补——BM25 擅长精确关键词匹配，向量搜索擅长语义相似性。

---

## 三、AI / 机器学习核心概念

### Embedding {#embedding}

**Embedding**（嵌入）是将离散对象（词、句子、图像）映射为连续的固定维度向量的过程。语义相似的对象在向量空间中距离更近。文本 Embedding 模型（如 OpenAI text-embedding-3）将一段文字压缩为一个 1536 维向量，使得语义检索成为可能。

### Transformer {#transformer}

**Transformer**（Vaswani et al., 2017）是当前几乎所有大语言模型的基础架构。核心创新是**自注意力机制**（Self-Attention）——允许序列中每个位置直接关注其他所有位置，突破了 RNN 的顺序处理限制。由 Encoder-Decoder 两部分组成（GPT 系列只用 Decoder 部分）。

### Self-Attention {#self-attention}

**自注意力**（Self-Attention）是 Transformer 的核心操作。对于序列中的每个 token，计算它与所有其他 token 的相关性得分（通过 Q·K^T/√d），然后用这些得分对 Value 向量做加权求和。这允许模型捕捉任意距离的依赖关系。Multi-head attention 是并行运行多组独立的注意力计算。

### Attention Head {#attention-head}

**注意力头**（Attention Head）是多头注意力中的一个独立注意力计算单元。每个头有自己的 Q/K/V 投影矩阵，可以学习关注不同类型的关系（如语法关系、语义关系、位置关系）。多个头的输出拼接后经线性变换得到最终结果。

### MLP {#mlp}

**MLP**（Multi-Layer Perceptron，多层感知机）是最基本的前馈神经网络：由输入层、一个或多个隐藏层、输出层组成，相邻层全连接。在 Transformer 中每个注意力层后面都跟一个 MLP（也称 FFN），它负责对注意力层输出做非线性变换，存储和检索"知识"。

### Residual Stream {#residual-stream}

**残差流**（Residual Stream）是 Transformer 的信息传递主干。每一层（注意力 + MLP）的输出通过残差连接（x + layer(x)）累加回主流。可以理解为一条"信息高速公路"——每层只往上面"写入"增量信息，而非覆盖。这一视角对模型可解释性研究（Mechanistic Interpretability）至关重要。

### RLHF {#rlhf}

**RLHF**（Reinforcement Learning from Human Feedback，基于人类反馈的强化学习）是将 LLM 对齐到人类偏好的核心技术。流程：(1) 收集人类对模型输出的偏好排序；(2) 训练一个奖励模型（Reward Model）来预测人类偏好；(3) 用 PPO 等 RL 算法优化 LLM 使其最大化奖励模型的评分，同时用 KL 散度约束防止过度偏离。

### PPO {#ppo}

**PPO**（Proximal Policy Optimization，近端策略优化）是一种策略梯度强化学习算法（Schulman et al., 2017）。通过裁剪目标函数（clipped objective）限制每次策略更新的幅度，兼顾训练稳定性和样本效率。是 RLHF 中最常用的优化算法。

### GRPO {#grpo}

**GRPO**（Group Relative Policy Optimization，群组相对策略优化）是 DeepSeek 团队提出的 RL 训练算法。与 PPO 不同，它不需要单独的 Critic（价值网络），而是在一组采样回答中通过相对排序来估计优势函数，显著降低训练开销。用于训练 DeepSeek-R1 等推理模型。

### SFT {#sft}

**SFT**（Supervised Fine-Tuning，监督微调）是在预训练模型基础上，使用人工标注的（指令, 回答）配对数据进行有监督训练。它是 RLHF 流程的第一步——先 SFT 让模型学会"对话格式"，再用 RL 优化质量。也可独立使用，用于特定任务适配。

### DPO {#dpo}

**DPO**（Direct Preference Optimization，直接偏好优化）是 Rafailov et al. (2023) 提出的 RLHF 替代方案。它证明 RLHF 的目标可以转化为一个简单的分类损失，直接在偏好数据上训练，无需单独的奖励模型和 RL 循环。训练更简单稳定，效果与 PPO 相当。

### PRM / ORM {#prm-orm}

**PRM**（Process Reward Model，过程奖励模型）对推理的每一步打分，**ORM**（Outcome Reward Model，结果奖励模型）只对最终答案打分。PRM 能提供更细粒度的训练信号（"第 3 步推理出错了"），但标注成本更高。在数学推理任务中，PRM 被证明显著优于 ORM。

### LoRA {#lora}

**LoRA**（Low-Rank Adaptation）是一种参数高效的微调方法。核心思想：冻结原始模型权重，为每层注入低秩分解的增量矩阵（ΔW = BA，其中 B∈R^(d×r), A∈R^(r×d), r<<d）。训练参数量减少 10-1000 倍，推理时可将 ΔW 合并回原始权重，无额外延迟。

### Few-shot / Zero-shot {#few-zero-shot}

**Few-shot**（少样本）指在提示中提供少量示例让模型学习任务模式；**Zero-shot**（零样本）指不提供任何示例，仅靠指令描述让模型执行任务。这是 LLM 的核心能力——无需梯度更新即可通过上下文学习（In-Context Learning）适应新任务。

### In-Context Learning {#in-context-learning}

**上下文学习**（In-Context Learning, ICL）是大语言模型的涌现能力之一：将任务描述和示例放入提示（Prompt），模型在不更新参数的情况下"学会"新任务。机制仍有争议——可能是隐式的梯度下降（Akyürek et al., 2022），也可能是任务识别后检索预训练中见过的相似模式。

### Emergent Abilities {#emergent-abilities}

**涌现能力**（Emergent Abilities）指仅在模型规模超过某个阈值后才突然出现的能力——小模型完全不具备，大模型突然"获得"。经典例子：思维链推理在 ~100B 参数后涌现。Wei et al. (2022) 首次系统研究了这一现象（但后续也有研究质疑"涌现"可能是评估指标的假象）。

### Alignment {#alignment}

**对齐**（Alignment）指让 AI 系统的行为符合人类意图和价值观的技术与理念。包括"有用性"（Helpfulness）、"无害性"（Harmlessness）、"诚实性"（Honesty）三个维度。技术手段包括 RLHF、Constitutional AI、DPO 等。是当前 AI 安全的核心议题。

### Hallucination {#hallucination}

**幻觉**（Hallucination）指 LLM 生成看似流畅但与事实不符的内容。分为两类：事实性幻觉（编造不存在的论文、数据）和忠实性幻觉（与给定上下文矛盾）。根本原因包括：训练数据中的噪声、自回归生成的概率采样特性、模型倾向于"编造"来填补知识空白。

### Grounding {#grounding}

**接地**（Grounding）指将语言模型的输出锚定到真实世界的信息来源。ReAct 范式中，Agent 通过调用搜索引擎或数据库来"接地"——验证或补充 LLM 的知识。RAG（检索增强生成）是 grounding 的主要技术手段。

### RAG {#rag}

**RAG**（Retrieval-Augmented Generation，检索增强生成）将信息检索与文本生成结合：先从外部知识库检索相关文档片段，再将其作为上下文提供给 LLM 生成回答。解决了 LLM 知识过时和幻觉问题，是当前企业级 Agent 应用的基础架构。

### Constrained Decoding {#constrained-decoding}

**约束解码**（Constrained Decoding）指在 LLM 生成过程中施加硬约束，确保输出满足特定格式（如合法 JSON、有效 SQL）。实现方式包括：在每一步只允许合法 token 的 logit masking、基于文法的引导（Grammar-guided decoding）等。保证 Agent 的工具调用输出格式严格正确。

### Data Contamination {#data-contamination}

**数据污染**（Data Contamination）指评估基准的测试数据在 LLM 的预训练语料中出现过，导致评测成绩虚高——模型可能是在"回忆"答案而非真正推理。这是 LLM 评估中的核心威胁，需要通过持续更新测试集、检测泄漏等方式应对。

### LLM-as-Judge {#llm-as-judge}

**LLM-as-Judge** 是一种自动化评估方法：用另一个（通常更强的）LLM 来评判 Agent 输出的质量。优点是比人工评估便宜快速，缺点是可能引入系统性偏见（如偏好更长或更正式的回答）。通常作为人工评估的辅助而非替代。

### Ablation Experiment {#ablation}

**消融实验**（Ablation Study/Experiment）是一种实验方法论：系统性地移除或禁用系统中的某个组件，观察性能变化来评估该组件的贡献。名称来源于医学（切除组织观察影响）。在 AI 研究中用于回答"这个模块到底有没有用"的问题。

### Mechanistic Interpretability {#mechanistic-interpretability}

**机制可解释性**（Mechanistic Interpretability）是一个研究方向，试图理解神经网络内部"如何"完成特定计算。通过分析单个神经元、注意力头、回路（circuit）的功能来逆向工程模型行为。代表性成果包括 Induction Heads（归纳头）的发现、Superposition（叠加）假说等。

---

## 四、强化学习

### MDP {#mdp}

**MDP**（Markov Decision Process，马尔可夫决策过程）是强化学习的数学框架。由五元组 (S, A, P, R, γ) 定义：状态集 S、动作集 A、转移概率 P(s'|s,a)、奖励函数 R(s,a)、折扣因子 γ。**马尔可夫性**指"未来只取决于当前状态，与到达方式无关"。Agent 的决策过程通常被建模为 MDP。

### POMDP {#pomdp}

**POMDP**（Partially Observable MDP，部分可观测马尔可夫决策过程）是 MDP 的推广——Agent 无法直接观测到完整环境状态，只能获得带噪声的观测。需要维护一个"信念状态"（belief state）——对真实状态的概率分布估计。对话系统常被建模为 POMDP。

### DQN {#dqn}

**DQN**（Deep Q-Network，深度 Q 网络）是 DeepMind (2015) 提出的里程碑算法，首次成功将深度学习与 Q-learning 结合。用神经网络近似 Q 函数 Q(s,a)——预测在状态 s 执行动作 a 的长期期望回报。两个关键创新：经验回放（Experience Replay）和目标网络（Target Network）。

### Q-function {#q-function}

**Q 函数**（Q-function / Action-Value Function）定义为 Q(s,a) = E[Σ γ^t · r_t | s₀=s, a₀=a]——从状态 s 执行动作 a 后，按最优策略行动所能获得的期望累计折扣回报。Q-learning 通过反复更新 Q 值来学习最优策略：总是选择 argmax_a Q(s,a)。

### Policy Gradient {#policy-gradient}

**策略梯度**（Policy Gradient）方法直接参数化策略 π_θ(a|s)（一个从状态到动作概率的映射），通过梯度上升来优化期望回报。REINFORCE 是最简单的策略梯度算法。相比 Q-learning，策略梯度能处理连续动作空间和随机策略，但方差较高。

### Experience Replay {#experience-replay}

**经验回放**（Experience Replay）将 Agent 与环境交互产生的经验 (s, a, r, s') 存入一个缓冲区，训练时从中随机采样而非使用最新经验。两个好处：(1) 打破数据时序相关性（近似 i.i.d.）；(2) 提高数据利用率（同一经验可被多次使用）。

### Target Network {#target-network}

**目标网络**（Target Network）是 DQN 的稳定训练技巧。维护两个网络：在线网络（频繁更新，用于选择动作）和目标网络（定期从在线网络复制，用于计算 TD 目标值）。避免"自己追自己"的不稳定——如果用同一个网络既选择动作又计算目标，训练会振荡。

### Domain Randomization {#domain-randomization}

**域随机化**（Domain Randomization）是 sim-to-real 迁移的技术：在仿真训练时随机化环境参数（光照、摩擦力、纹理、噪声等），使策略对环境变化具有鲁棒性。目标是让真实世界只是"随机化参数空间中的又一个采样"，从而实现仿真到真实的零样本迁移。

### Sim-to-Real {#sim-to-real}

**Sim-to-Real**（仿真到真实迁移）指在仿真器中训练 Agent 策略后部署到真实物理环境。核心挑战是"reality gap"——仿真器无法完美模拟真实物理。常用技术包括域随机化、系统辨识、逐步微调。

---

## 五、形式化方法与规划

### STRIPS {#strips}

**STRIPS**（Stanford Research Institute Problem Solver, 1971）是第一个自动规划系统，也定义了经典规划的标准表示。每个动作由前置条件（Preconditions）、添加效果（Add list）和删除效果（Delete list）描述。规划器搜索满足目标条件的动作序列。

### PDDL {#pddl}

**PDDL**（Planning Domain Definition Language，规划域定义语言）是 STRIPS 的标准化继承者，自 1998 年国际规划竞赛起成为标准。分为域文件（定义动作及其效果）和问题文件（定义初始状态和目标）。LLM+P 等方法用 LLM 生成 PDDL 描述再交给经典规划器求解。

### 框架问题 {#frame-problem}

**框架问题**（Frame Problem, McCarthy & Hayes, 1969）是 AI 哲学中的根本困难：当一个动作执行后，如何形式化描述"哪些事情没有改变"？在逻辑中需要大量"框架公理"来声明不变的状态，数量随动作和状态变量的增多而爆炸。

### 手段-目的分析 {#means-ends-analysis}

**手段-目的分析**（Means-Ends Analysis）是 Newell & Simon (1961) 在 GPS 中提出的问题求解策略：比较当前状态与目标状态的差异（difference），选择能减少该差异的操作（means），递归地应用直到差异为零。是分层规划的思想起源。

### 物理符号系统假说 {#physical-symbol-system}

**物理符号系统假说**（Physical Symbol System Hypothesis, Newell & Simon, 1976）主张：一个物理符号系统（能创建、复制、修改、销毁符号结构的系统）具有通用智能的充分必要条件。这是符号 AI 的哲学基础，也是其局限性的根源——不考虑感知、具身和学习。

### Speech Act Theory {#speech-act-theory}

**言语行为理论**（Speech Act Theory, Austin, 1962; Searle, 1969）认为说话不只是传递信息，也是在执行"行为"——如请求、承诺、命令、声明等。在多 Agent 通信中被借鉴为 Agent 消息的"施为力"（performative）分类，如 FIPA-ACL 协议中的 inform、request、propose 等。

### FIPA-ACL {#fipa-acl}

**FIPA-ACL**（Foundation for Intelligent Physical Agents - Agent Communication Language）是多 Agent 系统中的标准通信语言。基于言语行为理论定义了一组通信行为（communicative acts）：inform（告知）、request（请求）、propose（提议）、accept-proposal（接受提议）等。是 A2A 等现代协议的理论前身。

### 本体论 {#ontology}

**本体论**（Ontology）在 AI/知识工程中指对一个领域中概念、关系和规则的形式化描述。它定义了 Agent 之间的"共享词汇"——确保不同 Agent 对"订单"、"客户"、"地址"等概念有一致的理解。典型标准包括 OWL（Web Ontology Language）。

### Constitutional AI {#constitutional-ai}

**Constitutional AI**（宪法式 AI, Bai et al., 2022）是 Anthropic 提出的对齐方法。用一组明确的"宪法原则"替代人类标注者：(1) 让 AI 自我批评并修改回答（RLAIF 中的 AI 反馈）；(2) 根据宪法原则进行偏好排序训练。减少了对人类标注的依赖，且原则可以明确审计。

---

## 六、认知科学与经典 Agent 架构

### 产生式规则 {#production-rules}

**产生式规则**（Production Rules）是认知架构中知识表示的基本单元，形如"IF 条件 THEN 动作"。在 Soar 和 ACT-R 中，Agent 的行为完全由产生式规则库驱动：工作记忆中的状态匹配规则条件 → 触发规则执行。

### Chunking {#chunking}

**Chunking**（组块化）在 Soar 中是学习机制：当 Agent 通过子目标搜索解决了一个问题后，将整个搜索过程编译为一条新的产生式规则。下次遇到相同情况时直接触发该规则，不需要再搜索。类比人类"熟练化"——初学者逐步推理，专家直接反应。

### 扩散激活 {#spreading-activation}

**扩散激活**（Spreading Activation）是 ACT-R 中的记忆检索机制。当某个概念被激活时，激活沿语义网络的连接扩散到相关概念——就像池塘中投石后涟漪向外传播。用于模拟人类联想记忆和语境效应。

### Subsumption Architecture {#subsumption}

**包容架构**（Subsumption Architecture, Brooks, 1986）是反应式 Agent 的经典设计。多层行为模块并行运行，高层可以"包容"（抑制/替代）低层的输出。无需世界模型和显式规划——复杂行为从简单行为层的交互中涌现。

### BDI {#bdi}

**BDI**（Belief-Desire-Intention，信念-愿望-意图）是 Agent 架构的哲学基础（Bratman, 1987; Rao & Georgeff, 1995）。Belief 是 Agent 对世界的认知，Desire 是目标集合，Intention 是已承诺去追求的目标和计划。通过"实践推理"从 Belief 和 Desire 中选择 Intention 并执行。

### 情景记忆 {#episodic-memory}

**情景记忆**（Episodic Memory, Tulving, 1972）是认知科学中三种长期记忆之一（另两种是语义记忆和程序记忆）。存储个人经历的"时空事件"——什么时候在什么地方发生了什么。在 Agent 中对应执行历史记录——Reflexion 用情景记忆存储过去的成功/失败经验来指导未来决策。

---

## 七、NLP 与对话系统

### Seq2Seq {#seq2seq}

**Seq2Seq**（Sequence-to-Sequence, Sutskever et al., 2014）是将输入序列映射到输出序列的模型框架。由编码器（Encoder）将输入压缩为固定向量，解码器（Decoder）从该向量生成输出序列。是机器翻译、对话生成、文本摘要的基础架构，后被 Transformer 取代。

### Slot Filling {#slot-filling}

**槽填充**（Slot Filling）是任务型对话系统中 NLU 的核心技术。预定义对话中需要收集的信息"槽"（如订餐需要 [日期]、[时间]、[人数]、[餐厅]），系统逐步从用户话语中提取这些信息填入对应槽位。

### DST {#dst}

**DST**（Dialogue State Tracking，对话状态跟踪）在多轮对话中维护当前"对话状态"——所有已填充的槽值和用户意图的综合表示。是对话管理的前置模块——DST 的准确性直接决定系统能否理解用户在多轮交互中的完整需求。

### CRF {#crf}

**CRF**（Conditional Random Field，条件随机场）是一种判别式序列标注模型。给定输入序列，对输出标签序列建模其条件概率。考虑了标签之间的依赖关系（如 NER 中 B-Person 后面不能跟 B-Organization）。在深度学习时代常作为 BiLSTM/BERT 的最后一层。

### RNN / LSTM {#rnn-lstm}

**RNN**（Recurrent Neural Network，循环神经网络）通过隐藏状态在时间步间传递信息来处理序列数据。**LSTM**（Long Short-Term Memory）是 RNN 的改进变体，通过门控机制（遗忘门、输入门、输出门）解决了长序列中的梯度消失问题。曾是 NLP 的主流架构，后被 Transformer 取代。

### OpenAPI Specification {#openapi}

**OpenAPI Specification**（前身为 Swagger）是描述 REST API 的标准化格式。用 YAML/JSON 定义 API 的端点、请求参数、响应格式、认证方式等。在 Agent 工具调用中，OpenAPI 文档为 LLM 提供了结构化的工具描述——模型"阅读" API 规范后即可生成正确的调用。

---

## 八、分布式系统与工程

### JSON-RPC {#json-rpc}

**JSON-RPC** 是一个轻量级的远程过程调用协议，使用 JSON 格式编码。核心要素：method（方法名）、params（参数）、id（请求标识）。MCP 和 A2A 协议都基于 JSON-RPC 2.0 构建通信层。

### SSE {#sse}

**SSE**（Server-Sent Events，服务器推送事件）是 HTTP 协议上的单向实时通信标准。服务端通过一个持久 HTTP 连接不断向客户端推送文本事件流。与 WebSocket 的区别：SSE 是单向的（服务器→客户端），基于标准 HTTP（天然支持代理/防火墙），自动重连。适合 Agent 流式输出。

### Chunked Transfer Encoding {#chunked-encoding}

**分块传输编码**（Chunked Transfer Encoding）是 HTTP/1.1 的传输机制：服务器不需要预知响应总长度，可以分块发送数据，每块前标注该块长度，最后发送一个零长度块表示结束。是 LLM 流式响应的底层传输基础。

### Pub-Sub {#pub-sub}

**Pub-Sub**（Publish-Subscribe，发布-订阅）是消息通信模式：发布者不直接发送消息给接收者，而是发布到"主题"（Topic）；订阅者注册感兴趣的主题后自动收到该主题的消息。实现了发送方和接收方的解耦。在多 Agent 系统中用于事件广播和松耦合通信。

### OpenTelemetry {#opentelemetry}

**OpenTelemetry**（OTel）是可观测性数据的开源标准和 SDK 集合。统一了三大可观测性信号的采集：Traces（追踪）、Metrics（指标）、Logs（日志）。提供语言无关的 API + SDK + Collector 架构，是当前 Agent 系统可观测性的事实标准。

### Trace / Span {#trace-span}

**Trace**（追踪）表示一个请求在分布式系统中的完整生命周期路径。**Span** 是 Trace 中的一个工作单元（如一次 API 调用、一次数据库查询）。Span 之间有父子关系，形成树状结构。在 Agent 中，一次用户请求 → LLM 推理 → 工具调用 → 结果整合可以表示为一条 Trace 中的多个 Span。

### Fail-safe / Defense-in-depth {#fail-safe}

**Fail-safe**（失败安全）是设计原则：系统故障时默认进入安全状态而非危险状态。**Defense-in-depth**（纵深防御）是安全架构原则：部署多层独立防护，使得单层被突破不会导致整体失陷。在 Agent 安全中，两者结合意味着：工具调用失败时拒绝执行而非随意尝试 + 多层输入/输出过滤器。

### Docker 容器 {#docker}

**Docker 容器**是轻量级虚拟化技术：将应用及其依赖打包为一个标准化"容器"，在任何支持 Docker 的环境中一致运行。通过 Linux cgroup（资源限制）和 namespace（隔离）实现进程级隔离。在 Agent 评估中（如 SWE-bench），每个任务在独立容器中执行以确保环境隔离。

---

## 九、博弈论与多 Agent

### Vickrey 拍卖 {#vickrey-auction}

**Vickrey 拍卖**（维克里拍卖 / 第二价格密封拍卖）：投标者提交密封出价，最高出价者获胜，但只需支付第二高出价的金额。核心性质：真实报价是每个参与者的最优策略（激励兼容），无需猜测他人出价。在多 Agent 资源分配中用于设计激励相容的协调机制。

### 组合拍卖 {#combinatorial-auction}

**组合拍卖**（Combinatorial Auction）允许投标者对物品的组合（而非单个物品）出价。例如"我愿意出 100 元买 A+B，但单独买 A 或 B 都不值"。找到最优分配方案是 NP-hard 的。在多 Agent 任务分配中用于处理任务间存在互补或替代关系的场景。

### Contract Net Protocol {#contract-net}

**合同网协议**（Contract Net Protocol, Smith, 1980）是多 Agent 协调的经典机制。流程：管理者广播任务公告 → 承包者提交投标 → 管理者评估并授予合同 → 承包者执行并报告。类比人类世界的招投标过程，实现了去中心化的任务分配。

---

## 十、Agent 工程实践

### Instruction Tuning {#instruction-tuning}

**指令微调**（Instruction Tuning）是在大量（指令, 回答）配对数据上对预训练模型做 SFT，使模型学会"遵循自然语言指令"的能力。与普通 SFT 的区别在于覆盖了海量不同类型的指令（翻译、总结、编程、问答等），使模型获得泛化的指令跟随能力。FLAN-T5, InstructGPT 是代表性工作。

### DAG {#dag}

**DAG**（Directed Acyclic Graph，有向无环图）是图论中的基础结构：节点间有方向性连接，且不存在环路。在 Agent 工程中用于表示任务依赖关系——节点是任务，边是依赖（A→B 表示 B 依赖 A 的完成）。DAG 保证不会出现循环依赖，可通过拓扑排序确定执行顺序。

### FSM {#fsm}

**FSM**（Finite State Machine，有限状态机）是计算理论中最基础的计算模型：有限个状态 + 转移规则（当处于状态 A 且收到输入 X 时，转移到状态 B）。在 Agent 架构中用于管理对话流程或任务执行阶段——每个状态对应 Agent 的一个工作模式，事件触发状态转换。

### A/B 测试 {#ab-test}

**A/B 测试**是在线对照实验方法：将用户随机分为两组，一组使用原版（A/控制组），一组使用新版（B/实验组），通过统计检验比较两组的指标差异是否显著。在 Agent 评估中用于比较不同提示策略、模型版本或架构选择的实际效果。

### Fail-to-pass {#fail-to-pass}

**Fail-to-pass** 是 SWE-bench 的评判标准：Agent 提交的代码补丁必须使原本失败的测试用例变为通过（fail → pass），同时不能让原本通过的测试用例变为失败（pass → fail）。这确保了补丁既修复了 bug 又不引入新的回归。

---

## 快速索引

| 首字母 | 术语 |
|--------|------|
| A | [A*](#a-star), [A/B 测试](#ab-test), [Ablation](#ablation), [Alignment](#alignment), [ANN](#ann), [Attention Head](#attention-head) |
| B | [BDI](#bdi), [BFS/DFS](#bfs-dfs), [BM25](#bm25), [贝叶斯网络](#bayesian-network) |
| C | [Chunking](#chunking), [CRF](#crf), [Constrained Decoding](#constrained-decoding), [Constitutional AI](#constitutional-ai), [Contract Net](#contract-net) |
| D | [DAG](#dag), [DQN](#dqn), [DPO](#dpo), [DST](#dst), [Docker](#docker), [Domain Randomization](#domain-randomization), [Dead-letter queue](#dead-letter-queue) |
| E | [Embedding](#embedding), [Emergent Abilities](#emergent-abilities), [ETL](#etl), [Experience Replay](#experience-replay) |
| F | [FSM](#fsm), [FIPA-ACL](#fipa-acl), [Fail-safe](#fail-safe), [Fail-to-pass](#fail-to-pass), [Fan-out](#fan-out), [Few-shot/Zero-shot](#few-zero-shot) |
| G | [GRPO](#grpo), [Grounding](#grounding) |
| H | [Hallucination](#hallucination), [HNSW](#hnsw), [启发式函数](#heuristic) |
| I | [i.i.d.](#iid), [ICL](#in-context-learning), [Instruction Tuning](#instruction-tuning), [IVF](#ivf) |
| J | [JSON-RPC](#json-rpc) |
| K | [K-Means](#k-means), [KL 散度](#kl-divergence) |
| L | [Logits](#logits), [LoRA](#lora), [LLM-as-Judge](#llm-as-judge), [LRU](#lru) |
| M | [MapReduce](#mapreduce), [MCTS](#mcts), [MDP](#mdp), [MLP](#mlp), [Mechanistic Interpretability](#mechanistic-interpretability) |
| N | [NP-hard](#np-hard), [NP-complete](#np-complete) |
| O | [O(N)](#big-o), [OpenAPI](#openapi), [OpenTelemetry](#opentelemetry), [ORM/PRM](#prm-orm), [本体论](#ontology) |
| P | [PDDL](#pddl), [POMDP](#pomdp), [PPO](#ppo), [PSPACE-complete](#pspace-complete), [Pareto optimal](#pareto-optimal), [Policy Gradient](#policy-gradient), [Pub-Sub](#pub-sub), [产生式规则](#production-rules) |
| Q | [Q-function](#q-function), [Quorum](#quorum) |
| R | [RAG](#rag), [RLHF](#rlhf), [RNN/LSTM](#rnn-lstm), [Residual Stream](#residual-stream) |
| S | [Saga](#saga), [Scaling Laws](#scaling-laws), [Self-Attention](#self-attention), [Seq2Seq](#seq2seq), [SFT](#sft), [Sim-to-Real](#sim-to-real), [Slot Filling](#slot-filling), [Softmax](#softmax), [SSE](#sse), [STRIPS](#strips), [Subsumption](#subsumption), [Speech Act Theory](#speech-act-theory), [扩散激活](#spreading-activation) |
| T | [Target Network](#target-network), [Temperature](#temperature), [Trace/Span](#trace-span), [Transformer](#transformer), [拓扑排序](#topological-sort) |
| V | [Vickrey 拍卖](#vickrey-auction), [Voronoi](#voronoi) |
| 其他 | [2-EXPTIME](#2-exptime), [Data Contamination](#data-contamination), [Chunked Encoding](#chunked-encoding), [框架问题](#frame-problem), [物理符号系统假说](#physical-symbol-system), [手段-目的分析](#means-ends-analysis), [情景记忆](#episodic-memory), [组合拍卖](#combinatorial-auction), [组合爆炸](#combinatorial-explosion) |
