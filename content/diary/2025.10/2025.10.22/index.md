+++
date = '2025-10-22T10:00:00+08:00'
draft = true
title = '2025.10.22'
+++

<!--more-->

 
工作：
- 压测
  - 晚点找zhe对下，要这两天给结果，笑死，不过应该很快
    - 去掉过载保护
    - 随便写个job_id
    - 开始跑就完了

```bash

要先TEST_CONCURRENCY 改成1
然后 python test_controller.py 跑一次抓取（缓存 llm 的内容）

然后 TEST_CONCURRENCY 改成要测的并发
MOCK_MODE=1 python test_controller.py 跑压测

```
- 先压40，看监控cpu跑到多少，然后逐步向上压

- 看监控
  https://zhiyan.woa.com/monitor/9860/detail_chart/#/redirect?_link=share&to=fb31a6065f
  https://zhiyan.woa.com/monitor/9860/detail_chart/#/redirect?_link=share&to=8512fc2ff0
  https://zhiyan.woa.com/monitor/9860/detail_chart/#/redirect?_link=share&to=a869a0ac47
  https://zhiyan.woa.com/monitor/9860/detail_chart/#/redirect?_link=share&to=2f53652929

【企微文档】proxy压测
https://doc.weixin.qq.com/sheet/e3_AEUADQZHACgCNrRwVmp6fRVOQ1oSX?scode=AJEAIQdfAAodlaRLrdAEUADQZHACg&tab=BB08J2


~~- QueryRewrite 告警~~
  - 让云三开白名单

- wx和yx确认明天发版内容
  - 确认了，需要改下
  - 改完了，等yx体验下

提升：
- L12
~~- 练功~~

其他：
- 行家时间预约
  - 等回复
- 周末M2 要不要去

