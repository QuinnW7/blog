+++
date = '2025-11-04T10:00:00+08:00'
draft = false
title = 'AngelAgent'
+++

feature/angentflow-deepswe 分支


<!--more-->

使用了openai gym的标准env环境接口，对接我们的内部接口，现在需要找到具体怎么用的

资源相关：
https://git.woa.com/AngelAGI/AngelAgent/tree/feature/angentflow-deepswe/scripts/agentflow


分为3个步骤：
- 申请资源
- 启动job
- 启动task


# 智能体和环境交互
## 接口
- CreateEnv
- AgentInput
- Submit
- Finish
- Reward
- CloseEnv

## 时序图
```mermaid
sequenceDiagram
    participant Agent as 智能体 (Agent)
    participant Environment as 环境 (Environment)

    %% ======================= 1. 初始化阶段 (Initialization) =======================
    Note over Agent, Environment: 1. 初始化阶段
    Agent->>Environment: CreateEnvSpec(job_name, task_name)
    activate Environment
    Note right of Environment: 环境开始创建任务实例...
    Environment-->>Agent: CreateEnvResponse(status='success', error=None)
    deactivate Environment

    %% ======================= 2. 核心交互循环 (Interaction Loop) =======================
    Note over Agent, Environment: 2. 核心交互循环
    
    loop 任务交互循环
        Agent->>Environment: AgentInputSpec(task_name, step)
        activate Environment
        Note right of Environment: 环境准备当前状态
        Environment-->>Agent: AgentInputResponse(observation, done, ...)
        deactivate Environment
        
        Note left of Agent: 智能体检查 `done` 标志
        
        %% --- 思考与决策 (Think & Decide) ---
        activate Agent
        Note left of Agent: 分析 observation，决定下一步行动
        deactivate Agent
        
        alt Agent 决定继续执行动作
            %% --- 行动 (Act) ---
            Agent->>Environment: SubmitSpec(response="some_action", step)
            activate Environment
            Note right of Environment: 环境执行动作并更新状态
            Environment-->>Agent: SubmitResponse(status='success', error=None)
            deactivate Environment

        else Agent 决定主动结束任务
            Agent->>Environment: FinishSpec(finish_reason="completed", step)
            activate Environment
            Environment-->>Agent: FinishResponse(error=None)
            deactivate Environment
            Note over Agent, Environment: 智能体主动结束，跳出循环
            %% 修正点：移除了此处无效的 `break` 关键字
        end
        
    end
    Note over Agent, Environment: 循环结束 (因 done=True 或 Agent 主动 break)

    %% ======================= 3. 奖励评估阶段 (Evaluation) =======================
    Note over Agent, Environment: 3. 奖励评估阶段
    Agent->>Environment: RewardSpec(task_name)
    activate Environment
    Note right of Environment: 环境计算最终奖励
    Environment-->>Agent: RewardResponse(reward=0.95, finish=True, ...)
    deactivate Environment

    %% ======================= 4. 资源清理阶段 (Cleanup) =======================
    Note over Agent, Environment: 4. 资源清理阶段
    Agent->>Environment: CloseEnvSpec(task_name)
    activate Environment
    Note right of Environment: 环境释放所有相关资源
    Environment-->>Agent: CloseEnvResponse(status='success')
    deactivate Environment
    
```

```mermaid
classDiagram
    direction LR

    class BaseEnv {
        <<abstract>>
        +reset()
        +step()
        +close()
    }

    class AgentFlowEnv {
        -base_url: str
        -job_name: str
        -env: AgentFlowGym
        +reset()
        +step()
        +close()
        +compute_final_reward()
    }

    class AgentFlowSWEEnv {
        -entry: dict
        -max_steps: int
        +create_metadata(): dict
        +step(action)
    }

    class AgentFlowGym {
        -client: V1Client
        -task_name: str
        -step_id: int
        +reset()
        +step(action)
        +close()
        +watch_observation()
        +watch_reward()
        +compute_final_reward()
    }
    
    %% --- Relationships ---
    BaseEnv <|-- AgentFlowEnv : "继承"
    AgentFlowEnv <|-- AgentFlowSWEEnv : "继承"
    AgentFlowEnv o-- "1" AgentFlowGym : "组合/使用"
  
```




```mermaid
sequenceDiagram
    actor rllm
    participant SWE_Env as AgentFlowSWEEnv
    participant Base_Env as AgentFlowEnv
    participant Gym_Env as AgentFlowGym
    participant Service as AgentFlow Service (API)

    User/Framework->>SWE_Env: step(action)

    activate SWE_Env
    Note right of SWE_Env: 包含 try...except 错误处理
    SWE_Env->>Base_Env: super().step(action) // 调用父类的 step
    deactivate SWE_Env

    activate Base_Env
    Base_Env->>Gym_Env: step(action) // 将 step 调用委托给 AgentFlowGym
    deactivate Base_Env

    activate Gym_Env
    Note right of Gym_Env: 提交动作并等待结果
    Gym_Env->>Service: HTTP POST /submit (携带 action)
    Service-->>Gym_Env: Ack (动作已接收)

    loop Polling for next observation
        Gym_Env->>Gym_Env: watch_observation()
        Gym_Env->>Service: HTTP GET /get_agent_input
        Service-->>Gym_Env: return next_observation, done, info (或空)
    end
    Note right of Gym_Env: 成功获取到新 observation
    Gym_Env-->>Base_Env: return next_obs, reward, done, info
    deactivate Gym_Env

    Base_Env-->>SWE_Env: return next_obs, reward, done, info
    SWE_Env-->>User/Framework: return next_obs, reward, done, info
```