+++
date = '2025-11-11T10:00:00+08:00'
draft = true
title = '2025.11.11'
+++

<!--more-->

工作：
- 压测
- 接数据集


```bash

问题：
1c没有跑完，具体错误是什么？
cpu 跑满时，最耗cpu的操作是什么
问下运维同学，为什么内存99了却没有触发oom
最耗内存的是什么？

数据：
8c 16g
8c 24g
8c 32g


```


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

包含预训练/强化

算法-》数据-》infra

