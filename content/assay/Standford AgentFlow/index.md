+++
date = '2025-11-04T10:00:00+08:00'
draft = false
title = 'AgentFlow By Standford'
+++

https://agentflow.stanford.edu/

<!--more-->
![img.png](img.png)

可以看图中的memory和4个模块的关系

图中的Memory简写为M，tookit简写为K,用户的输入given query 简写为q,在planner上有个小火苗表示自动优化，适应



# 是什么？
A trainable, in-the-flow agentic framework that coordinates four modules (planner, executor, verifier, generator) through an evolving memory and directly optimizes its planner inside the multi-turn loop
AgentFlow is a general-purpose tool-integrated agentic framework for solving complex reasoning tasks through fine-grained planning and effective tool use.


- 通用的集成工具的框架
- 为了解决复杂 问题
- 可训练、流程中的智能体框架
- 通过不断进化的memory协调4个模块(planner, executor, verifier, generator)
- 直接在多轮循环中优化planner


-- memory如何演化的？又是如何协调4个模块的？
- 那肯定是将很多数据直接更新到一个库里吧？
- 在流程上是在每一轮verifier后将结果加到memory中的
- verifier产生二进制信号，如果Vt==0，怎记忆直接更新，这个直到Vt==1或达到最大轮次
- 4个模块怎么使用这个memory的？
- 从memory中召回相关的上下文


--  如何直接优化planner的？
-


- 如何解决复杂问题的？
    - 通过多轮Markov Decision Process (MDP)，马尔可夫决策过程
- 什么是马尔可夫决策过程？
    - https://hrl.boyuai.com/chapter/1/%E9%A9%AC%E5%B0%94%E5%8F%AF%E5%A4%AB%E5%86%B3%E7%AD%96%E8%BF%87%E7%A8%8B

# 为什么？
# 怎么用？

https://huggingface.co/spaces/AgentFlow/agentflow
# 

https://iwiki.woa.com/p/4016371576?from=recent_edit



---


https://agentflow.stanford.edu/

成果





在 10 个不同的基准测试中进行性能比较。

基于 7B 规模的 AgentFlow在搜索、代理、数学和科学推理任务方面均较表现最佳的基准取得了显著的改进。

定义

A trainable, in-the-flow agentic framework that coordinates four modules (planner, executor, verifier, generator) through an evolving memory and directly optimizes its planner inside the multi-turn loopAgentFlow is a general-purpose tool-integrated



为了解决复杂问题

通用的工具集成的框架

可训练、流程中的智能框架

通过不断进化的memory协调4个模块(planner, executor, verifier, generator)

直接在多轮循环中优化planner





工作流程





如图所示，是AgentFlow 在第t轮处理任务的工作流程。流程中涉及了四个核心模块：规划器（Planner）、执行器（Executor）、验证器（Verifier）和记忆（Memory）。

q: Query，即用户提出的问题或任务。

M: Memory，即智能体的记忆，包含了之前的对话历史、行动和观察结果。

K: Toolset，即智能体可以使用的工具集，比如代码解释器、搜索引擎等。

其中黄色的规划器是代表“可训练的”，它的参数πθ可以在训练过程中被优化和调整，使其能力不断提升。蓝色的执行器和验证器在流程中是不进行参数更新的。

Planner

directly optimizes its planner inside the multi-turn(多轮对话内直接优化计划模块)

功能：分析用户的查询，结合全局目标和现有技能，决定当前这一步具体要做什么。

特点：

可训练的：它的参数πθ可以在训练过程中被优化和调整

在当前时间步 t 开始时的记忆状态。包含了之前的对话历史、已完成的步骤等上下文。





Executor

使用工具执行操作

工具集合：



Verifier

功能：验证器分析执行结果，判断这一步操作是否成功、是否有用、是否离最终目标更近了一步。二进制数据

v^t = 1 意味着：“任务完成！ 当前的结果 e^t 已经足够好了，可以终止循环并生成最终答案了。”

v^t = 0 意味着：“任务还没完，继续干！ 当前的结果 e^t 还不够好，或者只是一个中间步骤，我们需要基于现在的结果进行下一步的思考。”

Memory

evolving memory (不断"进化"的记忆)



读：规划器（Planner）、执行器（Executor）、验证器（Verifier）都会召回记忆（Memory）中相关的上下文

写：来自验证器的验证结果 vᵗ 会与其他信息（如思考 aᵗ 和执行结果 eᵗ）一起被存入记忆。这个 vᵗ 信号本身，会指导系统的控制器决定是基于更新后的记忆继续下一个循环（当 vᵗ=0 时），还是终止任务（当 vᵗ=1 时）。



技术核心：Flow-GRPO

解决问题：Planner的参数πθ如何在训练过程中被优化和调整？

流程

Flow-GRPO（Flow-based Group Refined Policy Optimization）算法，属于强化学习的一种。

目标：训练planner，即策略模型Policy Model。



如图所示，从左到右展示了单次训练迭代（training iteration）的全过程：

左侧：输入(query/memory/tools)

中间：

Policy Model策略模型--planner

参数在训练中更新

它接收 q, M, K 作为输入，然后决定下一步的行动（action）

Reference Model参考模型

参数在训练中不变

防止训练中的策略模型偏离一个已知的、相对可靠的基线太远

KL 箭头表示训练时会计算策略模型与参考模型之间的KL散度（Kullback-Leibler divergence），并将其作为惩罚项加入到目标函数中。

KL散度：也称为相对熵，是一种用来衡量两个概率分布之间差异的度量。 它的值总是非负的，当两个分布完全相同时，值为零。

生成多轮轨迹(Multi-turn Agentic System Rollouts)

多轮：同一个任务执行多次尝试，并记录每次尝试的轨迹

图中的每一行代表每一个轨迹，图中有G个轨迹

Reward Modek奖励模型

所有G次尝试的结果O都会被送到奖励模型评估，并生成奖励值R

这个算法采用最终奖励。即每个给定轨迹的中的所有A，获得的奖励都是相同的，都是基于最终结果 O 的奖励

二元奖励，即失败为0，成功为1

右侧：策略更新

收集所有G次的轨迹行动和奖励

利用这些数据，结合当前的策略模型和参考模型，计算如何更新策略模型

反馈给策略模型，完成一次参数的更新，闭环

Flow-GRPO 公式



非常稳健的强化学习规则，其核心在于通过并行运行多组尝试（rollouts）并比较它们的相对好坏，来生成高质量、低方差的训练信号。

解决了稀疏奖励问题，因为即使只有少数尝试成功，它们的优势（Advantage）也会非常显著。

同时，该公式内置了两种关键的安全机制：

采用PPO的裁剪目标函数（min{...}），这就像一个“安全刹车”，严格限制了每一步策略更新的幅度，防止模型因某次大奖励而过度优化导致性能崩溃；

通过KL散度惩罚，确保模型在学习新任务时，其核心能力不会与一个可靠的参考模型偏离太远，从而保证了学习的稳定性和泛化能力。





解决稀疏奖励问题

1、稀疏奖励：在很多复杂的任务中，AI 只有在最终完成任务时才能得到一个奖励（比如+1代表成功，0代表失败）。在中间的所有步骤，它都得不到任何反馈（奖励全是0）。

（这就像在黑暗的迷宫里找出口：你走了99步，都不知道自己走得对不对，只有在第100步找到出口时，才突然得到“你成功了”的信号。这让学习变得非常困难，因为AI不知道这99步里，哪一步是关键，哪一步是多余的。这个学习信号非常稀疏和微弱。）

2、传统方法：

假设我们并行运行了 20次 尝试 (G=20)，其中只有 1次 成功，19次失败。

成功的奖励 R_success = 1

失败的奖励 R_failure = 0

如果直接用这个奖励 R 作为学习信号：

成功的那条路径，得到的信号是 +1。

失败的那19条路径，得到的信号是 0。

+1 的信号虽然比 0 好，但它并不突出，信号强度很有限。模型可能需要很多很多次这样的 +1 才能慢慢学会。

3、Flow-GRPO 的巧妙之处：利用“比较”来放大信号



我们用上面那个 1 次成功、19 次失败的例子来代入计算：

计算团队平均分 mean:mean = (1 * 1 + 19 * 0) / 20 = 1 / 20 = 0.05

计算那条“成功路径”的优势 A_success:

分子 (与平均分的差距): R_success - mean = 1 - 0.05 = 0.95

分母 (团队得分的波动 std): 当大部分数据都一样时（19个0），标准差会非常小。我们大致估算，它约等于 0.22。

最终优势: A_success = 0.95 / 0.22 ≈ +4.3

计算一条“失败路径”的优势 A_failure:

分子: R_failure - mean = 0 - 0.05 = -0.05

最终优势: A_failure = -0.05 / 0.22 ≈ -0.22

4.结论：信号被戏剧性地放大了！

现在对比一下：

传统方法 (直接用奖励)

Flow-GRPO 方法 (用优势A)

成功的信号

+1

+4.3

失败的信号

0

-0.22

原本一个不起眼的 +1 奖励，通过和“团队的普遍失败”进行比较，被放大成了一个极其强大的 +4.3 的正向信号。同时，失败的信号只是一个微弱的负值，告诉模型“这只是普遍情况，不用过度反应”。

这就是这句话的核心含义：
在稀疏奖励的环境下，成功本身就是一个统计上的异常事件 (outlier)。Flow-GRPO的公式通过数学手段捕捉到了这个“异常”的显著性。它不再问“你是否成功？”，而是问“在大家都失败的情况下，你的成功有多么突出？”。

因为成功非常罕见，所以它的“突出程度”就非常高，从而产生了一个巨大、清晰的学习信号，让模型能够从极少数的成功案例中高效地学习，从而巧妙地解决了稀疏奖励问题。



PPO的裁剪目标函数

在传统的强化学习算法中，策略更新的大小约等于 ρ * A (重要性比率 × 优势)。这里存在一个巨大的风险：

如果某次更新中，新策略 π_θ 偶然发现了一个能获得巨大奖励 A 的动作，它可能会变得非常“激进”，使得 ρ 的值变得极大（比如 10 或 20）。

ρ * A 的结果就会是一个爆炸性的数值。

这次巨大的更新会彻底改变模型的行为，很可能把之前学到的所有好策略都“冲刷”掉，导致性能断崖式下跌。这就是所谓的“策略崩溃” (Policy Collapse)。





这是Proximal Policy Optimization (PPO) 算法的标志性部分，其设计目标是实现稳定的策略更新。

它计算了两个值，并取其中较小的一个作为最终的目标。

第一个值 ρ * A 是标准的策略梯度目标，它根据动作的优势 A 和新旧策略的比率 ρ 来调整策略。

第二个值 clip(ρ, 1-ε, 1+ε) * A 是一个被约束的版本。其中clip函数强行将比率 ρ 限制在一个很小的区间 [1-ε, 1+ε] 内。

min 的作用是施加一个悲观界限：它防止策略更新的步子过大。如果新策略相比旧策略变化过于激进（ρ 值过大或过小），这个 clip 机制就会生效，从而限制单次更新的幅度，避免了训练过程中的性能崩溃。



其中重要性比率 ρ ：



分子是新策略：代表在某个情况下，你现在会怎么做的

分母是旧策略：代表着当初你收集数据时，过去是怎么做的

对于同一个行为，新策略去做的可能性是旧策略的多少倍

ρ > 1: 新策略比旧策略更倾向于采取这个行动。

ρ < 1: 新策略比旧策略更不倾向于采取这个行动。

ρ = 1: 新旧策略在这个行动上的表现完全一样

重要性比率 ρ 就是一个修正因子。当它与优势 A 相乘时 (ρ * A)，它在做这样一件事：

如果一个动作在旧策略下很少发生，但在新策略下很可能发生 (ρ 很大)，那么我们就应该增加这次动作结果（奖励或惩罚）的权重。

反之，如果一个动作在旧策略下很常见，但在新策略下不太可能发生 (ρ 很小)，我们就应该降低这次结果的权重。



使用

https://huggingface.co/spaces/AgentFlow/agentflow

