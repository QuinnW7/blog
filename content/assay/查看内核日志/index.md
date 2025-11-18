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



curl --location 'http://trpc.agent-flow.controller.x8.v2.production.polaris:8000/api/agentflow/v1/task/messages' \
--header 'Content-Type: application/json' \
--data '{
"task_id": "agentflow-swe-754b0174-ca2f-460f-bd69-870793cc1825"
}'


8c24g 不改的时候搞一下 test_job_quinnyqwu_103
![Clipboard_Screenshot_1763022099.png](..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2Fprivate%2Ftmp%2F.tempData_V2%2F.new_Clipboard%2FClipboard_Screenshot_1763022099.png)

  
```bash
kubectl cp angel-agent-pre-prj-jp2sncp2/agent-flow-openhands-proxy-86475b6b6d-5mkdm:/usr/local/agent-flow-proxy/remote_openhands/test_job_quinnyqwu_103_single_2025-11-13_16-07-10_21_profile_RunEval.prof test_job_quinnyqwu_103_single_2025-11-13_16-07-10_21_profile_RunEval.prof
kubectl cp angel-agent-pre-prj-jp2sncp2/agent-flow-openhands-proxy-86475b6b6d-5mkdm:/usr/local/agent-flow-proxy/remote_openhands/test_job_quinnyqwu_103_single_2025-11-13_16-07-10_21_profile_RunInfer.prof test_job_quinnyqwu_103_single_2025-11-13_16-07-10_21_profile_RunInfer.prof
[test_job_quinnyqwu_103_single_2025-11-13_16-07-10_21_profile_RunInfer.prof](..%2F..%2F..%2F..%2F..%2F..%2FDownloads%2Ftest_job_quinnyqwu_103_single_2025-11-13_16-07-10_21_profile_RunInfer.prof)
[test_job_quinnyqwu_103_single_2025-11-13_16-07-10_21_profile_RunEval.prof](..%2F..%2F..%2F..%2F..%2F..%2FDownloads%2Ftest_job_quinnyqwu_103_single_2025-11-13_16-07-10_21_profile_RunEval.prof)
```

发现没去掉日志也是100多秒，没有400多秒了，所以有问题，而且这次的cpu利用率高起来了
8c24g 改完搞一下

8c36g 不改的时候搞一下
8c36g 改完搞一下


使用perf分析吧
```bash
perf
sudo py-spy top --pid 1610
sudo py-spy record -p  --duration 800 -o python_view.svg


# 针对同一个 PID
sudo perf record -p 1610 -g -F 99 --period 800

sudo perf script | ./FlameGraph/stackcollapse-perf.pl | ./FlameGraph/flamegraph.pl > system_view.svg



```