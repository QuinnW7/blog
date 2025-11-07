+++
date = '2025-11-07T10:00:00+08:00'
draft = false
title = 'Linux-内核日志'
+++

背景：

在压测一个模块，但发现pod里有oom的日志，想定位下先

```bash
[Fri Nov  7 15:28:33 2025] oom-kill:constraint=CONSTRAINT_MEMCG,nodemask=(null),cpuset=/,mems_allowed=0,oom_memcg=/tms/sec_saas,task_memcg=/tms/sec_saas,task=tdsp-container,pid=2080332,uid=0
[Fri Nov  7 15:28:33 2025] Memory cgroup out of memory: Killed process 2080332 (tdsp-container) total-vm:284668kB, anon-rss:90932kB, file-rss:2872kB, shmem-rss:0kB, UID:0 pgtables:260kB oom_score_adj:0 
```
所有的 OOM (Out of Memory) 事件都指向了同一个内存控制组（cgroup）：`oom_memcg=/tms/sec_saas`，并且被杀死的进程都叫 `tdsp-container`。在 Kubernetes/容器环境中，每个 Pod 的资源限制都是通过 cgroup 来实现的，这个路径 `/tms/sec_saas` 就是这个 Pod 所属的 cgroup。


然后就发现这个container不是我的，就找了管理集群的人，把住主机隔离了，是pod路由不到这台机器上

```bash
dmesg -T
# 查看是否有被oom的

dmesg -T | grep "Killed process"

```