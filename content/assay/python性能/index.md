+++
date = '2025-11-12T10:00:00+08:00'
draft = false
title = 'python性能分析'
+++

参考文章：https://blog.csdn.net/weixin_43700866/article/details/134443380


kubectl cp angel-agent-pre-prj-jp2sncp2/agent-flow-openhands-proxy-75767487d6-7zd28:/usr/local/agent-flow-proxy/remote_openhands/mem.py mem.py

当前在压测openhands proxy,底层openhands需要分析下cpu利用率

<!--more-->

cpu：使用cprofile性能分析

提供了python程序的确定性性能分析。


查看该容器被允许使用的最大内存量（内存天花板），单位是字节 (Bytes)
cat /sys/fs/cgroup/memory/memory.limit_in_bytes
该命令执行的那一瞬间，该容器内所有进程总共使用的内存量，单位是字节 (Bytes)
cat /sys/fs/cgroup/memory/memory.usage_in_bytes


```bash
kubectl exec -it memory-test-pod -- bash

kubectl cp angel-agent-pre-prj-jp2sncp2/agent-flow-openhands-proxy-75767487d6-7zd28:/usr/local/agent-flow-proxy/remote_openhands/test_job_quinnyqwu_101_single_2025-11-12_21-47-40_0_profile_RunInfer.prof test_job_quinnyqwu_101_single_2025-11-12_21-47-40_0_profile_RunInfer.prof
test_job_quinnyqwu_102_single_2025-11-13_10-38-50_15_profile_RunEval.prof  


kubectl cp angel-agent-pre-prj-jp2sncp2/agent-flow-openhands-proxy-595556bf48-c9lfc:/usr/local/agent-flow-proxy/remote_openhands/test_job_quinnyqwu_102_single_2025-11-13_10-38-50_15_profile_RunEval.prof test_job_quinnyqwu_102_single_2025-11-13_10-38-50_15_profile_RunEval.prof
kubectl cp angel-agent-pre-prj-jp2sncp2/agent-flow-openhands-proxy-595556bf48-c9lfc:/usr/local/agent-flow-proxy/remote_openhands/test_job_quinnyqwu_102_single_2025-11-13_10-38-50_15_profile_RunInfer.prof test_job_quinnyqwu_102_single_2025-11-13_10-38-50_15_profile_RunInfer.prof
```





# cpu