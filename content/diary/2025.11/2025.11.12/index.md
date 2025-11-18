+++
date = '2025-11-12T10:00:00+08:00'
draft = true
title = '2025.11.12'
+++

<!--more-->

工作：
- 压测
  - 压30分钟，持续压30分钟
  - 迭代轮次增加，需要蒸馏平台
  - 添加性能分析


- 开会
  - 外部需求
    - agent和数据集有什么关系？
      - swe-agent 天花板更高，这都是agent能力决定的，对他们的影响是什么，sy搞的
      - 开源外网，适配外网
        - 原生有bug，10多个能确定能修对么？-字节同学提的-一个个修-开源镜像不对，中国以外跑不过
        - 更多的确认是bug-明确改哪里
      - ai search agent
        - 自己的定位和分工是什么？
  - 内部规划
    - 蓝绿发布 控制不一致
    - 接口标准化
      - 训练框架的标准接口--假定一个场景，否则没有办法验收是标准的，什么样的接口是标准的？有些平台是标准的，无需额外的开发，现存的agent开发
      - 用coze /dify有没有标准，用agent kit的怎么迁移过来
      - 对env提供gym标准接口 上游是rllm，gym就是可以的
        - https://gymnasium.farama.org/index.html
      - 幂等性保证：当一个task失败了，一个job
        - agent：chat complation无状态自身不幂等，用response api和上游taskd+step_id，这个要重新想办法
      - 想给openhands加监控，openhands，监控只是发现问题，并不解决问题，定位问题在哪里
        - 精简一点的监控
        - 因为遇到过资源不对等
        - 开源脚手架打点，先不谈怎么做，能不能提升排查效率
      - 工具限流
        - 暴露的问题不明显所以不急
      - RL infra竞品接口，agent能力常用框架
      - 推理和训练都是通用的
        - 一站式是怎么训练的？-RL infra
    - 接入测试
      - 保证系统正确性，想端到端的测试，我关心我们这块事情有没有cover住，其他模块完全不在我们控制之下，按照测试的规范做事的


  
其他：
- 找了心理咨询师聊了一下，主要就是讨论为什么迟迟行动不了
  - 结论是 投入时间还是少，缺少规划
  - 我是一直有能力的，只不过不习惯使用了，捡起来！



```bash
python -m cProfile -s time your_script.py
```



```bash

curl --location 'http://trpc.agent-flow.controller.x3.pre.polaris:8000/api/agentflow/v1/resource/release' \
--header 'Content-Type: application/json' \
--data '{
    "job_id": "test_job_quinnyqwu_073'"
}'

```


