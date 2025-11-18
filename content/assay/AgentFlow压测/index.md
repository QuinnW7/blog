+++
date = '2025-11-04T10:00:00+08:00'
draft = false
title = 'AngelFlow压测'
+++
<!--more-->

# 内存
无非就是加大并发，然后查看内存日志看下有没有oom
```bash
free -h # 查看当前内存剩余情况 
dmesg -T|grep "killed"
```

在这里深入探讨了下oom的分类情况：
## CONSTRAINT_MEMCG (Cgroup OOM)
```bash
[Wed Nov 12 20:13:52 2025] oom-kill:constraint=CONSTRAINT_MEMCG,nodemask=(null),cpuset=c5ecaabf7ab8c6aa87517337d80574f585bed86875a8a48f699a23e15fd607d7,mems_allowed=0,oom_memcg=/kubepods/pod1497cd14-3341-4f8e-914a-f3981c865110,task_memcg=/kubepods/pod1497cd14-3341-4f8e-914a-f3981c865110/c5ecaabf7ab8c6aa87517337d80574f585bed86875a8a48f699a23e15fd607d7,task=python3,pid=2830644,uid=0
[Wed Nov 12 20:13:52 2025] Memory cgroup out of memory: Killed process 2830644 (python3) total-vm:25102400kB, anon-rss:25022796kB, file-rss:5388kB, shmem-rss:0kB, UID:0 pgtables:49036kB oom_score_adj:-998
```

这里的“资源池”是单个 Cgroup 可用的内存。
触发条件非常简单直接：当一个 Cgroup 的当前总内存使用量 (memory.usage_in_bytes) 超过了 为其设定的硬性限制 (memory.limit_in_bytes) 时，OOM Killer 立即被针对该 Cgroup 触发。
这里的“总内存使用量”通常包括： 匿名内存 (Anonymous Pages)和页面缓存 (Page Cache)。这个判断过程是确定性的、机械的。 在这个过程中，内核完全不关心整个系统（节点）还有多少空闲内存。即使节点上还有 100GB 的空闲内存，只要你的这个 Cgroup 试图突破它自己的那 5GB 的小限制，OOM Killer 就会被精准地调用。

## CVM-CONSTRAINT_NONE (Global OOM)

```bash
[Wed Nov 12 16:21:27 2025] oom-kill:constraint=CONSTRAINT_NONE,nodemask=(null),cpuset=/,mems_allowed=0,global_oom,task_memcg=/user.slice/user-0.slice/session-1860939.scope,task=python3,pid=3668890,uid=0
[Wed Nov 12 16:21:27 2025] Out of memory: Killed process 3668890 (python3) total-vm:5663336kB, anon-rss:5553100kB, file-rss:0kB, shmem-rss:0kB, UID:0 pgtables:10968kB oom_score_adj:0
```
这里的“资源池”是整个操作系统的总可用内存。

触发条件要复杂得多，它更像是一个多因素综合评估，而非简单的阈值判断。

内核并不会等到可用内存精确为 0 的时候才行动，因为那样就太晚了。它有一套复杂的水线 (watermark) 机制和内存回收 (reclaim) 压力评估来决定何时触发全局 OOM。
简化的触发流程如下：
内存压力出现: 一个进程请求分配内存，但内核发现没有足够数量的空闲页 (free pages) 来满足请求。
尝试直接回收 (Direct Reclaim): 内核会暂停当前请求内存的进程，并尝试同步地、紧急地释放一些内存。它会尝试：
收缩页面缓存 (Shrinking page cache): 将干净的文件缓存页释放掉（因为数据还在磁盘上，以后可以再读回来）。
换出匿名页 (Swapping out anonymous pages): 将不常用的匿名内存（如程序堆内存）写入到 Swap 交换空间。 ...以及其他一些回收策略。
回收失败或压力过大: 如果经过一轮或多轮的“直接回收”后，仍然无法释放出足够的内存来满足最初的请求，说明系统正处于极度内存压力之下。内核会评估这种回收的“努力程度”和“成功率”。
触发全局 OOM Killer: 当内核的内存管理子系统判断**“通过常规回收手段已经无法满足内存分配请求，系统即将陷入不稳定状态”时，它就会最终决定**启动全局 OOM Killer。这被视为“最后的手段”。
所以，Global OOM 的触发条件可以概括为：
当系统处于高内存压力下，并且内核的内存回收机制 (reclaim) 在尝试满足一个内存分配请求时持续失败，无法在合理的时间内释放出足够的空闲内存页时，全局 OOM Killer 才会被触发，以避免系统完全锁定或崩溃。




内存分配脚本：
```python
import time
import sys

# --- 配置 ---
# 目标内存占用量 (GB)
TARGET_GB = 23.8
# --- 结束配置 ---

# 1 GB = 1024*1024*1024 bytes
BYTES_PER_GB = 1024 ** 3
TARGET_BYTES = int(TARGET_GB * BYTES_PER_GB)

# 为了避免因内存碎片化一次性分配失败，我们分块进行分配
# 例如，每次分配 100MB
CHUNK_SIZE_MB = 100
CHUNK_SIZE_BYTES = CHUNK_SIZE_MB * 1024 ** 2

# 用于持有所有内存块的列表，防止被垃圾回收
memory_holder = []

def allocate_memory():
    """
    逐步分配内存直到达到目标大小。
    """
    allocated_bytes = 0
    print(f"目标：占用 {TARGET_GB:.2f} GB 内存。")
    
    try:
        while allocated_bytes < TARGET_BYTES:
            # 计算本次需要分配的字节数
            bytes_to_alloc = min(CHUNK_SIZE_BYTES, TARGET_BYTES - allocated_bytes)
            
            # 创建一个 bytearray 对象来占用内存
            # bytearray 是一个可变的字节序列，非常适合这个任务
            chunk = bytearray(bytes_to_alloc)
            memory_holder.append(chunk)
            
            allocated_bytes += len(chunk)
            
            # 实时显示进度
            allocated_gb = allocated_bytes / BYTES_PER_GB
            progress = (allocated_bytes / TARGET_BYTES) * 100
            # 使用 \r 实现单行刷新，避免刷屏
            sys.stdout.write(f"\r已分配: {allocated_gb:.2f} GB / {TARGET_GB:.2f} GB ({progress:.1f}%)")
            sys.stdout.flush()

        print("\n\n内存分配完成。")
        print("脚本将保持运行以占用内存。")
        print("按 Ctrl+C 退出，脚本将自动释放所有内存。")

    except MemoryError:
        # 如果系统物理内存不足，会在这里捕获到异常
        allocated_gb = len(memory_holder) * CHUNK_SIZE_BYTES / BYTES_PER_GB
        print(f"\n\n错误：内存不足！系统无法分配更多内存。")
        print(f"已成功分配约 {allocated_gb:.2f} GB。")
        print("脚本将保持已分配的内存，按 Ctrl+C 退出。")
    except Exception as e:
        print(f"\n\n发生未知错误: {e}")


def hold_memory():
    """
    进入一个无限循环，以保持脚本运行，从而持有已分配的内存。
    """
    while True:
        try:
            time.sleep(1)
        except KeyboardInterrupt:
            # 当用户按下 Ctrl+C 时，这个异常会被触发
            break

if __name__ == "__main__":
    allocate_memory()
    hold_memory()
    print("\n程序已退出。Python 垃圾回收器将自动释放内存。")



```



# cpu


