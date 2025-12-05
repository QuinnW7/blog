+++
date = '2025-11-27T10:00:00+08:00'
draft = true
title = '2025.11.27'
+++
<!--more-->

工作：
- 压测分析遗留事项：
  1、completion改成异步调用接口
  2、日志级别提升
  3、sleep改成1秒
- X0分享
  - 需要调研一下nix https://nixos.org/




压测分析遗留事项：
1、completion改成异步调用接口
2、日志级别提升
3、sleep改成1秒

跟进：
1、改成异步后,其他条件不变情况下(80并发)，cpu利用率上去了，且锁的时间降低很多


https://zhiyan.woa.com/monitor/9860/tke_detail_chart/#/redirect?_link=share&to=3e5026f6fc

具体改动：
在pod中，找到lib/openhands文件夹
- openhands/llm/llm.py
# from litellm import completion as litellm_completion
from litellm import acompletion as litellm_acompletion
# ... (在 __init__ 方法中) ...
        # litellm_completion改成了 litellm_acompletion
        self._completion = partial(
            litellm_acompletion,  
           ...）
将 wrapper 声明为 async def，并在调用底层函数时添加 await。
async def wrapper(*args: Any, **kwargs: Any) -> Any
resp: ModelResponse = await self._completion_unwrapped(*args, **kwargs)
- openhands/controller/agent_controller.py
  找到 async def _step(self) -> None: 方法，将其中的 self.agent.step(self.state) 改为 await 调用
- openhands/controller/agent.py
  class Agent:
  async def step(self, state: State) -> Action

2、日志级别官方不支持修改，与日志相关的配置只有如下两个变量

3、改成sleep(1)后，确实可以缩短cpu占用时间



提升：
- CL14，作业挺多的，时间跨度很长，我觉得主要是练习中枢，当然了作为之前的复习真的很不错




  

  

  
